# Project Pulse — Implementation Plan

## 1. Summary

Project Pulse is a small, framework-free static web app that gives Mona an at-a-glance dashboard of her team's projects: name, owner, status, priority, recent activity, and progress. It lives under `app/` with `index.html` as the entry point, is styled by `app/styles.css`, and is data-driven from `app/project-data.json`. A `.vscode/launch.json` "Run Project Pulse Dashboard" configuration opens `index.html` with `cwd` set to `${workspaceFolder}/app` so learners land on the dashboard UI (not a directory listing). Work is split into non-overlapping file scopes so the **Designer** (styling) and **Coder** (markup, data, launch config) can execute in parallel once a small CSS-hook contract is agreed.

## 2. Ordered implementation steps

| # | Step | Owner | File(s) touched |
|---|------|-------|-----------------|
| 1 | Agree the CSS-hook contract and card field contract (hook names: `.dashboard`, `.project-card`, `.status-badge`, `.priority`, `.progress`, `.owner`, `.recent-activity`; data fields: `id`, `name`, `owner`, `status`, `priority`, `progress`, `dueDate`, `recentActivity`, `summary`). Recorded inline in this plan; no files written. | Planner (already done here) | — |
| 2 | Author the semantic HTML skeleton with the agreed hooks and an inline render script that fetches `project-data.json` and renders cards (with a safe embedded fallback for `file://`). | Coder | `app/index.html` |
| 3 | Author the JSON data file with a top-level `projects` array matching the field contract. | Coder | `app/project-data.json` |
| 4 | Style the dashboard: layout grid, cards, badges, priority treatment, typography, responsive breakpoints, focus/contrast accessibility. | Designer | `app/styles.css` |
| 5 | Author `.vscode/launch.json` with a "Run Project Pulse Dashboard" configuration, strict JSON, `cwd` = `${workspaceFolder}/app`, opening `index.html`. | Coder | `.vscode/launch.json` |
| 6 | Validate: open via the launch config, confirm cards render, hooks present, no console errors, responsive at ~360px width. | Orchestrator + learner | — |

## 3. File assignments table

| File | Owner | Purpose / what to produce |
|------|-------|---------------------------|
| `app/index.html` | **Coder** | Semantic HTML5 skeleton: `<header>` with dashboard title, `<main class="dashboard">` containing a `<section>` list where each project renders as `<article class="project-card">` with child hooks `.status-badge`, `.priority`, `.owner`, `.progress`, `.recent-activity`. Includes a small inline `<script>` that `fetch()`es `./project-data.json`, falls back to an embedded copy if `fetch` fails (for `file://`), and renders cards. Links `styles.css`. Includes empty-state markup. |
| `app/styles.css` | **Designer** | All visual styling. Implements a responsive grid for `.dashboard`, card design for `.project-card` (rounded corners, subtle shadow, spacing), color-coded `.status-badge` variants (e.g., `--on-track`, `--at-risk`, `--blocked`, `--done`), priority treatment (`.priority.high/med/low`), progress bar visual, typography scale, focus rings, and media queries. No JS, no HTML. |
| `app/project-data.json` | **Coder** | Strict JSON. Top-level `{ "projects": [ ... ] }`. Each project: `id`, `name`, `owner`, `status`, `priority`, `progress` (0–100), `dueDate` (ISO), `recentActivity`, `summary`. Provide 4–6 realistic sample projects covering each status and priority value. |
| `.vscode/launch.json` | **Coder** | Strict JSON, no comments. Configuration named `Run Project Pulse Dashboard`, `cwd` = `${workspaceFolder}/app`, opens `index.html` so the dashboard renders (not a directory listing). |

**Ownership rationale:** Designer owns only `styles.css` (pure visual concerns). Coder owns markup, data, and tooling — all runnable-app concerns per `coder.agent.md`. Scopes do not overlap, enabling parallel execution.

## 4. Designer responsibilities (`app/styles.css` only)

- **Overall dashboard look:** Clean, modern manager-facing dashboard; light theme with high-contrast text; consistent 8px spacing scale; page max-width and centered layout.
- **Project card design:** `.project-card` with rounded corners (~12px), soft shadow, internal padding, clear vertical hierarchy: name → owner/due → status/priority → progress → recent activity → summary.
- **Status badges:** `.status-badge` pill shape with color coding per status value (`on-track`, `at-risk`, `blocked`, `done`). Ensure WCAG AA contrast on badge text.
- **Priority treatment:** `.priority` variants (`high`, `medium`, `low`) using color + iconography/text so it's not color-only (accessibility).
- **Progress:** `.progress` bar with accessible label; visible track and fill.
- **Spacing & typography:** System font stack; readable line-height (~1.5); hierarchy via weight/size, not color alone.
- **Responsive behavior:** CSS grid `.dashboard` using `repeat(auto-fill, minmax(280px, 1fr))`; single column at ≤480px; comfortable tap targets.
- **Accessibility:** Visible `:focus-visible` outlines; text/background contrast ≥ 4.5:1; respect `prefers-reduced-motion`; do not rely on color alone for status/priority.
- **Required CSS hooks (contract):** `.dashboard`, `.project-card`, `.status-badge`, `.status-badge--<status>`, `.priority`, `.priority--<level>`, `.progress`, `.owner`, `.recent-activity`, `.empty-state`.

## 5. Coder responsibilities

### `app/index.html`
- Semantic HTML5: `<!doctype html>`, `<html lang="en">`, `<meta charset>`, `<meta name="viewport" content="width=device-width, initial-scale=1">`, `<title>Project Pulse</title>`, `<link rel="stylesheet" href="./styles.css">`.
- Header with app title/subtitle; `<main class="dashboard" aria-label="Project dashboard">`.
- Inline `<script>` renders cards from `./project-data.json` via `fetch()`, with try/catch that falls back to an embedded `const PROJECTS = [...]` copy if fetch fails under `file://`.
- Card template uses the CSS-hook contract exactly (see §4).
- Escapes user-visible strings to avoid HTML injection.
- Renders an `.empty-state` node when `projects.length === 0`.
- Applies `.status-badge--<status>` and `.priority--<level>` class modifiers.
- Includes an `aria-label` on the progress element and `datetime` on `dueDate`.

### `app/project-data.json`
- Strict JSON, top-level `projects` array (matches brief).
- Shape per project:
  ```
  {
    "id": "string",
    "name": "string",
    "owner": "string",
    "status": "on-track" | "at-risk" | "blocked" | "done",
    "priority": "high" | "medium" | "low",
    "progress": 0-100,
    "dueDate": "YYYY-MM-DD",
    "recentActivity": "string",
    "summary": "string"
  }
  ```
- 4–6 sample entries covering every status and priority.

### `.vscode/launch.json`
- Strict JSON, **no comments**.
- One configuration named exactly `Run Project Pulse Dashboard`.
- `cwd`: `${workspaceFolder}/app`.
- Opens `index.html` (not a directory) — use a browser-launch-capable configuration (e.g., built-in `chrome`/`msedge` debug type with `file` URL, or a task-backed launch) that displays the rendered dashboard.
- Deterministic name/paths so learners can invoke it from the Run panel.

## 6. Dependencies between steps

- Step 1 (hook + data contract) **must precede** Steps 2, 3, and 4.
- Step 2 (`index.html` skeleton with hooks) **must precede** Step 4 in the sense that Designer relies on the hook contract — but because the contract is fixed in Step 1, Designer can start in parallel with Step 2.
- Step 3 (`project-data.json` schema) **must precede** the render logic being usable, but the render logic in Step 2 is written against the schema fixed in Step 1, so 2 and 3 can proceed in parallel.
- Step 5 (`.vscode/launch.json`) is only meaningful once `app/index.html` exists (Step 2). It should be authored after or alongside Step 2, and validated only after Step 2 lands.
- Step 6 (validation) requires Steps 2–5 complete.

## 7. Parallel vs. sequential work

**Sequential:**
- Step 1 (contract) → everything else.
- Step 2 (`index.html`) → Step 5 validation of launch config.
- Steps 2–5 → Step 6 validation.

**Parallel (after Step 1):**
- Designer: Step 4 (`app/styles.css`).
- Coder: Steps 2 (`app/index.html`), 3 (`app/project-data.json`), 5 (`.vscode/launch.json`) — all Coder-owned but on distinct files; can be authored in one Coder phase.

Because file scopes do not overlap, Designer and Coder phases run concurrently with zero merge risk.

## 8. Edge cases to handle

- **Empty `projects` array** → render `.empty-state` message ("No projects yet").
- **Long project names / owner strings** → CSS truncation or wrapping without breaking card layout.
- **Missing optional fields** (`summary`, `recentActivity`, `dueDate`) → render gracefully; hide the field or show an em dash.
- **Unknown `status` / `priority` values** → fall back to a neutral badge/class and do not crash rendering.
- **`progress` out of range or non-numeric** → clamp to 0–100; default to 0.
- **`fetch()` fails under `file://`** → embedded fallback data path so the dashboard still renders when opened without a server.
- **Small screens (~360px)** → single-column grid, readable typography, tap-friendly spacing.
- **Keyboard/focus accessibility** → visible focus rings; logical tab order; cards not interactive unless made so (avoid focus traps).
- **HTML/JSON injection** → escape all interpolated strings.
- **Invalid JSON** in `project-data.json` → surface a clear console error and render the empty state.

## 9. Validation expectations

- `.vscode/launch.json` parses as strict JSON (`jq . .vscode/launch.json` succeeds); contains a configuration named `Run Project Pulse Dashboard` with `cwd` = `${workspaceFolder}/app` and opens `index.html`.
- Running that launch configuration displays the Project Pulse dashboard UI — not a directory listing.
- DOM contains at least one `.dashboard` element and one `.project-card` per project in `project-data.json`.
- All sample statuses and priorities render with the correct badge/priority classes.
- Layout is responsive: at ~360px width, cards stack in a single column; no horizontal overflow.
- Browser DevTools console shows no errors or unhandled promise rejections.
- Basic a11y: visible focus outlines; status/priority not conveyed by color alone; contrast passes AA on badges.
- Removing all entries from `projects` shows the empty state without errors.

## 10. Open questions

1. **Status vocabulary** — Confirm the exact set. Proposed: `on-track`, `at-risk`, `blocked`, `done`.
2. **Priority vocabulary** — Confirm. Proposed: `high`, `medium`, `low`.
3. **v1 scope** — Include filters/search/sort in v1, or defer to v2? Plan currently defers.
4. **Progress source** — Is `progress` a manual number, or derived? Plan assumes manual integer 0–100.
5. **Launch mechanism** — Preferred VS Code debug type for opening `index.html` (built-in browser debug vs. a task that starts a static server and then opens the URL). Both satisfy `cwd` = `${workspaceFolder}/app`; learner should confirm which is expected in this course environment.
6. **Theme** — Light-only for v1, or also dark-mode via `prefers-color-scheme`? Plan assumes light-only.
7. **Branding** — Any logo, color palette, or product voice to match? Plan assumes neutral modern styling.
