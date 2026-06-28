# AGENTS.md

Notes for anyone (human or agent) working on this repo.

## Project shape

Single-file vanilla-JS app: everything lives in `index.html` (HTML, CSS, and a `<script>`
block). No build step, no framework, no backend. Open `index.html` directly in a browser.

## Persistence system (read before adding fields)

User-entered values are persisted to `localStorage` so they survive page reloads/revisits.
There is **no API/backend** — the data is treated as non-sensitive.

- **Storage key:** `hynt-planner-state`
- **Stored value:** a JSON object mapping `inputId → value` (values are strings, exactly
  as read from `input.value`).
- **What gets persisted:** every `<input type="number">` whose `id` starts with **`inp-`**
  (the `PERSIST` selector in `index.html`).
- **What does NOT get persisted:** computed/display outputs, which use the `sum-*-disp`
  id pattern and are rewritten by `recalc()` via `setInp()` on every recalc.

### Adding a new field

- **To make a field persist: give it `<input type="number">` with an `id` starting
  `inp-`.** That's all — it's saved and restored automatically, no extra wiring.
- **Do not give a computed/display output an `inp-` id.** Use the `sum-*-disp` pattern so
  it stays excluded (otherwise stale derived values get stored).

### Editable vs. readonly / programmatic fields

- Editable inputs persist automatically via the `input` event listener (each keystroke
  saves just that one field).
- **`readonly` fields and any field set programmatically do NOT fire `input`.** Code that
  assigns to such a field's `.value` must call **`saveField(el)`** afterward to persist it.
  (This is the integration point for the planned future tab that populates the 7 readonly
  `inp-cost-*` / email fields — `inp-cost-acq`, `inp-cost-lead`, `inp-cost-value`,
  `inp-cost-google`, `inp-email-file`, `inp-emails-week`, `inp-income-1000`.)

### Lifecycle

- On load: `loadState()` restores saved values, then `recalc()`, then `saveAll()` seeds
  any persistable field not yet in storage (including the readonly ones on a first visit).
- Reset (`resetAll()`) restores defaults and clears the stored state (`hynt-planner-state`
  is removed) — reset means "start fresh".
- The only full scans (`loadState`, `saveAll`) run once on init; per-keystroke saves touch
  a single field.

### Implementation locations (in `index.html`)

- Helpers (`STORAGE_KEY`, `PERSIST`, `state`, `persist`, `saveField`, `saveAll`,
  `loadState`): just above `recalc()` (~line 2550).
- Input wiring (`input` → `recalc()` + `saveField`): ~line 2890.
- Reset (`resetAll`): ~line 2900.
- Init (`loadState` / `recalc` / `saveAll`): end of the script (~line 3290).
