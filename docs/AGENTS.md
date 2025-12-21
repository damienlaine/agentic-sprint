# Agent Architecture

> Comprehensive documentation of the Sprint agent system.

## Overview

Sprint uses a hierarchical multi-agent architecture where specialized agents collaborate under the coordination of an orchestrator and architect.

```
┌─────────────────────────────────────────────────────────┐
│                    /sprint Command                       │
│                   (Orchestrator Logic)                   │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                  Project Architect                       │
│            (Decision Maker & Coordinator)                │
└─────────────────────────┬───────────────────────────────┘
                          │
          ┌───────────────┼───────────────┐
          │               │               │
          ▼               ▼               ▼
    ┌──────────┐   ┌──────────┐   ┌──────────┐
    │ Backend  │   │ Frontend │   │  CI/CD   │
    │   Dev    │   │   Dev    │   │  Agent   │
    └──────────┘   └──────────┘   └──────────┘
          │               │
          └───────┬───────┘
                  ▼
    ┌─────────────────────────────────────┐
    │           Testing Phase              │
    │  ┌─────────┐     ┌───────────────┐  │
    │  │   QA    │     │   UI Test +   │  │
    │  │  Agent  │     │  Diagnostics  │  │
    │  └─────────┘     └───────────────┘  │
    └─────────────────────────────────────┘
```

## Agent Roles

### Orchestrator (Sprint Command)

**Role:** Conductor of the development workflow

**Responsibilities:**
- Parse `/sprint` command
- Locate sprint specifications
- Manage the phase loop (Planning → Implementation → Testing → Review)
- Spawn agents at the right time
- Collect and persist reports
- Track iteration count (max 5)

**Does NOT:**
- Make architectural decisions (that's the architect's job)
- Implement code
- Decide which agents to spawn (follows architect's requests)

### Project Architect

**Role:** Technical decision-maker and coordinator

**File:** `agents/project-architect.md` (plugin)

**Responsibilities:**
- Analyze requirements and create specifications
- Create API contracts and technical specs
- Maintain `project-map.md` (system overview)
- Maintain `status.md` (sprint state)
- Request agent spawns via `## SPAWN REQUEST` blocks
- Review reports and decide next steps
- Signal sprint completion with `FINALIZE`

**Key Outputs:**
- `api-contract.md` - Shared API interface
- `backend-specs.md` - Backend implementation guide
- `frontend-specs.md` - Frontend implementation guide
- `qa-specs.md` - Test scenarios
- `ui-test-specs.md` - E2E test scenarios
- `cicd-specs.md` - CI/CD requirements

### Implementation Agents

#### Python Dev

**File:** `agents/python-dev.md` (plugin)

**Stack:** FastAPI, PostgreSQL, async patterns

**Input:** `api-contract.md`, `backend-specs.md`

**Output:** `## BACKEND IMPLEMENTATION REPORT`

#### Next.js Dev

**File:** `agents/nextjs-dev.md` (plugin)

**Stack:** Next.js 16, React 19, TypeScript, TailwindCSS

**Input:** `api-contract.md`, `frontend-specs.md`

**Output:** `## FRONTEND IMPLEMENTATION REPORT`

#### CI/CD Agent

**File:** `agents/cicd-agent.md` (plugin)

**Scope:** Pipelines, deployments, secrets

**Input:** `cicd-specs.md`

**Output:** `## CICD IMPLEMENTATION REPORT`

#### Allpurpose Agent

**File:** `agents/allpurpose-agent.md` (plugin)

**Scope:** Any technology not covered by specialized agents

**Input:** Task-specific specs

**Output:** `## IMPLEMENTATION REPORT`

### Testing Agents

#### QA Test Agent

**File:** `agents/qa-test-agent.md` (plugin)

**Scope:** API tests, unit tests, integration tests

**Input:** `api-contract.md`, `qa-specs.md`

**Output:** `## QA REPORT`

#### UI Test Agent

**File:** `agents/ui-test-agent.md` (plugin)

**Scope:** Browser-based E2E testing with Chrome browser MCP

**Tools:** `mcp__claude-in-chrome__*` only

**Modes:**
- `AUTOMATED` - Execute all test scenarios from specs
- `MANUAL` - Open browser for user interaction; detects tab close to know when testing is complete

**Output:** `## UI TEST REPORT`

#### Next.js Diagnostics Agent

**File:** `agents/nextjs-diagnostics-agent.md` (plugin)

**Scope:** Runtime error monitoring during UI testing

**Tools:** `mcp__next-devtools__*` only

**Output:** `## NEXTJS DIAGNOSTICS REPORT`

### Website Designer

**File:** `agents/website-designer.md` (plugin)

**Scope:** Static marketing websites for GitHub Pages

**Focus:** SEO, conversion optimization

**Output:** Files changed + design decisions

## Agent Communication

### Spawn Request Format

Architect requests agents via structured blocks:

```markdown
## SPAWN REQUEST

- python-dev
- nextjs-dev
- cicd-agent
```

### Report Format

All agents return structured reports:

```markdown
## [AGENT TYPE] REPORT

### CONFORMITY STATUS: [YES/NO]

### DEVIATIONS:
[List or "None"]

### FILES CHANGED:
- [file paths]

### ISSUES FOUND:
- [issues]
```

## Agent Constraints

### What Agents CAN Do

- Read specification files
- Read project-map.md (read-only)
- Implement/modify application code
- Return structured reports

### What Agents CANNOT Do

- Spawn other agents
- Modify `.claude/sprint/[index]/status.md`
- Modify `.claude/project-map.md`
- Create methodology or documentation files
- Start servers or infrastructure
- Push to git repositories

## Model Selection

Agents specify their preferred model:

```yaml
model: opus    # For complex implementation work
model: sonnet  # For lighter tasks (CI/CD, diagnostics)
```

## Adding Custom Agents

**Option 1: Contribute to the plugin** - Add agents to `agents/` in the plugin

**Option 2: Project-local agents** - Create agents in your project's `.claude/agents/` directory

Steps:
1. Create agent definition as a markdown file
2. Follow the standard frontmatter format:
   ```yaml
   ---
   name: agent-name
   description: What this agent does
   model: opus/sonnet
   ---
   ```
3. Define clear input specs and output format
4. Specify tool restrictions (if any)
5. The architect can then request your agent via SPAWN REQUEST blocks

## Sprint Iteration Flow

```
Iteration 1:
  Architect → analyzes, creates specs
  Architect → SPAWN REQUEST: python-dev, nextjs-dev
  Orchestrator → spawns implementation agents
  Agents → return reports
  Orchestrator → saves reports, sends to architect

Iteration 2:
  Architect → reviews reports, updates specs
  Architect → SPAWN REQUEST: qa-test-agent, ui-test-agent
  Orchestrator → spawns testing agents
  Agents → return reports
  Orchestrator → saves reports, sends to architect

Iteration 3:
  Architect → reviews test results
  Architect → FINALIZE (or more spawn requests)
  Orchestrator → completes sprint
```

## The Convergent Design

The multi-agent architecture is designed to **converge** rather than diverge:

### Context Preservation
Each agent receives only what it needs:
- Implementation agents get specs + contract (not full conversation history)
- Testing agents get contract + test specs (not implementation details)
- No wasted tokens on irrelevant context

### Specs Shrink Over Time
Unlike traditional prompting where context grows:
- Completed work is **removed** from spec files
- Each iteration focuses only on remaining work
- TODOs decrease, clarity increases
- Status.md is rewritten (not appended)

### Errors Get Erased
The iterative process naturally eliminates mistakes:
- Working code stays untouched
- Only failing parts get re-implemented
- Signal-to-noise ratio improves each pass
- Like a diffusion model: noise reduces, clarity emerges

Most sprints converge well before 5 iterations. If not, the system pauses and asks you what to do — adjust specs, continue iterating, or intervene manually. The limit is a checkpoint, not a hard stop.

## Best Practices

1. **Keep agents focused** - Each agent has a single responsibility
2. **Explicit contracts** - API contract is the source of truth
3. **Structured reports** - Machine-parseable for iteration
4. **Minimal output** - Concise reports, no verbose logs
5. **No cross-talk** - Agents don't communicate directly
6. **Orchestrator manages state** - Agents are stateless
7. **Prune aggressively** - Remove completed items from specs each iteration
