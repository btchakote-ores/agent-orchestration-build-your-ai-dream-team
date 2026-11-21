# Agent team

I am using GitHub Copilot CLI in a Codespace to orchestrate four custom agents that build Mona's Project Pulse dashboard. All definitions live in `.github/agents/`.

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 5.5 | Breaks the request into phases, delegates to the specialists with explicit file scopes, runs tasks in parallel only when scopes don't overlap, verifies the integrated result, and reports. Does not implement. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 5.5 | Researches the repo and docs, then returns an implementation plan with steps, file assignments, dependencies, parallel/sequential work, edge cases, validation, and open questions. Writes no code. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 | Implements code and logic within its assigned files, plus runnable-app support such as `.vscode/launch.json` (`cwd` of `${workspaceFolder}/app`, opening `index.html`). Validates before reporting. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.7 Flash | Handles UI/UX, accessibility, and visual design: polished dashboard styling with project cards, status badges, priority treatment, responsive layout, and `.dashboard` / `.project-card` CSS hooks. | `.github/agents/designer.agent.md` |

## Workflow

1. Orchestrator asks Planner for a plan.
1. Orchestrator splits the plan into phases with non-overlapping file scopes.
1. Coder and Designer implement their assigned files.
1. Orchestrator verifies the integrated result and reports.

No agent stages, commits, or pushes; I control all git operations through Copilot CLI prompts.
