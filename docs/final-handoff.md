# Project Pulse final handoff

## Summary

Mona's Project Pulse dashboard is built. It is a static page that loads project data and shows each project as a card with its status, priority, owner, recent activity and an optional summary. It was built with GitHub Copilot CLI in a Codespace, using four custom agents defined in `.github/agents/`.

## Agent team and contributions

| Agent | Model | Contribution |
| --- | --- | --- |
| Orchestrator | Claude Opus 5.5 | Split the plan into phases, ran Designer and Coder in parallel on separate files, reviewed the combined result, sent fixes to the right agent and coordinated validation. Did not write app code. |
| Planner | Claude Opus 5.5 | Researched the repo and grading workflow, then wrote `docs/project-pulse-plan.md`: file assignments, shared contracts, dependencies, parallel work and validation. |
| Designer | Gemini 3.7 Flash | Owned visual design and accessibility in `app/styles.css`. Fixed a CSS rule that kept an empty loading box visible (`.dashboard-message[hidden]`). |
| Coder | GPT-5.5 | Wrote `app/index.html`, `app/project-data.json` and `.vscode/launch.json`, and ran validation. |

## Deliverables

| File | Description |
| --- | --- |
| `app/index.html` | Page structure plus an inline script that loads `project-data.json` and builds a `.project-card` for each project. |
| `app/styles.css` | Dashboard layout, cards, status and priority badges, responsive grid, focus, reduced-motion and forced-colors styles. |
| `app/project-data.json` | Sample data: 5 projects covering every status and priority. |
| `.vscode/launch.json` | The **Run Project Pulse Dashboard** launch configuration. |
| `docs/agent-team.md` | Describes the four agents and the workflow. |
| `docs/project-pulse-plan.md` | The implementation plan from the Planner. |

## How to run

1. Open **Run and Debug** in VS Code.
1. Choose **Run Project Pulse Dashboard** (defined in `.vscode/launch.json`).
1. It runs `python3 -m http.server 5500` from `app/` and opens `http://localhost:5500/index.html`, so the dashboard appears instead of a directory listing.

Opening `index.html` directly through `file://` shows an error message on purpose, because browsers block `fetch` there.

## Dashboard validation

All checks below were run on 2026-11-22 against the committed files.

| Check | Result |
| --- | --- |
| `python3 -m json.tool app/project-data.json` | Pass |
| `python3 -m json.tool .vscode/launch.json` | Pass |
| `app/index.html` contains `<title>Project Pulse</title>`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority` | Pass (all 7 found) |
| `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius`, `box-shadow`, `@media` | Pass (all 5 found) |
| `project-data.json` has a `projects` list; every project has non-empty `name`, `owner`, `status`, `recentActivity`, `priority` | Pass (5 projects, none missing a field) |
| `launch.json` has `Run Project Pulse Dashboard` with the expected command, `cwd` and `uriFormat` | Pass |
| Smoke test: `python3 -u -m http.server 5500` from `app/` | Pass: `index.html`, `styles.css`, `project-data.json` all HTTP 200; page contains "Project Pulse"; log matched `Serving HTTP on .* port ([0-9]+)`; server stopped and port 5500 free afterward |
| Inline script run in Node v22.15.0 with a minimal DOM stub | Pass: 5 `.project-card` elements, 4 with a summary, loading message hidden, no `undefined` text |

**Not yet verified**

- Running the launch configuration in VS Code and the browser opening automatically.
- Visual check of the responsive layout at 320px, 768px and 1280px.
- Browser contrast or Lighthouse accessibility check.

## Known limitations

- One sample project (Search Improvements) has no `summary`, so its card has no summary line. This is expected: `summary` is optional.
- Port 5500 must be free. If it is busy, the server fails and the browser will not open.
- Python can buffer output when it is not attached to a terminal. If the browser ever fails to open automatically, change the command to `python3 -u -m http.server 5500`.
- The page uses `replaceChildren`, which needs a modern browser.

## Learner handoff

- All agent work so far is committed and pushed through prompts you ran:
  - `e93184f Document the Project Pulse agent team`
  - `10d446d Plan the Project Pulse dashboard`
  - `f53c65a Build the Project Pulse dashboard`
- This file, `docs/final-handoff.md`, is new and not yet staged.

Next steps:

1. Run **Run Project Pulse Dashboard** and confirm the dashboard shows 5 project cards.
1. Stage, commit and push `docs/final-handoff.md`.
1. Optional: follow up on the plan's open questions, such as filtering or sorting cards by status or priority. You could also add a `summary` to Search Improvements.
