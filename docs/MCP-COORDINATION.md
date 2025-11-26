# MCP Coordination Patterns

> How Agentic Sprint coordinates parallel agents using MCP tools and file-based signaling.

## Overview

When running UI tests, two agents work in parallel:
- **UI Test Agent** - Uses Playwright to interact with the browser
- **Next.js Diagnostics Agent** - Monitors for runtime errors

These agents have no direct communication channel. We use a **file-based signaling pattern** to coordinate their lifecycle.

## The Problem

```
┌─────────────────┐     ┌─────────────────────┐
│  UI Test Agent  │     │  Diagnostics Agent  │
│  (Playwright)   │     │  (Next.js DevTools) │
└────────┬────────┘     └──────────┬──────────┘
         │                         │
         │  Both spawned           │
         │  simultaneously         │
         │                         │
         │  ← No communication →   │
         │                         │
         │  How does diagnostics   │
         │  know when to stop?     │
         │                         │
```

**Challenge:** The diagnostics agent polls for errors continuously. It has no way to know when the UI test agent finishes (either automated tests complete or user closes browser in manual mode).

## The Solution: Signal File

We use a simple file-based signal:

```
.claude/sprint/[N]/.ui-test-done
```

### Flow

```
1. Orchestrator deletes any stale signal file
   rm -f .claude/sprint/[N]/.ui-test-done

2. Orchestrator spawns BOTH agents in parallel
   (single message with multiple Task calls)

3. UI Test Agent:
   - Runs tests (or waits for manual interaction)
   - When done: writes signal file
   - Returns report

4. Diagnostics Agent:
   - Polls for errors
   - Between polls: checks for signal file
   - When signal file exists: stops polling
   - Returns report

5. Orchestrator:
   - Collects both reports
   - Cleans up signal file
   - Proceeds with workflow
```

### Signal File Content

Simple content: `done`

```bash
# UI Test Agent writes:
echo "done" > .claude/sprint/[N]/.ui-test-done
```

```python
# Diagnostics Agent checks:
try:
    read(".claude/sprint/[N]/.ui-test-done")
    # File exists → stop polling
except:
    # File doesn't exist → continue polling
```

## MCP Tool Separation

Each agent uses a specific set of MCP tools:

### UI Test Agent - Playwright Only

```
mcp__playwright__browser_navigate
mcp__playwright__browser_snapshot
mcp__playwright__browser_click
mcp__playwright__browser_type
mcp__playwright__browser_fill_form
mcp__playwright__browser_take_screenshot
mcp__playwright__browser_console_messages
mcp__playwright__browser_wait_for
mcp__playwright__browser_close
```

**Why:** Clear responsibility - handles browser interaction.

### Diagnostics Agent - Next.js DevTools Only

```
mcp__next-devtools__nextjs_index
mcp__next-devtools__nextjs_call
mcp__next-devtools__nextjs_docs
```

**Why:** Monitors server-side errors that browser tools can't see.

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

## Testing Modes

### Automated Mode

```
┌─────────────────┐     ┌─────────────────────┐
│  UI Test Agent  │     │  Diagnostics Agent  │
└────────┬────────┘     └──────────┬──────────┘
         │                         │
         │ Run test scenarios      │ Poll for errors
         │                         │
         │ All tests complete      │ Check signal file
         │                         │   (not found)
         │ Write signal file       │
         │                         │ Check signal file
         │                         │   (found!)
         │ Return report           │ Stop polling
         │                         │ Return report
```

### Manual Mode

```
┌─────────────────┐     ┌─────────────────────┐
│  UI Test Agent  │     │  Diagnostics Agent  │
└────────┬────────┘     └──────────┬──────────┘
         │                         │
         │ Open browser            │ Poll for errors
         │                         │
         │ User interacts...       │ Check signal file
         │                         │   (not found)
         │                         │ Poll for errors
         │ User closes browser     │   ...continues...
         │                         │
         │ Write signal file       │ Check signal file
         │                         │   (found!)
         │ Return report           │ Stop polling
         │                         │ Return report
```

## Implementation Details

### Orchestrator Cleanup

Before spawning:
```bash
rm -f .claude/sprint/[N]/.ui-test-done
```

After collecting reports:
```bash
rm -f .claude/sprint/[N]/.ui-test-done
```

### UI Test Agent Signal Write

Using the Write tool:
```
Path: .claude/sprint/[N]/.ui-test-done
Content: done
```

### Diagnostics Agent Signal Check

Using the Read tool:
```
Path: .claude/sprint/[N]/.ui-test-done

If read succeeds → signal file exists → stop
If read fails → signal file missing → continue polling
```

## Error Handling

### Agent Crash

If one agent crashes:
- The other agent may wait indefinitely
- Orchestrator should implement timeout
- Consider health check mechanisms

### Stale Signal File

Always delete before starting:
```bash
rm -f .claude/sprint/[N]/.ui-test-done
```

This prevents false positives from previous runs.

### Parallel Spawn Requirement

Both agents MUST be spawned in the **same message**:

```
# CORRECT - Parallel execution
Task: ui-test-agent ...
Task: nextjs-diagnostics-agent ...

# WRONG - Sequential execution
Task: ui-test-agent ...
[wait for completion]
Task: nextjs-diagnostics-agent ...
```

## Why File-Based Signaling?

**Alternatives considered:**

1. **Shared memory** - Agents don't share state
2. **Direct communication** - Agents can't call each other
3. **Orchestrator polling** - Adds complexity and latency
4. **Timeout-based** - Unpredictable test durations

**File-based advantages:**

- Simple and universal
- Works with any agent type
- No inter-process communication needed
- Easy to debug (check if file exists)
- Works across different execution environments

## Extending the Pattern

For other parallel agent scenarios:

1. Define a signal file path convention
2. Writer agent creates file when done
3. Reader agent polls for file existence
4. Orchestrator cleans up before/after

Example for hypothetical parallel builds:
```
.claude/sprint/[N]/.build-backend-done
.claude/sprint/[N]/.build-frontend-done
```
