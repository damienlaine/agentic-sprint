# Agentic Sprint - Claude Code Configuration

This project uses the Agentic Sprint framework for autonomous, spec-driven development.

## Available Commands

- `/sprint` - Run the full sprint workflow
- `/sprint --manual` - Quick manual UI testing mode
- `/new-sprint` - Create a new sprint
- `/generate-map` - Generate project-map.md
- `/clean-sprints` - Remove old sprint directories

## Important Guidelines

### Git Practices
- Never reference AI or Claude in commits
- Never push to remote repositories unless explicitly instructed
- Create checkpoints before running sprints

### Sprint Workflow
- Always create specs.md before running /sprint
- Review agent reports after each iteration
- Maximum 5 iterations per sprint before pause

### Agent Constraints
- Agents cannot modify status.md or project-map.md (architect only)
- Agents return structured reports, not verbose logs
- API contract is the shared source of truth

## Project Structure

See `/docs/AGENTS.md` for the full agent architecture.
See `/docs/MCP-COORDINATION.md` for parallel agent patterns.
