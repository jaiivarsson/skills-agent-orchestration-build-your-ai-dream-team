# Project Pulse — Implementation Plan

## 1. Summary

Project Pulse is a small static dashboard that helps Mona's contributors see, at a glance, which projects are active, who owns them, their current status, recent activity, and priority/risk level. It is built as three static files under `app/` (`index.html`, `styles.css`, `project-data.json`) plus a VS Code launch configuration (`.vscode/launch.json`) so learners can preview it with one click. This exercise is also a vehicle for practicing multi-agent orchestration: the Orchestrator delegates visual/UX decisions to Designer and implementation/data/launch work to Coder, based on the file-scope boundaries already codified in `.github/agents/coder.agent.md` and `.github/agents/designer.agent.md`.

The build must satisfy exact, machine-checked expectations (mirrored in `scripts/validate-exercise.sh` and `.github/workflows/3-step.yml`):

- `app/index.html` contains the exact title `Project Pulse`, links `styles.css`, references `project-data.json`, renders project cards using the class `project-card`, and displays each project's `status`, `recentActivity`, and `priority`.
- `app/styles.css` contains a `.dashboard` selector, a `.project-card` selector, `border-radius`, and `box-shadow` (plus responsive layout).
- `app/project-data.json` is valid JSON with a top-level `"projects"` key; each project object includes `name`, `owner`, `status`, `recentActivity`, and `priority`.
- `.vscode/launch.json` is strict JSON (no comments), contains a configuration named exactly `Run Project Pulse Dashboard`, serves from `${workspaceFolder}/app`, runs `python3 -m http.server 5500`, and uses a `serverReadyAction` pattern that opens `http://localhost:%s/index.html` (not a directory listing).

## 2. Ordered implementation steps

1. **Define the data contract** — Coder authors `app/project-data.json` first, since HTML rendering and (to a lesser extent) CSS badge/priority styling depend on knowing the field names and possible values (status values, priority values).
2. **Build the dashboard markup and data-loading logic** — Coder authors `app/index.html`, consuming `app/project-data.json` and referencing `app/styles.css`, rendering `.project-card` elements with visible `status`, `recentActivity`, and `priority`.
3. **Design and style the dashboard** — Designer authors `app/styles.css`, styling the `.dashboard` container, `.project-card` cards, status badges, priority treatment, and responsive layout, based on the markup structure and CSS hooks Coder committed to in step 2.
4. **Create the launch/preview configuration** — Coder authors `.vscode/launch.json`, pointing at `${workspaceFolder}/app`, running `python3 -m http.server 5500`, and opening `http://localhost:%s/index.html` via `serverReadyAction`.
5. **Integrated validation** — Orchestrator (or learner) opens the app via the launch configuration and confirms the dashboard renders correctly end-to-end.

Note: steps 1–2 are both Coder-owned and are naturally sequential (data shape before markup), but can be done as a single Coder work session/phase. Step 3 (Designer) depends on the HTML structure existing (specifically the `.dashboard`/`.project-card` class names Coder uses), so it should start only after step 2's markup skeleton is in place — or Coder and Designer must agree on the class-name contract (`.dashboard`, `.project-card`) up front so both can work with a shared markup skeleton in parallel. Step 4 (launch.json) has no dependency on file contents, only on the fixed directory path (`app/`) and filename (`index.html`), so it can run in parallel with steps 1–3.

## 3. File assignments

| File | Owner | Responsibility |
|---|---|---|
| `app/project-data.json` | **Coder** | Sample data: top-level `"projects"` array; each project has `name`, `owner`, `status`, `recentActivity`, `priority` (and optionally `progress` as an extra field for Designer's progress bar, if used). |
| `app/index.html` | **Coder** | Dashboard markup/structure: exact title `Project Pulse`, `<link>` to `styles.css`, script/fetch reference to `project-data.json`, renders `.project-card` elements showing `status`, `recentActivity`, `priority`; must use the `.dashboard` container class as the root wrapper Designer will style. |
| `app/styles.css` | **Designer** | Visual styling: `.dashboard` selector, `.project-card` selector, status badge styling, priority visual treatment (e.g., color-coded borders/pills), `border-radius`, `box-shadow`, responsive layout (media queries / flex or grid), readable spacing/typography. |
| `.vscode/launch.json` | **Coder** | Strict JSON, no comments. One configuration named exactly `Run Project Pulse Dashboard`, `cwd` set to `${workspaceFolder}/app`, command `python3 -m http.server 5500`, `serverReadyAction` pattern opening `http://localhost:%s/index.html`. |

This matches the established role split: `coder.agent.md` explicitly states Coder "may also create support configuration such as `.vscode/launch.json`" and gives the exact `cwd`/`index.html` guidance for Project Pulse; `designer.agent.md` explicitly owns "polished dashboard... visible project cards, status badges, clear priority treatment... deterministic CSS hooks such as `.dashboard` and `.project-card`."

## 4. Designer responsibilities (this task)

- Own `app/styles.css` only. Do not edit `app/index.html` or `app/project-data.json` unless the Orchestrator explicitly reassigns scope (e.g., if a class name needs to change, Designer should request Coder make the markup change rather than editing HTML directly).
- Implement `.dashboard` as the page/grid container selector.
- Implement `.project-card` with visual affordances: `border-radius`, `box-shadow`, padding/spacing, and card-based layout.
- Create status badges (e.g., pill-shaped elements with distinct colors per status such as "Active", "At Risk", "Blocked", "Completed").
- Create clear priority treatment (e.g., color-coded left border, icon, or badge distinguishing High/Medium/Low).
- Implement responsive layout so cards reflow into fewer columns on narrow viewports (flexbox/grid + media queries).
- Ensure sufficient color contrast and non-color-only status/priority indicators (add text labels, not just color) for accessibility.
- Keep typography readable (font sizing, line-height, hierarchy between project name, owner, and metadata).
- Report back: which selectors were added, what accessibility considerations were applied (contrast, text labels alongside color), and how responsiveness was implemented (breakpoints used).

## 5. Coder responsibilities (this task)

- Own `app/project-data.json`, `app/index.html`, and `.vscode/launch.json`.
- In `project-data.json`: produce a top-level `"projects"` array/key with several (recommend 4–6) sample projects, each with `name`, `owner`, `status`, `recentActivity`, `priority` as required fields, keeping value vocabularies small and consistent (e.g., status ∈ {Active, At Risk, Blocked, Completed}; priority ∈ {High, Medium, Low}) so Designer can build deterministic badge styling against a known, finite set of values.
- In `index.html`: use the exact title `Project Pulse` (in `<title>` and a visible heading), link `styles.css`, load `project-data.json` (via `fetch` for client-side rendering, since this is a static file served by `python3 -m http.server`), and render each project into an element with class `project-card`, showing `status`, `recentActivity`, and `priority` values as visible text. Provide a `.dashboard` wrapper element as the container Designer will style.
- Handle the no-JS/fetch-failure case gracefully (e.g., a visible fallback message) since `fetch` against `project-data.json` requires being served over HTTP (which the launch config provides) — opening `index.html` via `file://` will fail CORS/fetch in some browsers. This should be flagged to the Orchestrator as a known behavior, expected to work correctly only when launched via the provided server config.
- In `.vscode/launch.json`: write strict JSON (no trailing commas, no comments), add configuration named exactly `Run Project Pulse Dashboard`, `cwd`: `${workspaceFolder}/app`, command running `python3 -m http.server 5500`, and a `serverReadyAction` (or equivalent pattern-matching launch config compatible with the VS Code debug/task launcher being used) that opens `http://localhost:%s/index.html`.
- Validate JSON files parse (`python3 -m json.tool`) and that HTML has no unclosed tags before reporting completion.

## 6. Dependencies between steps/files

- `app/project-data.json` field names and value vocabulary must be finalized **before** `app/index.html` can render them, and **before** `app/styles.css` can build deterministic badge/priority styling keyed to specific values.
- `app/index.html`'s class names (`.dashboard`, `.project-card`) must be fixed **before** `app/styles.css` is written, since Designer styles by selector, not by owning markup. This is the primary cross-agent dependency — establish a locked contract early: root container class `dashboard`, per-project card class `project-card`.
- `.vscode/launch.json` depends only on the fixed structural facts already known (folder is `app/`, entry file is `index.html`, port 5500) — it has **no dependency** on the content of the other three files, only on their existence at the expected paths.
- Integrated validation (opening the dashboard) depends on all four files being complete and internally consistent.

## 7. Parallel vs. sequential work

**Can run in parallel (no file-scope overlap):**
- Coder building `.vscode/launch.json` can run in parallel with Coder building `app/project-data.json`/`app/index.html`, or even be done first/independently, since it has no content dependency — only needs the fixed `app/` + `index.html` + port 5500 facts, which are already known upfront.
- Once the markup skeleton and class-name contract (`.dashboard`, `.project-card`) are agreed/fixed, Designer can begin `app/styles.css` while Coder still refines data-rendering logic in `index.html`, as long as Coder does not rename the agreed classes without notifying Designer.

**Must run sequentially:**
- `app/project-data.json` → `app/index.html`: field names/values must exist before HTML can reference them (both are Coder-owned, so this is an internal sequencing note, not a cross-agent handoff).
- `app/index.html` (markup/class contract) → `app/styles.css`: Designer needs the class names before writing selectors that must match exactly (`.dashboard`, `.project-card`). Recommend the Orchestrator have Coder produce/lock the markup skeleton (even before full data-binding logic is finished) and share the class-name contract with Designer before Designer's phase starts, to avoid rework.
- Because Coder owns 3 of 4 files and Designer owns 1, the practical phase order Orchestrator should use is: Phase 1 (Coder: data + markup + launch.json) → Phase 2 (Designer: styles.css) → Phase 3 (integrated validation). Running Designer fully in parallel from the start risks Designer guessing at class names that Coder later changes.

## 8. Edge cases to handle

- **Empty or missing `projects` array** — `index.html` rendering logic should not throw; show a "No projects to display" state inside `.dashboard` rather than a blank page.
- **Fetch failure when opened via `file://`** — since `index.html` uses `fetch('project-data.json')`, opening the file directly (not via the launch server) will fail in most browsers due to CORS restrictions on local file access. Document this clearly; the dashboard must be run via the `Run Project Pulse Dashboard` launch config for correct behavior.
- **Long project names / owner names** — CSS must handle text overflow (wrapping or ellipsis) so cards don't break layout.
- **Many projects (10+)** — grid/flex layout must reflow without breaking; consider max-width constraints per card.
- **Unexpected/unstyled status or priority values** — if `project-data.json` contains a status/priority value not covered by CSS badge classes, styling should degrade gracefully (e.g., a default/neutral badge style) rather than showing unstyled text.
- **Accessibility of status/priority indicators** — status and priority must not be conveyed by color alone; include visible text labels and ensure adequate contrast ratios (WCAG AA) for badge text/background combinations.
- **Duplicate or missing required fields** — if a project is missing `owner` or `recentActivity`, the UI should show a placeholder (e.g., "Unknown") instead of `undefined` or blank rendering artifacts.
- **JSON validity** — trailing commas or comments in `project-data.json` or `launch.json` will break `python3 -m json.tool` / VS Code launch parsing; both must be strict JSON.
- **Port conflicts** — `python3 -m http.server 5500` may fail if port 5500 is already in use in the Codespace; this is a known constraint of the fixed, deterministic port requirement and should be surfaced as a runtime risk, not something to silently change (the exact port value is baked into validation).
- **Responsive breakpoints colliding with card min-widths** — test at narrow widths (e.g., 320–375px) to confirm cards stack to a single column without horizontal scroll.

## 9. Validation expectations

The Orchestrator/learner should verify:

1. **Static checks** (matching `scripts/validate-exercise.sh` / step 3 workflow expectations):
   - `app/index.html` contains the string `Project Pulse`, a `<link>`/reference to `styles.css`, a reference to `project-data.json`, and elements with class `project-card`; visibly shows `status`, `recentActivity`, and `priority` values.
   - `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius`, and `box-shadow`.
   - `app/project-data.json` parses via `python3 -m json.tool` and contains a top-level `"projects"` key; each project has `name`, `owner`, `status`, `recentActivity`, `priority`.
   - `.vscode/launch.json` parses via `python3 -m json.tool`, contains the literal name `Run Project Pulse Dashboard`, references `app` as the working directory, runs `python3 -m http.server 5500`, and opens `http://localhost:%s/index.html`.
2. **Runtime check**: Open Run and Debug in VS Code, select `Run Project Pulse Dashboard`, launch it, and confirm the browser opens `http://localhost:5500/index.html` showing the rendered Project Pulse dashboard (not a directory listing), with visible cards for each sample project, correct status/priority/recentActivity text, and polished card styling (rounded corners, shadows). Stop the server afterward.
3. **Responsive check**: Resize the browser/preview to a narrow width and confirm cards reflow to a single column without overflow or broken layout.
4. **Accessibility spot-check**: Confirm status/priority are legible without relying solely on color (visible text labels present), and text contrast is readable against badge backgrounds.

## 10. Open questions

- Should `progress` (percentage complete) be included as an extra data field per the task's mention of "progress" in sample data, even though the brief/step-3 prompt only strictly requires `name`, `owner`, `status`, `recentActivity`, `priority`? Recommend including it as an optional bonus field Coder may add, with Designer optionally rendering a progress bar — but it is not part of the validated/required contract, so it should not replace or complicate the five required fields.
- Should `index.html` fetch data client-side (`fetch`) or should Coder consider inlining data as a `<script>` JSON blob to avoid the `file://` CORS caveat when learners double-click the HTML file directly instead of using the launch config? The step 3 prompt and required launch config strongly imply the intended run path is via the server, so `fetch` is the expected approach — but this should be confirmed with the Orchestrator since it affects whether opening the file directly (bypassing the launch config) will silently show a broken/empty dashboard.
- Exact status/priority vocabulary (e.g., "At Risk" vs "AtRisk", "High/Medium/Low" vs numeric) is not mandated by validation beyond field presence — Coder should pick a small, finite, consistent set and communicate it to Designer before Designer builds badge color-coding, to avoid rework if Designer assumes different values.
- Whether VS Code's `launch.json` schema in this environment supports `serverReadyAction` for a plain `python3 -m http.server` (a non-debug task) or whether a `tasks.json`-driven pre-launch task plus a `chrome`/`edge` launch type is needed to actually trigger `serverReadyAction`. This is an environment/tooling detail Coder should verify against the VS Code debug configuration schema available in this devcontainer before finalizing `launch.json`, since `serverReadyAction` is typically documented for debug configurations with a `program`/`request` type, not a bare shell command — Coder may need to use a `node`/`pwa-node`-agnostic wrapper or a `"type": "node-terminal"`/`"cppvsdbg"`-independent approach; the safest determinism is to match exactly what `scripts/validate-exercise.sh` checks (command string and URL pattern), even if the "run" experience needs a companion task in `.vscode/tasks.json` to actually start the server (worth confirming this repo's existing `.vscode/tasks.json` conventions before writing `launch.json`).
