# Sprint - Claude Code Plugin

Autonomous multi-agent development framework with spec-driven sprints.

## Available Commands

- `/sprint:setup` - Interactive project onboarding (creates project-goals.md and project-map.md)
- `/sprint` - Run the full sprint workflow
- `/sprint:test` - Quick manual UI testing with Chrome browser
- `/sprint:new` - Create a new sprint
- `/sprint:generate-map` - Auto-generate project-map.md
- `/sprint:clean` - Remove old sprint directories

## Important Guidelines

### Git Practices
- Never reference AI or Claude in commits
- Never reference sprints in commits (sprints are ephemeral internal workflow)
- Never push to remote repositories unless explicitly instructed
- Create checkpoints before running sprints

### Sprint Workflow
- Run `/sprint:setup` first to create project-goals.md and project-map.md
- Create specs.md before running sprint (use `/sprint:new`)
- Review agent reports after each iteration
- Maximum 5 iterations per sprint before pause

### Agent Constraints
- Agents cannot modify status.md or project-map.md (architect only)
- Agents return structured reports, not verbose logs
- API contract is the shared source of truth

## Project Structure

See `docs/AGENTS.md` for the full agent architecture.
