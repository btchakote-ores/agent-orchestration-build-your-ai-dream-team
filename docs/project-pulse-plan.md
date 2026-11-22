# Project Pulse implementation plan

## Summary

Project Pulse is a small static dashboard for Mona's team. It shows each project as a card with its name, owner, status, recent activity, priority and a short summary. It has four files: `app/index.html`, `app/styles.css`, `app/project-data.json` and `.vscode/launch.json`. Learners start it from the VS Code **Run Project Pulse Dashboard** launch configuration. That configuration serves the `app/` folder with `python3 -m http.server 5500` and opens `http://localhost:5500/index.html`, so the dashboard appears instead of a directory listing.

**Research notes**

- `app/` is empty.
- `.vscode/` contains only `tasks.json`, which must not be changed.
- None of the four target files exist yet.
- `.github/workflows/3-step.yml` checks for these exact phrases:
  - `app/index.html`: `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority`
  - `app/styles.css`: `.dashboard`, `.project-card`, `border-radius`, `box-shadow`
  - `app/project-data.json`: `projects`, `name`, `owner`, `status`, `recentActivity`, `priority`; must parse with `python3 -m json.tool`
  - `.vscode/launch.json`: `Run Project Pulse Dashboard`, `index.html`; must parse with `python3 -m json.tool`
- Because the checks read `app/index.html` directly, the rendering script must be **inline in `app/index.html`**, not in a separate `app.js`. This also keeps the app to exactly the four assigned files.

## File assignments

| File | Owner | Purpose |
| --- | --- | --- |
| `app/index.html` | **Coder** | Page structure, "Project Pulse" title, link to `styles.css`, inline script that loads `project-data.json` and builds `.project-card` elements |
| `app/styles.css` | **Designer** | All visual design: `.dashboard` layout, `.project-card`, status badges, priority styling, responsive grid, focus states, rounded corners and shadows |
| `app/project-data.json` | **Coder** | Sample data with a top-level `projects` array |
| `.vscode/launch.json` | **Coder** | **Run Project Pulse Dashboard** configuration that serves `app/` and opens `index.html` |

This is the expected split, and the research supports it. `coder.agent.md` gives `.vscode/launch.json` and runnable-app work to the Coder. `designer.agent.md` gives styling and the `.dashboard` / `.project-card` hooks to the Designer. The Designer does not edit `app/index.html`. Markup or accessibility changes go to the Coder through the Orchestrator.

## Shared contracts (agree on these before Phase 1)

### Data schema (`app/project-data.json` → `app/index.html`)

```json
{
  "projects": [
    {
      "name": "Docs Refresh",
      "owner": "Mona",
      "status": "On track",
      "recentActivity": "Merged navigation redesign PR",
      "priority": "High",
      "summary": "Rewriting contributor docs so new folks ship in a day."
    }
  ]
}
```

- Required fields on every project: `name`, `owner`, `status`, `recentActivity`, `priority`.
- Optional field: `summary`.
- Allowed `status` values: `On track`, `At risk`, `Blocked`, `Completed`.
- Allowed `priority` values: `High`, `Medium`, `Low`.
- Include 4–6 projects that cover every status and every priority.

### CSS class contract (`app/index.html` ↔ `app/styles.css`)

| Hook | Element |
| --- | --- |
| `.dashboard` | `<main>` wrapper |
| `.dashboard-header`, `.dashboard-title`, `.dashboard-subtitle` | Header area with the `<h1>Project Pulse</h1>` |
| `.project-grid` | `<ul>` that holds the cards |
| `.project-card` | `<li><article>` for each project |
| `.project-name` | `<h2>` inside each card |
| `.project-owner`, `.project-summary`, `.project-activity` | Owner, summary and recent activity text |
| `.status-badge` + `.status--on-track` / `--at-risk` / `--blocked` / `--completed` / `--unknown` | Status badge |
| `.priority-badge` + `.priority--high` / `--medium` / `--low` / `--unknown` | Priority badge |
| `.dashboard-message` (+ `.dashboard-message--error` / `--empty`) | Loading, error and empty states |

**Converting values to class names:** lowercase the value and replace each run of non-letter/non-digit characters with `-`. For example, `On track` becomes `on-track`. Any value not in the allowed list gets `--unknown`.

### Launch contract (`.vscode/launch.json` → `app/index.html` location)

- Strict JSON: no comments and no trailing commas.
- `"version": "0.2.0"`.
- One configuration:
  - `"name": "Run Project Pulse Dashboard"`
  - `"type": "node-terminal"` (built into VS Code; no Python extension needed)
  - `"request": "launch"`
  - `"command": "python3 -m http.server 5500"`
  - `"cwd": "${workspaceFolder}/app"`
- `serverReadyAction`:
  - `"pattern": "Serving HTTP on .* port ([0-9]+)"`
  - `"uriFormat": "http://localhost:%s/index.html"`
  - `"action": "openExternally"`

## Designer responsibilities

- Write `app/styles.css` only, following the class contract.
- Make it look like a finished dashboard, not a plain page:
  - a header that makes the Project Pulse title obvious
  - a card grid with `border-radius`, `box-shadow`, readable spacing and clear type sizes
- Use CSS custom properties (`:root`) for colors, spacing, radius and shadow.
- Status badges must show the status as text, not only as color, and meet WCAG AA contrast (4.5:1 for text).
- Priority: high priority stands out (for example, an accent border plus a badge), and low priority is visually quieter.
- Responsive layout: `grid-template-columns: repeat(auto-fill, minmax(280px, 1fr))`, one column below about 600px, no horizontal scrolling at 320px.
- Add visible `:focus-visible` styles, support `prefers-reduced-motion`, and style the loading, error and empty message states.
- After Phase 1, review the rendered markup for heading order, landmarks and contrast. Send any HTML change requests through the Orchestrator.

## Coder responsibilities

- `app/index.html`:
  - semantic HTML5 with `lang="en"`, `<title>Project Pulse</title>` and `<link rel="stylesheet" href="styles.css">`
  - `<main class="dashboard">` with `<h1>Project Pulse</h1>`
  - an inline `<script>` that runs `fetch("project-data.json")`, checks `response.ok`, and builds `.project-card` elements showing `status`, `recentActivity`, `priority`, `owner`, `name` and `summary`
  - set values with `textContent`, never `innerHTML`, so data cannot inject HTML
  - put the cards in a list and give the list region `aria-live="polite"`
- `app/project-data.json`: valid JSON that follows the schema.
- `.vscode/launch.json`: follows the launch contract. Do not change `.vscode/tasks.json`.
- Validate before reporting (see Validation expectations).

## Ordered implementation steps

1. **Phase 0 – Confirm contracts (Orchestrator).** Send the data schema, class contract and launch contract above to both specialists. No files are written.
2. **Phase 1 – Build (runs in parallel).**
   - 2a. Coder writes `app/project-data.json`.
   - 2b. Coder writes `app/index.html`, using the data from 2a.
   - 2c. Coder writes `.vscode/launch.json`.
   - 2d. Designer writes `app/styles.css`.
3. **Phase 2 – Integration review (sequential).** The Designer reviews the rendered page and lists any accessibility or markup problems. The Orchestrator passes HTML fixes to the Coder and CSS fixes to the Designer. Only one agent edits a given file at a time.
4. **Phase 3 – Validation (Coder runs it, Orchestrator checks it).** Run the checks below, then report to the learner. The learner runs the launch configuration and handles all git operations.

## Dependencies

- `app/index.html` depends on the `app/project-data.json` **schema** (field names and allowed values) and on the CSS **class contract**.
- `app/styles.css` depends only on the class contract, not on the finished HTML.
- `.vscode/launch.json` depends on `app/index.html` being at `app/index.html` (`cwd` and `uriFormat` assume that location).
- Phase 2 depends on all Phase 1 files existing. Phase 3 depends on Phase 2.

## Parallel work decisions

- **Parallel:** the Designer (`app/styles.css`) and the Coder (`app/index.html`, `app/project-data.json`, `.vscode/launch.json`) work at the same time in Phase 1. Their files do not overlap, and the class and data contracts are fixed in Phase 0, so neither has to wait for the other's output.
- **Sequential within the Coder:** write `app/project-data.json` before or alongside `app/index.html` so the renderer is tested against real data. All three Coder files belong to one agent, so there are no conflicts.
- **Sequential:** Phase 2 fixes and Phase 3 validation run after Phase 1. Changing a contract mid-build forces both sides back through Phase 0.

## Edge cases

- **Opened via `file://`:** `fetch` is blocked. Show a `.dashboard-message--error` that says to use **Run Project Pulse Dashboard**. Do not show a blank page.
- **Fetch or parse failure, or HTTP status other than 200:** show a readable error message and log details to the console.
- **`projects` missing, not an array, or empty:** show the `.dashboard-message--empty` state.
- **Missing or empty field:** show "Not set" (or leave out the optional `summary`). Never render `undefined`.
- **Unknown `status` or `priority` value:** use `--unknown` styling and show the original text.
- **Long names or activity text:** wrap with `overflow-wrap: anywhere`. The layout must not break.
- **Port 5500 already in use:** the server fails and the browser does not open. Tell the learner to stop the old server.
- **Accessibility:** one `<h1>`, an `<h2>` per card, badges that show text (not color only), AA contrast, visible keyboard focus, and reduced-motion support.

## Validation expectations

- `python3 -m json.tool app/project-data.json` and `python3 -m json.tool .vscode/launch.json` both succeed.
- `app/index.html` contains `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity` and `priority`.
- `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius` and `box-shadow`.
- `.vscode/launch.json` contains `Run Project Pulse Dashboard`, `index.html` and `${workspaceFolder}/app`.
- Smoke test: in `app/`, run `python3 -m http.server 5500`, then:
  - `curl -s localhost:5500/index.html` returns HTML containing "Project Pulse"
  - `curl -s localhost:5500/project-data.json` returns the data
  - stop the server afterwards
- Manual check: **Run Project Pulse Dashboard** opens `/index.html` and shows the cards, not a directory listing.
- Responsive check at 320px, 768px and 1280px: no horizontal scrolling, and the grid reflows.
- Accessibility check: keyboard focus is visible, headings are in order, and badge contrast meets AA (verify in the browser's developer tools or with Lighthouse).
- Negative check: temporarily rename the data file (or open via `file://`) and confirm the error message appears.
- No agent stages, commits or pushes. `git status` shows only the four new files, and the learner commits them.

## Open questions

- Should `index.html` include an inline copy of the data so it still works over `file://`? Recommendation: no. It duplicates the data, and the launch configuration is the supported way to open the app.
- Should the dashboard have filtering or sorting (for example by status or priority)? Recommendation: leave it out for now.
- Is `node-terminal` acceptable for the launch type? It works without the Python extension. If the team prefers `debugpy`, only the Coder needs to know before Phase 1.
- Should the sample data use real team or project names, or placeholder names? Placeholder names are assumed.
