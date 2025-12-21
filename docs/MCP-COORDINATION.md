# MCP Coordination Patterns

> How Sprint coordinates parallel agents using Chrome browser MCP and Next.js DevTools MCP.

## Overview

When running UI tests, agents work in parallel:
- **UI Test Agent** - Uses Chrome browser MCP to interact with the browser
- **Next.js Diagnostics Agent** (optional) - Monitors for runtime errors via Next.js DevTools MCP

These agents use separate MCP toolsets and coordinate through the orchestrator.

## MCP Tool Separation

Each agent uses a specific set of MCP tools:

### UI Test Agent - Chrome Browser MCP

```
mcp__claude-in-chrome__tabs_context_mcp
mcp__claude-in-chrome__tabs_create_mcp
mcp__claude-in-chrome__navigate
mcp__claude-in-chrome__computer
mcp__claude-in-chrome__read_page
mcp__claude-in-chrome__read_console_messages
mcp__claude-in-chrome__read_network_requests
```

**Why:** Handles browser interaction and user-facing testing.

### Diagnostics Agent - Next.js DevTools MCP

```
mcp__next-devtools__nextjs_index
mcp__next-devtools__nextjs_call
mcp__next-devtools__nextjs_docs
```

**Why:** Monitors server-side errors, hydration issues, and compilation errors that browser tools can't see.

## Testing Modes

### Automated Mode

Both agents complete when their tasks finish:

```
┌─────────────────┐     ┌─────────────────────┐
│  UI Test Agent  │     │  Diagnostics Agent  │
└────────┬────────┘     └──────────┬──────────┘
         │                         │
         │ Run test scenarios      │ Poll for errors
         │                         │
         │ All tests complete      │ Monitoring active
         │                         │
         │ Return report           │ Return report
         │                         │
```

### Manual Mode

In manual mode, the UI test agent detects when the user closes the browser tab:

```
┌─────────────────┐     ┌─────────────────────┐
│  UI Test Agent  │     │  Diagnostics Agent  │
└────────┬────────┘     └──────────┬──────────┘
         │                         │
         │ Open browser            │ Poll for errors
         │                         │
         │ User interacts...       │ Monitoring active
         │                         │
         │ User closes tab         │ ...continues...
         │ (detected via           │
         │  tabs_context_mcp)      │ Agent times out
         │                         │   or orchestrator
         │ Return report           │   collects reports
         │                         │
```

**Tab Close Detection:** The UI test agent periodically checks if its tabId still exists in the tab list. When the user closes the tab, the agent knows testing is complete.

```
Call: mcp__claude-in-chrome__tabs_context_mcp

If tabId not in response → user closed tab → testing complete
```

## Parallel Spawn Requirement

Both agents MUST be spawned in the **same message** for true parallel execution:

```
# CORRECT - Parallel execution
Task: ui-test-agent ...
Task: nextjs-diagnostics-agent ...

# WRONG - Sequential execution
Task: ui-test-agent ...
[wait for completion]
Task: nextjs-diagnostics-agent ...
```

The orchestrator spawns both agents in a single message, then collects their reports when they complete.

## Docker vs Local Environments

### Local Development

```
Diagnostics Agent:
1. Call nextjs_index to discover running servers
2. Get port number automatically
3. Call nextjs_call with discovered port
```

### Docker Deployment

```
Diagnostics Agent:
1. SKIP nextjs_index (doesn't work for containers)
2. Use port directly from prompt (e.g., 8001)
3. Call nextjs_call with port="8001"
```

The MCP endpoint works through Docker port mapping:
```
http://localhost:8001/_next/mcp
```

## Error Handling

### Agent Timeout

If an agent takes too long:
- The orchestrator should implement timeout logic
- Consider health check mechanisms
- Diagnostics agent uses longer timeout for manual mode (~5 minutes)

### Missing Tab

If the browser tab is closed unexpectedly:
- UI test agent detects via `tabs_context_mcp`
- Agent captures final state if possible
- Returns partial report with explanation

## Manual Test Reports

The `/sprint:test` command saves reports to the sprint directory:

```
.claude/sprint/[N]/manual-test-report.md
```

When `/sprint` runs, it reads these reports and passes them to the architect for prioritization. Reports are automatically cleaned up when the sprint completes.

## Report Lifecycle

1. **User runs `/sprint:test`** → Report saved to sprint directory
2. **User runs `/sprint`** → Architect reads report, prioritizes fixes
3. **Sprint completes** → Report cleaned up (no longer relevant)

This ensures user observations from manual testing feed directly into the automated sprint workflow.
