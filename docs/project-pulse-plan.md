
# Project Pulse Dashboard — Implementation Plan

## 1. Summary

Mona needs a small static dashboard (`app/index.html`, `app/styles.css`, `app/project-data.json`) plus a `.vscode/launch.json` that serves the app and opens the dashboard UI directly (never a directory listing). The work splits along the Designer/Coder boundary defined in `.github/agents/*.agent.md`:

- **Designer** owns the visual/UX layer: the HTML document structure and semantics of `app/index.html` (skeleton, headings, `.dashboard` container, the `<article class="project-card">` template markup, accessibility attributes) and all of `app/styles.css`.
- **Coder** owns the data/logic layer: `app/project-data.json`, a separate `app/app.js` that fetches and renders data into the DOM, and `.vscode/launch.json` (paired with a `.vscode/tasks.json` preLaunchTask that starts the static server).

**Key architectural decision (resolves the "inline script vs. separate file" question):** Coder's rendering logic lives in `app/app.js`, referenced by a single `<script src="app.js" defer></script>` tag that Designer adds once during the initial skeleton pass. This means Designer and Coder do **not** need to make repeated edits to the same region of `index.html` — Designer owns the full HTML file end-to-end (including the `<script>` tag reference and the `<template>` markup for one card), and Coder never edits `index.html` itself, only `app.js`, `project-data.json`, and the `.vscode/` files. This removes the file-overlap risk from the original plan and lets Designer's and Coder's work run fully in parallel.

## 2. Ordered implementation steps

1. **Step 0 — Confirm scaffold (either agent).** Verify `app/` and `.vscode/` directories exist. No conflict risk.
2. **Step 1 — HTML skeleton & contract (Designer).** Create `app/index.html`:
   - `<!DOCTYPE html>`, `<html lang="en">`, `<meta charset>` and viewport meta.
   - `<title>Project Pulse</title>` and `<link rel="stylesheet" href="styles.css">`.
   - A header with an `<h1>Project Pulse</h1>` and short tagline.
   - A `<main class="dashboard" id="dashboard">` container (this is the single required rendering target).
   - A `<template id="project-card-template">` containing one `<article class="project-card">` with child elements for name, owner, status badge, recent activity, and priority badge (e.g., `.project-card__name`, `.project-card__owner`, `.status-badge`, `.project-card__activity`, `.priority-badge`).
   - A documented "contract" (HTML comment or, preferably, this plan doc) telling Coder exactly which classes/attributes to populate and which normalized token format to use for `data-status`/`data-priority` (lowercase, hyphenated — e.g., `"At Risk"` → `at-risk`).
   - A `<script src="app.js" defer></script>` tag at the end of `<body>` (added once, not touched again).
   - A `<p class="loading-state">Loading projects…</p>` fallback node inside `.dashboard` for the pre-render/error state.
3. **Step 2 — Visual design (Designer, can run in parallel with Step 1 or immediately after).** Create `app/styles.css`:
   - `.dashboard` — responsive grid/flex layout (e.g., CSS grid with `repeat(auto-fill, minmax(260px, 1fr))`).
   - `.project-card` — `border-radius`, `box-shadow`, padding, background, hover/focus affordances.
   - `.status-badge[data-status="..."]` and `.priority-badge[data-priority="..."]` — distinct colors per value, sufficient contrast, plus a text label (not color alone).
   - Responsive breakpoints (single-column on narrow viewports), readable typography, spacing scale.
   - `.loading-state` / `.empty-state` / `.error-state` visual treatment for Coder's fallback messages.
4. **Step 3 — Sample data (Coder, fully parallel with Steps 1–2).** Create `app/project-data.json` with a top-level `projects` array; each object has `name`, `owner`, `status`, `recentActivity`, `priority`. Use varied, realistic values across multiple statuses and priorities so the Designer's badge CSS has real cases to style against.
5. **Step 4 — Data fetch & render logic (Coder).** Create `app/app.js`:
   - `fetch('./project-data.json')`, parse JSON, handle network/parse errors explicitly (visible error message, not console-only).
   - Read the `#project-card-template` from the DOM, clone it once per project, populate the child elements/badges, normalize `status`/`priority` into lowercase-hyphenated tokens for `data-status`/`data-priority` attributes and badge text.
   - Handle an empty `projects` array (render an explicit "No projects found" message).
   - Defensively handle missing/undefined fields on any project object (fallback text rather than throwing).
   - Append all cards into `#dashboard`, replacing the loading-state placeholder.
   - **Dependency:** requires Designer's Step 1 skeleton (template structure, container id/class, class-name contract) to exist first, but does not require editing `index.html` itself.
6. **Step 5 — Launch & task configuration (Coder).** Create/confirm:
   - `.vscode/tasks.json` — a task (e.g., `"Start Project Pulse Server"`) running `python3 -m http.server 5500` with `"options": {"cwd": "${workspaceFolder}/app"}`, `"isBackground": true`, and a background problem matcher with `beginsPattern`/`endsPattern` matching the server's `"Serving HTTP on ..."` output (needed so `serverReadyAction` can detect readiness).
   - `.vscode/launch.json` — strict JSON, no comments, one configuration named exactly `"Run Project Pulse Dashboard"`, `"type": "chrome"`, `"request": "launch"`, `"preLaunchTask"` pointing at the task above, `"webRoot": "${workspaceFolder}/app"`, and `"serverReadyAction"` with `"uriFormat": "http://localhost:%s/index.html"` and `"action": "openExternally"` (explicit `/index.html`, never a bare port URL).
7. **Step 6 — Integration pass (either agent).** Confirm all relative paths in `index.html` (`styles.css`, `project-data.json` via `app.js`, `app.js` itself) resolve correctly given the server's document root is `app/` (no leading `/app/` prefixes).
8. **Step 7 — Validation (learner or either agent).** Run through the Validation Expectations checklist below.

## 3. File assignments

| File | Owner | Action |
|---|---|---|
| `app/index.html` | **Designer** | Create/own fully — structure, semantics, `.dashboard` container, `<template>` markup, single `<script src="app.js">` reference. Coder does not edit this file. |
| `app/styles.css` | **Designer** | Create/own fully — `.dashboard`, `.project-card`, badges, responsive layout, loading/empty/error states. |
| `app/project-data.json` | **Coder** | Create/own fully — top-level `projects` array, five required fields per entry. |
| `app/app.js` | **Coder** | Create/own fully — fetch, render, error/empty handling, normalization of status/priority tokens. |
| `.vscode/tasks.json` | **Coder** | Create/own fully — background task that starts `python3 -m http.server 5500` with `cwd = ${workspaceFolder}/app`. |
| `.vscode/launch.json` | **Coder** | Create/own fully — strict JSON, `"Run Project Pulse Dashboard"` configuration, `preLaunchTask` + `serverReadyAction` opening `index.html`. |

`docs/agent-team.md` must not be modified by this work.

## 4. Designer responsibilities (detailed)

- Own `app/index.html` end-to-end: valid `<!DOCTYPE html>`, `<title>Project Pulse</title>`, `<link rel="stylesheet" href="styles.css">`.
- Define the rendering contract Coder will consume: a `.dashboard`/`#dashboard` container, a `<template>` with one `.project-card` article containing named sub-elements for name/owner/status/recentActivity/priority, and the exact class names/attribute names Coder must target (document this clearly, e.g., in an HTML comment and/or by cross-referencing this plan).
- Add the single `<script src="app.js" defer></script>` tag so Coder never needs to edit `index.html`.
- Include a visible loading-state placeholder and ensure semantic structure (heading levels, `<header>`/`<main>`/`<footer>` landmarks, `aria-live="polite"` on the dashboard region so dynamically injected cards are announced).
- Author `app/styles.css`: responsive `.dashboard` grid/flex layout; `.project-card` with `border-radius`, `box-shadow`, padding, and clear visual polish; status badges color-coded per `data-status` value with paired text labels (not color-only signaling) for accessibility; priority treatment that's visually distinct at a glance (e.g., left-border accent, icon, weight); responsive breakpoints down to narrow/mobile widths; sufficient color contrast throughout.
- Style the loading/empty/error states Coder's script will render.
- Report design decisions, tradeoffs, and files touched.

## 5. Coder responsibilities (detailed)

- Create `app/project-data.json` with a top-level `projects` array; every entry includes `name`, `owner`, `status`, `recentActivity`, `priority` with realistic, varied values (consistent enums recommended: status ∈ {Active, At Risk, Blocked, Completed}; priority ∈ {High, Medium, Low}) so Designer's badge selectors have full coverage.
- Create `app/app.js`:
  - Fetch `./project-data.json`; explicitly catch and surface fetch/parse failures as a visible message (not just a console error) — important because opening `index.html` via `file://` will fail fetch, reinforcing why the launch config/server is required.
  - Clone the `#project-card-template` once per project, populate its fields, normalize `status`/`priority` strings into lowercase-hyphenated tokens for `data-status`/`data-priority` attributes (e.g., `"At Risk"` → `at-risk`) so they match Designer's CSS attribute selectors exactly.
  - Handle an empty `projects` array and any missing/undefined project fields defensively (fallback text, never throw and blank the page).
  - Append rendered cards into `#dashboard`, removing/replacing the loading-state node.
  - Do not edit `app/index.html` or `app/styles.css`.
- Create `.vscode/tasks.json` task that runs `python3 -m http.server 5500` with `cwd = ${workspaceFolder}/app`, `isBackground: true`, and a background problem matcher whose `beginsPattern`/`endsPattern` match the server's startup log line so `serverReadyAction` can detect readiness.
- Create `.vscode/launch.json`:
  - Strict JSON: no comments, no trailing commas.
  - Exactly one configuration named `"Run Project Pulse Dashboard"`.
  - `"type": "chrome"`, `"request": "launch"`, `"preLaunchTask"` referencing the tasks.json label, `"webRoot": "${workspaceFolder}/app"`.
  - `"serverReadyAction"` with a `pattern` matching the server log, `"uriFormat": "http://localhost:%s/index.html"`, `"action": "openExternally"` — must always resolve to `index.html`, never a bare directory URL.
- Stay within assigned files; do not modify `app/index.html` or `app/styles.css`.
- Report what changed, what was validated, and any remaining risk.

## 6. Dependencies between steps

- Step 1 (Designer's HTML skeleton/contract) → Step 4 (Coder's `app.js`): Coder needs the template structure and class/attribute contract before writing render logic. This is a **soft dependency on the contract being defined**, not a file-edit dependency — Coder can start writing `app.js` against the agreed contract even slightly before Designer finishes final wording/copy, but final wiring must be validated against the actual DOM structure Designer ships.
- Step 3 (Coder's JSON data) has no dependency on Steps 1–2; the schema is fixed by the brief and can be authored immediately.
- Step 4 depends on Step 1 (template/contract) and benefits from Step 3 (real sample data) existing to test against, but does not require Step 2 (CSS) to be finished.
- Step 5 (`tasks.json` + `launch.json`) depends only on the final file layout (`app/index.html` as the entry point, `app/` as server root) — can be authored in parallel with everything else.
- Step 6 (integration/relative-path check) depends on Steps 1, 3, 4, 5 all being complete.
- Step 7 (validation) is last, after everything else.

## 7. Parallel work decisions

**Can run in parallel (no file overlap, no hard data dependency):**
- Designer's Step 1 (`index.html`) and Step 2 (`styles.css`) — both Designer-owned, no cross-agent conflict.
- Coder's Step 3 (`project-data.json`), Step 4 (`app.js` — once the contract from Step 1 is agreed), and Step 5 (`tasks.json`/`launch.json`).
- Designer's entire track (`index.html` + `styles.css`) and Coder's entire track (`project-data.json` + `app.js` + `.vscode/*`) can run **fully in parallel** because, by design, `index.html` is touched only by Designer and `app.js` is a separate file touched only by Coder. This is the key improvement over a design where Coder must also edit `index.html` inline.

**Must run sequentially / needs a checkpoint:**
- The **contract handoff**: Coder should not finalize `app.js`'s selector logic until Designer has committed to the template's class names and `#dashboard` id. In practice, run Designer's Step 1 first (or have the Orchestrator distribute the agreed contract from this plan to both agents up front) so Coder isn't guessing.
- Step 6 (integration/relative-path validation) must run after all files exist, since it checks cross-file references.
- Step 7 (final validation) is last.

**Rationale:** Per the Orchestrator's rules ("Keep overlapping file scopes in separate phases," "Run tasks in parallel only when file scopes do not overlap and there are no data dependencies"), splitting `app.js` out from `index.html` eliminates the one shared-file bottleneck from a naive plan and lets Designer and Coder work simultaneously once the contract (container id, template class names, normalized status/priority token format) is fixed — which this plan fixes explicitly in Step 1/Section 4 so no back-and-forth is needed.

## 8. Edge cases to handle

- **Empty `projects` array:** `app.js` must render a visible "No projects found" message rather than an empty `.dashboard`.
- **Malformed JSON / fetch failure:** `app.js` must catch and show a visible error state (styled via Designer's `.error-state` class), not fail silently or only log to console.
- **`fetch()` over `file://`:** Opening `index.html` by double-click (no server) will fail fetch/CORS in most browsers — this is precisely why `launch.json` + `tasks.json` must start `python3 -m http.server`; the learner should be told not to test by opening the file directly.
- **Missing/undefined fields on a project object:** `app.js` should fall back to placeholder text (e.g., "Unknown") per field rather than throwing and aborting the whole render loop.
- **Status/priority casing/token mismatches:** JSON may contain `"At Risk"` (Title Case with a space) while CSS attribute selectors expect `at-risk`; Coder's normalization step (lowercase, replace spaces with hyphens) in `app.js` is the single source of truth — Designer's CSS must target the normalized token form consistently.
- **Port 5500 already in use:** `python3 -m http.server 5500` will fail to bind; this is a runtime risk to flag to the learner, not something to code around.
- **Directory listing fallback:** Both `launch.json`'s `serverReadyAction.uriFormat` and its top-level `url` must explicitly end in `/index.html`; never leave a bare `http://localhost:%s` or `http://localhost:5500` that resolves to a directory listing.
- **Relative path breakage:** All references in `index.html` (`styles.css`, `app.js`) and in `app.js` (`./project-data.json`) must be relative, not `/app/...`-prefixed, since the server's document root (`cwd`/`webRoot`) is `app/` itself.
- **Background task detection reliability:** The `tasks.json` problem matcher's `beginsPattern`/`endsPattern` must actually match Python's `http.server` startup line (`Serving HTTP on 0.0.0.0 port 5500 ...`) or `serverReadyAction` may never fire; verify this string exactly against the Python version in the Codespace.
- **Re-running the launch config:** If the server task is still running from a previous session, the preLaunchTask may fail to bind port 5500 again; note as a known rerun risk.

## 9. Validation expectations

- **`app/project-data.json`:** Parses cleanly (`python3 -m json.tool app/project-data.json` succeeds); top-level key is `projects` (array); every entry has non-empty `name`, `owner`, `status`, `recentActivity`, `priority`.
- **`app/index.html`:** Contains `<title>Project Pulse</title>`; contains `<link ... href="styles.css">`; contains `<script src="app.js" ...>`; contains a `.dashboard`/`#dashboard` container and a `<template>` with `.project-card` markup; after `app.js` runs, `document.querySelectorAll('.project-card').length` equals the number of entries in `projects`.
- **`app/styles.css`:** Contains a `.dashboard` selector and a `.project-card` selector; computed styles show `border-radius` and `box-shadow` on `.project-card`; layout reflows to single-column at a narrow viewport; status/priority badges show visually distinguishable colors with accompanying text (not color-only).
- **`app/app.js`:** Renders cards from live fetch data with no console errors on a happy path; renders a visible message for an empty `projects` array and for a simulated fetch failure (e.g., temporarily rename the JSON file to test); `data-status`/`data-priority` attribute values are lowercase-hyphenated and match the CSS selectors exactly.
- **`.vscode/tasks.json`:** Valid strict JSON; task starts `python3 -m http.server 5500` with `cwd` resolving to `${workspaceFolder}/app`; background problem matcher's patterns actually match the server's console output (verify by running the task manually and watching status).
- **`.vscode/launch.json`:** Valid strict JSON (`python3 -m json.tool .vscode/launch.json` succeeds, no comments/trailing commas); contains exactly one configuration with `"name": "Run Project Pulse Dashboard"`; `webRoot` is `${workspaceFolder}/app`; `serverReadyAction.uriFormat` ends in `/index.html`.
- **End-to-end:** Launching "Run Project Pulse Dashboard" from VS Code's Run and Debug panel starts the server, opens a browser tab showing rendered project cards (not a directory listing, not a blank page), with no console errors, correct data per card, responsive layout at multiple widths, and visually distinct status/priority treatments.

## 10. Open questions

- Should the launch configuration use VS Code's `"type": "chrome"` debugger (current approach, requires the Chrome/Edge debug extension or built-in support) versus a simpler `"type": "node"`/external-browser-opener approach? The existing `.vscode/launch.json` already uses `"type": "chrome"` with `preLaunchTask` + `serverReadyAction`, which is the pattern this plan adopts — confirm this matches what's available in the learner's Codespace environment (browser debug extension present) before relying on it as the sole verification method; if not available, a manual "open the printed URL" fallback should still work since `serverReadyAction` and the task's console output both surface `http://localhost:5500/index.html`.
- The brief mentions a "short contributor-friendly summary" as a desirable dashboard trait, but only five fields are strictly required in `project-data.json`. Recommend treating `recentActivity` as satisfying this need (as the existing sample data does) rather than adding a sixth field, to keep the schema minimal and match the required-field validation exactly — confirm this interpretation is acceptable rather than adding an optional `summary` field.
