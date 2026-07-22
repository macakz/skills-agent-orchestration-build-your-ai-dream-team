# Project Pulse Dashboard — Implementation Plan

## 1. Summary

Mona needs a small static dashboard (`app/index.html`, `app/styles.css`, `app/project-data.json`) plus a `.vscode/launch.json` that serves the app and opens the dashboard UI (not a directory listing). Work splits cleanly along the Designer/Coder boundary defined in `.github/agents/*.agent.md`:

- **Designer** owns the visual/UX layer: the HTML structure and semantics of `app/index.html` (markup skeleton, card template, accessibility) and all of `app/styles.css`.
- **Coder** owns the data and logic layer: `app/project-data.json`, the JavaScript that fetches/renders data into the HTML (wiring), and `.vscode/launch.json`.

Because both Designer and Coder touch `app/index.html` (structure vs. wiring), that file must be handled in two **sequential** sub-phases, not in parallel, to avoid overlapping edits and merge conflicts. `app/styles.css`, `app/project-data.json`, and `.vscode/launch.json` have no cross-owner overlap and can be built in parallel with each other once the HTML skeleton exists.

## 2. Ordered implementation steps

1. **Step 0 — Confirm scaffold (Coder or Designer, whoever starts first):** Verify `app/` and `.vscode/` exist (they do, currently empty of these files). No conflict risk; either agent can create the `app/` files as needed.
2. **Step 1 — HTML skeleton & semantics (Designer):** Create `app/index.html` with `<title>Project Pulse</title>`, `<link>` to `styles.css`, a `.dashboard` container element, a placeholder/template area for `.project-card` elements, semantic landmarks (header, main, cards as `<article>` or similar), and accessible markup (headings, ARIA labels for status/priority where useful). Include a `<script>` tag reference (e.g., `<script src="app.js" defer></script>` or inline `<script>` block) as a hook point, but leave data-fetch logic to Coder. Reference `project-data.json` conceptually in a comment/placeholder so Coder knows where to wire it.
3. **Step 2 — Visual design (Designer):** Create `app/styles.css` with `.dashboard` (grid/flex responsive layout) and `.project-card` (border-radius, box-shadow, padding) selectors, status badge styles (color-coded by status value), priority treatment (e.g., color/icon/border-left accent), responsive breakpoints, readable typography and spacing.
4. **Step 3 — Sample data (Coder):** Create `app/project-data.json` with top-level `projects` array; each object has `name`, `owner`, `status`, `recentActivity`, `priority` (and optionally a short summary field per the brief's "contributor-friendly summary," if included, must not break the required-field validation). Use realistic, varied sample data (multiple statuses/priorities) to exercise styling.
5. **Step 4 — Wire data to markup (Coder, sequential after Step 1):** Add JS (inline `<script>` in `index.html` or a separate `app.js` referenced from `index.html`) that fetches `project-data.json` (`fetch('./project-data.json')`), parses it, and dynamically creates one `.project-card` element per project inside `.dashboard`, populating name, owner, status, recentActivity, priority into the DOM using classes/structure the Designer defined in Step 1. This step **modifies `app/index.html` again** (adds/edits `<script>` content) — must happen after Designer's Step 1 skeleton is in place, and Designer should not edit the same script region concurrently.
6. **Step 5 — Launch configuration (Coder):** Create `.vscode/launch.json` (strict JSON, no comments) with a configuration named exactly `"Run Project Pulse Dashboard"`, using a server task or `preLaunchTask`/compound approach to run `python3 -m http.server 5500` with `cwd` = `${workspaceFolder}/app`, and a `serverReadyAction` that opens `http://localhost:%s/index.html` in the browser (not the bare root URL, to avoid a directory listing).
7. **Step 6 — Integration pass (Coder + Designer as needed):** Confirm HTML references resolve correctly relative to `app/` as server root (e.g., `styles.css`, `project-data.json`, `app.js` all referenced with relative paths, no `/app/` prefix, since the server's cwd is `app/`).
8. **Step 7 — Validation (either agent, or learner):** Manually verify JSON validity, card rendering, and launch behavior per Validation Expectations below.

## 3. File assignments

| File | Owner | Action |
|---|---|---|
| `app/index.html` | **Designer** (structure/semantics, Step 1) then **Coder** (data-wiring script, Step 4) | Create then edit — sequential, same file, two distinct phases |
| `app/styles.css` | **Designer** | Create (Step 2) |
| `app/project-data.json` | **Coder** | Create (Step 3) |
| `.vscode/launch.json` | **Coder** | Create (Step 5) — strict JSON, no comments |

No other files should be touched by either agent for this exercise. `docs/agent-team.md` must not be overwritten by this work.

## 4. Designer responsibilities (detailed)

- Author `app/index.html`'s document structure: `<!DOCTYPE html>`, `<head>` with `<title>Project Pulse</title>` and `<link rel="stylesheet" href="styles.css">`, `<body>` containing a `.dashboard` root container and a clear placeholder location where project cards will be injected (e.g., an empty `<div class="dashboard" id="dashboard"></div>` or a `<template>` for one `.project-card`).
- Ensure information hierarchy: dashboard title/heading visible, each card clearly presents name, owner, status, recentActivity, priority in a scannable order.
- Design `app/styles.css`: `.dashboard` as a responsive grid/flexbox (e.g., `display:grid; grid-template-columns: repeat(auto-fill, minmax(260px,1fr)); gap:1rem;`), `.project-card` with `border-radius`, `box-shadow`, padding, background/border for polish.
- Status badges: distinct visual treatment per status value (e.g., `.status-active`, `.status-blocked`, `.status-completed`) with color-coding and sufficient contrast for accessibility.
- Priority treatment: visually distinct (e.g., left border accent color, icon, or bold label) so high-priority projects stand out at a glance.
- Responsive behavior: layout adapts from multi-column to single-column on narrow viewports; verify with browser resize or dev tools mobile emulation.
- Accessibility: sufficient color contrast for badges/text, semantic heading levels, avoid color-only signaling (pair color with text label like "Status: Active").
- Explain design tradeoffs and report files touched per Designer agent rules.

## 5. Coder responsibilities (detailed)

- Create `app/project-data.json` with a top-level `projects` array; each entry has all five required fields (`name`, `owner`, `status`, `recentActivity`, `priority`) with realistic string values (status/priority as consistent enums, e.g., `"Active"|"At Risk"|"Blocked"|"Completed"` and `"High"|"Medium"|"Low"`) so Designer's badge/priority CSS selectors have consistent hooks to target (e.g., via `data-status` / `data-priority` attributes or class names generated from status value).
- Add data-fetch/render logic to `app/index.html` (inline script or separate `app.js` file referenced from index.html) that:
  - Fetches `project-data.json` via `fetch()`.
  - Handles fetch/parse errors explicitly (e.g., catch block that renders a visible error message instead of failing silently) — important since `file://` fetch will fail without a server, reinforcing the need for the launch config.
  - Iterates `projects` array and creates one `.project-card` element per project, setting text content for name, owner, status, recentActivity, priority, and adding a data attribute or class reflecting status/priority for the Designer's CSS hooks (e.g., `card.dataset.status = project.status`).
  - Appends cards into the `.dashboard` container defined by Designer.
- Create `.vscode/launch.json`:
  - Strict JSON syntax, no comments, no trailing commas.
  - One configuration object named exactly `"Run Project Pulse Dashboard"`.
  - Type/request appropriate for launching a browser against a local static server (VS Code's built-in browser debug type, e.g., `"type": "pwa-chrome"` or `"chrome"`, or a compound with a task that starts `python3 -m http.server 5500`).
  - Since `launch.json` alone can't start a Python server, pair it with a `preLaunchTask` (defined in `tasks.json`) that runs `python3 -m http.server 5500` with `"options": {"cwd": "${workspaceFolder}/app"}`, and the launch config includes `"serverReadyAction"` with `"pattern"`, `"uriFormat": "http://localhost:%s/index.html"`, `"action": "openExternally"`.
  - Verify `serverReadyAction` opens `.../index.html` explicitly, never a bare port URL that would resolve to a directory listing.
  - Stay within assigned scope; do not modify `app/styles.css` or restructure Designer's HTML.

## 6. Dependencies between steps

- Step 1 (HTML skeleton) → must precede Step 4 (data-wiring script), since Coder's script needs the `.dashboard` container and card class conventions Designer establishes.
- Step 2 (CSS) has no hard dependency on Step 1's completion content-wise, but benefits from knowing the class names/structure Designer itself is defining — since Designer owns both, this is naturally sequential within the Designer's own work, not a cross-agent blocker.
- Step 3 (JSON data) has no dependency on Steps 1–2; can start immediately and in parallel with Designer's work.
- Step 4 depends on Step 1 (HTML skeleton must exist) and Step 3 (data file must exist and its schema known) — Coder should have both before wiring.
- Step 5 (`launch.json`) depends only on knowing the final file layout (`app/index.html` as entry point) — can be authored in parallel with Steps 1–4, but final validation (Step 8) requires all files present.
- Step 6 (integration/relative-path check) depends on Steps 1, 3, 4, 5 all being complete.

## 7. Parallel work decisions

**Can run in parallel:**
- Designer's Step 1 (HTML skeleton) and Coder's Step 3 (`project-data.json`) — no file overlap, no data dependency for the JSON schema (schema is fixed by the brief).
- Designer's Step 2 (`app/styles.css`) and Coder's Step 5 (`.vscode/launch.json`) — completely separate files, no shared state.
- Coder's Step 3 (JSON) and Step 5 (launch.json) can both run alongside Designer's Steps 1–2.

**Must run sequentially:**
- Designer's Step 1 (HTML skeleton) → Coder's Step 4 (data-wiring script in `index.html`). This is the critical sequencing point: both agents touch `app/index.html`, and overlapping file scopes must be phased, not parallelized. Coder must wait for Designer to deliver the skeleton (container element, expected card class names) before wiring in the fetch/render logic, otherwise Coder risks guessing at class names or clobbering Designer's markup.
- Step 6 (integration pass checking relative paths) must run after all file creation steps complete, since it validates cross-file references.
- Step 8 (final validation) is sequential and last, after everything else.

**Rationale:** Per Orchestrator rules ("Keep overlapping file scopes in separate phases" and "Run tasks in parallel only when file scopes do not overlap and there are no data dependencies"), `index.html` is the single point of forced sequencing. All other files are independent and safely parallelizable.

## 8. Edge cases to handle

- **Empty or malformed `project-data.json`:** Coder's render script should handle an empty `projects` array (render "No projects found" message) and malformed JSON (catch parse errors, show a visible fallback message rather than a blank page).
- **`fetch()` over `file://`:** Opening `index.html` directly by double-click (no server) will fail CORS/fetch for local JSON in most browsers — this is exactly why `launch.json` must start an HTTP server; document this constraint so the learner doesn't test by opening the file directly.
- **Missing required fields in a project object:** Render script should defensively handle missing/undefined fields (e.g., fallback to "Unknown" or empty string) rather than throwing and breaking the whole render loop.
- **Status/priority value variety:** Ensure CSS selectors for badges cover the actual set of status/priority strings Coder puts in the JSON; mismatched casing (e.g., "active" vs "Active") between JSON values and CSS class-generation logic is a common bug — Coder should normalize (e.g., lowercase, replace spaces with hyphens) when generating class/data-attribute values, and Designer's CSS should target the normalized form.
- **Port conflicts on 5500:** If port 5500 is already in use in the Codespace, `python3 -m http.server 5500` will fail; note this as a runtime risk (not something to code around, just something the learner should know to check).
- **Directory listing fallback:** Always specify `/index.html` explicitly in the `serverReadyAction` URI to remove ambiguity and match the requirement of opening the dashboard frontend, not a directory listing.
- **Relative path breakage:** If `index.html`'s `<link>`/`<script>`/`fetch` paths use absolute paths like `/app/styles.css`, they will break because the server's document root is `app/` itself (cwd), not the repo root — all references must be relative (`styles.css`, `project-data.json`), not `/app/styles.css`.
- **JSON syntax strictness in `launch.json`:** No comments, no trailing commas — a common mistake when copying from JSONC-style VS Code examples; must be validated as strict JSON.

## 9. Validation expectations

- **`app/project-data.json`:** Parses successfully with `python3 -m json.tool app/project-data.json` or `JSON.parse` in browser console; confirm top-level key is `projects` (array); confirm every entry has `name`, `owner`, `status`, `recentActivity`, `priority` present and non-empty.
- **`app/index.html`:** Contains `<title>Project Pulse</title>`; contains a `<link ... href="styles.css">` (or equivalent relative reference); contains a script that fetches `project-data.json`; after running, DOM contains one or more elements with class `project-card`; each rendered card visibly displays status, recentActivity, and priority text content (verify via browser dev tools or a quick DOM query like `document.querySelectorAll('.project-card').length === projects.length`).
- **`app/styles.css`:** Contains a `.dashboard` selector and a `.project-card` selector; visually confirm border-radius and box-shadow render (via computed styles in dev tools); confirm layout reflows at a narrow viewport width (responsive check); confirm status badges have distinguishable colors per status.
- **`.vscode/launch.json`:** Valid strict JSON (parses with `python3 -m json.tool .vscode/launch.json` with no errors, no trailing commas/comments); contains a configuration with `"name": "Run Project Pulse Dashboard"`; `cwd` (directly or via paired task) resolves to `${workspaceFolder}/app`; `serverReadyAction.uriFormat` ends in `/index.html`; launching it from VS Code's Run and Debug panel opens a browser tab showing the rendered dashboard (cards visible), not a bare file/directory listing page.
- **End-to-end:** Launch via the configuration, confirm no console errors, confirm all project cards render with correct data, confirm responsive layout at multiple widths, confirm badges/priority treatments are visually distinct.

## 10. Open questions

- Should `.vscode/launch.json` use a `serverReadyAction` paired with a `preLaunchTask` in `tasks.json` (recommended, since `launch.json` alone cannot start `python3 -m http.server`), or should the Orchestrator/Coder instead rely on VS Code's Live Preview extension conventions? A `tasks.json` entry (new task, not touching the existing "Open Copilot CLI exercise terminal" task) will likely be needed alongside `launch.json` — confirm with the Orchestrator whether creating/editing `.vscode/tasks.json` is in Coder's scope or needs explicit assignment, since only `.vscode/launch.json` is named in the file assignments but a working launch typically requires both.
- Should the "short contributor-friendly summary" mentioned in the brief's bullet list be included as an extra JSON field (e.g., `summary`) even though it's not in the five required fields list? Recommend including it as an optional additional field for richer cards, but it's not required for validation pass/fail.
- Should the data-fetch script live inline in `index.html` or in a separate `app/app.js`? Recommend Coder use a separate `app/app.js` referenced via `<script src="app.js" defer></script>`, added once by Designer as an empty placeholder reference in Step 1, so Coder's Step 4 only creates/edits `app.js` (not `index.html` itself), reducing edit overlap risk on `index.html`. Confirm with the Orchestrator before delegating, as it would let Steps 1 and 4 run more independently, provided Designer includes the `<script src="app.js" defer>` tag as part of the initial skeleton.
