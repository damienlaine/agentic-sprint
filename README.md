# Agentic Sprint

> Autonomous multi-agent development system for Claude Code. Spec-driven, iterative sprints with specialized agents.

**Stop prompting in circles.** Agentic Sprint replaces ad-hoc AI coding with structured, specification-driven development. Write specs, run `/sprint`, and let coordinated agents handle the rest.

At its core, the `/sprint` command is a **spec-driven, self-iterative state machine** — it reads your specifications, orchestrates specialized agents through defined phases, and loops autonomously until the work is done or validation passes.

## What is Agentic Sprint?

Agentic Sprint is a framework that turns Claude Code into an autonomous development team:

- **Project Architect** analyzes requirements, creates specifications, and coordinates work
- **Implementation Agents** (Python, Next.js, CI/CD) build features according to specs
- **Testing Agents** (QA, UI, Diagnostics) validate the implementation
- **Sprint Orchestrator** — the self-iterative state machine that manages phases, handoffs, and convergence

```
You write specs → Agents implement → Tests validate → Iterate until done
```

The orchestrator drives the loop: specs in, working code out. No manual intervention required between phases.

### Why It Works

Unlike single-shot prompting where context bloats and AI mistakes compound, Agentic Sprint uses a **convergent multi-pass approach**:

- **Context preservation** — Each agent receives only what it needs (specs, contract, relevant code). No wasted tokens on irrelevant history.
- **Specs shrink, not grow** — Completed work is removed from specs. Each iteration focuses only on what remains.
- **Errors get erased** — Working code stays untouched while issues get fixed. The signal-to-noise ratio improves with each pass.

Think of it like a **diffusion process**: the picture starts noisy, but with each iteration, the noise reduces and clarity emerges. By the final pass, only the solution remains.

Most sprints converge well before 5 iterations. If they don't, the system pauses and asks you what to do — adjust specs, continue iterating, or intervene manually. You stay in control.

### The Second Brain Effect

Two files give agents persistent memory across sprints — reducing token usage and keeping context focused:

**`.claude/project-goals.md`** — The business brain
- Product vision and target audience
- Market analysis and differentiators
- Success metrics and constraints
- What you're building and *why*

The architect reads this to make decisions aligned with your product goals, not just technical specs.

**`.claude/project-map.md`** — The technical brain
- Project structure and architecture
- API surface and database schema
- Routes, components, environment variables
- *Where* everything lives and *how* it connects

Agents read this instead of scanning the entire codebase. Straight to the relevant files.

Both files are maintained by the architect and stay lean — no bloat, just current truth. This is how agents "remember" your project without consuming your context window.

## Quick Start

### 1. Install

Copy the `.claude` folder to your project:

```bash
# Clone this repo
git clone https://github.com/damienlaine/agentic-sprint.git

# Copy to your project
cp -r agentic-sprint/.claude your-project/
```

### 2. Create Your First Sprint

```bash
# In your project directory with Claude Code
/new-sprint
```

This creates `.claude/sprint/1/specs.md`. Edit it with your requirements.

### 3. Run the Sprint

```bash
/sprint
```

Watch the agents work:
1. Architect analyzes specs and creates detailed specifications
2. Implementation agents build in parallel
3. Testing agents validate the work
4. Architect reviews and iterates (up to 5 times)
5. Sprint completes with a status summary

## Core Concepts

### Sprint Directory Structure

```
.claude/
├── agents/                    # Agent definitions
│   ├── project-architect.md   # Coordinator agent
│   ├── python-dev.md          # Backend agent
│   ├── nextjs-dev.md          # Frontend agent
│   ├── qa-test-agent.md       # API/unit testing
│   ├── ui-test-agent.md       # E2E testing
│   └── ...
├── commands/                  # Slash commands
│   ├── sprint.md              # Main workflow
│   ├── new-sprint.md          # Create sprints
│   ├── generate-map.md        # Generate project map
│   └── clean-sprints.md       # Cleanup utility
├── sprint/                    # Sprint artifacts
│   └── 1/                     # Sprint 1
│       ├── specs.md           # Your requirements
│       ├── api-contract.md    # Generated API spec
│       ├── backend-specs.md   # Backend guidance
│       ├── frontend-specs.md  # Frontend guidance
│       ├── status.md          # Sprint status
│       └── *-report-*.md      # Agent reports
├── project-map.md             # System overview
└── project-goals.md           # Product objectives
```

### Specification Files

**`specs.md`** - Your input (minimal or detailed):
```markdown
# Sprint 1 Specifications

## Goal
Add user authentication with OAuth

## Testing
- QA: required
- UI Testing: required
- UI Testing Mode: automated
```

**`api-contract.md`** - Generated shared interface:
```markdown
## POST /api/auth/login
Request: { email: string, password: string }
Response: { token: string, user: User }
```

### Agent Reports

Every agent returns a structured report:
```markdown
## BACKEND IMPLEMENTATION REPORT

### CONFORMITY STATUS: YES

### DEVIATIONS:
None

### FILES CHANGED:
- backend/api/auth.py
- backend/models/user.py

### ISSUES FOUND:
- None
```

## Commands

| Command | Description |
|---------|-------------|
| `/sprint` | Run the full sprint workflow |
| `/sprint --manual` | Manual UI testing with live browser |
| `/new-sprint` | Create a new sprint |
| `/generate-map` | Generate project-map.md |
| `/clean-sprints` | Remove old sprint directories |

### Manual Testing Mode

Sometimes you want to explore the UI yourself rather than run automated tests. The `--manual` flag enables **hybrid testing**:

```bash
/sprint --manual
```

What happens:
1. **Playwright opens a browser** pointing to your app (e.g., `http://localhost:8001`)
2. **A diagnostics agent runs in parallel**, monitoring for runtime errors in real-time
3. **You interact with the app manually** — click around, test forms, explore edge cases
4. **All errors are captured** by the diagnostics agent as you explore
5. **When you close the browser**, both agents finalize and return their reports

We ship a Next.js diagnostics agent out of the box. **Write your own** for other stacks — any MCP-compatible observability tool works (Laravel Telescope, Rails logs, etc.).

This is perfect for:
- Exploratory testing after implementation
- Debugging UI issues with live error monitoring
- Quick smoke tests without writing test specs
- Validating UX flows that are hard to automate

You can also trigger manual mode from specs:
```markdown
## Testing
- UI Testing Mode: manual
```

## Agents

The agent system is **extensible by design**. Start with the built-in agents, then add your own.

### Built-in Implementation Agents

| Agent | Tech Stack | Input Specs |
|-------|------------|-------------|
| `python-dev` | FastAPI, PostgreSQL | `api-contract.md`, `backend-specs.md` |
| `nextjs-dev` | Next.js 16, React 19 | `api-contract.md`, `frontend-specs.md` |
| `cicd-agent` | GitHub Actions, Docker | `cicd-specs.md` |

### Built-in Testing Agents

| Agent | Purpose | Tools |
|-------|---------|-------|
| `qa-test-agent` | API & unit tests | pytest, jest |
| `ui-test-agent` | E2E browser tests | Playwright MCP |
| `nextjs-diagnostics-agent` | Runtime monitoring (Next.js) | Next.js DevTools MCP |

The diagnostics agent pattern can be replicated for any stack — create your own using MCP-compatible observability tools (Laravel Telescope, Rails logs, Docker logs, etc.).

### Allpurpose Agent (Fallback)

No specialized agent for your stack? The `allpurpose-agent` adapts to **any technology** — Go, Rust, Flutter, Ruby, whatever you need.

The architect automatically uses it when no specialized agent exists, creating appropriate spec files (e.g., `mobile-specs.md`, `cli-specs.md`) and prompting the agent with the right context. You just write your `specs.md` — the architect handles the rest.

### Write Your Own Agents

These agents are starting points, not final products. **Fork, extend, contribute.**

See [Adding Custom Agents](docs/AGENTS.md#adding-custom-agents) for the guide.

## MCP Integration

Agentic Sprint leverages MCP (Model Context Protocol) for browser automation and Next.js monitoring:

### Playwright MCP
UI Test Agent uses Playwright tools for browser interaction:
```
mcp__playwright__browser_navigate
mcp__playwright__browser_click
mcp__playwright__browser_snapshot
...
```

### Next.js DevTools MCP
Diagnostics Agent monitors for runtime errors:
```
mcp__next-devtools__nextjs_call
mcp__next-devtools__get_errors
...
```

### Parallel Agent Coordination

When UI tests run, both agents work in parallel. They coordinate via a signal file:
```
.claude/sprint/[N]/.ui-test-done
```

See [MCP-COORDINATION.md](docs/MCP-COORDINATION.md) for details.

## Customization

### Adding Custom Agents

1. Create `.claude/agents/your-agent.md`:
```yaml
---
name: your-agent
description: What this agent does
model: opus
---

[Agent instructions...]
```

2. The architect can now request your agent in SPAWN REQUEST blocks.

### Extending Specs

Add any spec files your agents need:
- `mobile-specs.md` for mobile development
- `security-specs.md` for security-focused work
- `performance-specs.md` for optimization tasks

## Best Practices

1. **Write clear specs** - The better your `specs.md`, the better the output
2. **Use project-goals.md** - Give context about your product vision
3. **Iterate small** - Multiple small sprints beat one big sprint
4. **Checkpoint often** - Commit before running sprints
5. **Review reports** - Agent reports show what was done and why

## Project Structure for New Projects

Recommended setup:
```
your-project/
├── .claude/           # Agentic Sprint (this repo)
├── backend/           # Python/FastAPI
├── frontend/          # Next.js
├── docker-compose.yml # Development environment
└── README.md
```

## Troubleshooting

### Sprint stuck in iteration loop
- Check `status.md` for blockers
- Review agent reports for errors
- Max 5 iterations before pause

### Agents not following specs
- Ensure `api-contract.md` is clear and complete
- Check for conflicting information in spec files
- Architect may need to clarify specs

### MCP tools not working
- Verify MCP servers are running
- Check port configuration (Docker: 8001, Local: 3000)
- See [MCP-COORDINATION.md](docs/MCP-COORDINATION.md)

## Documentation

- [Agent Architecture](docs/AGENTS.md) - Deep dive into agent system
- [MCP Coordination](docs/MCP-COORDINATION.md) - Parallel agent patterns
- [Example Project Goals](examples/project-goals.md) - Templates for project-goals.md

## License

MIT License - See [LICENSE](LICENSE)

## Contributing

**Contributions are highly encouraged!** This is a community project — the built-in agents are just a starting point.

Ways to contribute:
- **Add new agents** for different tech stacks (Go, Rust, Vue, Django, etc.)
- **Improve existing agents** with better prompts, patterns, or capabilities
- **Share your workflows** — what sprint configurations work well?
- **Report issues** — what breaks? what's confusing?
- **Improve docs** — help others get started

Please read the [agent documentation](docs/AGENTS.md) before submitting PRs.

---

Built with Claude Code. Designed for autonomous development.

---

<sub>*This documentation was generated with AI assistance and may contain errors. Please report issues on GitHub.*</sub>
