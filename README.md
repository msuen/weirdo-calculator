# Weirdo Pricing Calculator

A single-page pricing calculator for design and creative projects. Allocates a project value across cost categories (profit, tax, owner comp, overhead, team, expenses) and visualizes budget distribution in real time.

**Live:** [msuen.github.io/weirdo-calculator](https://msuen.github.io/weirdo-calculator/)

## How it works

The user sets a **project value**, then defines percentage-based allocations (profit, tax, owner comp) and line-item costs (contractors, expenses). The calculator shows how the budget breaks down and whether the project nets positive or negative.

All state is persisted in the URL hash — copy the URL to save/share a specific configuration. This makes it embeddable in Notion or similar tools via iframe.

---

## Architecture

Everything lives in a single `index.html` file — no build step, no dependencies.

| Layer | Approach |
|-------|----------|
| Markup | Semantic HTML with grid-based card layout |
| Styles | CSS custom properties, light/dark mode via `prefers-color-scheme` |
| Logic | Vanilla JS — render functions, calculation pipeline, URL hash state |
| Hosting | GitHub Pages (static) |

---

## Data model

Three arrays store line items. Each row object has the same shape:

```js
{ name: string, rate: number, qty: number, unit: string, on: boolean }
```

- `unit` — one of `'hour'`, `'day'`, `'week'`, `'month'`, `'flat fee'`
- `on` — toggle for active/inactive (inactive rows excluded from totals)

| Array | Section | Description |
|-------|---------|-------------|
| `fohRows` | Front of House (Contractors) | Team members with rates |
| `ohRows` | Back of House (Expenses) | Operational overhead costs |
| `fohExpRows` | Front of House (Expenses) | Project-specific expenses |

Top-level inputs stored as form values:
- **Project value** — editable dollar amount
- **Net profit %** — percentage of project value
- **Tax %** — applied to total project value
- **Owner comp %** — percentage of project value

---

## Render pipeline

Each section has an independent render function (`renderFoh`, `renderOh`, `renderFohExp`) that:

1. Maps its array to template HTML with inline event handlers
2. Toggles the `.empty` class on the card body (hides content when no rows)
3. Calls `recalc()` after any change

`recalc()` is the central calculation function:
1. Reads all inputs and computes allocations
2. Updates every DOM element (amounts, subtotals, % pills, bar chart, warnings)
3. Calls `saveState()` to persist to URL hash

---

## State persistence (URL hash)

State is encoded as Base64 JSON in `location.hash`:

```
#eyJuIjoiVW50aXRsZWQg...
```

**`getState()`** serializes: project name (`n`), value (`v`), profit % (`p`), tax % (`t`), comp % (`c`), and the three row arrays (`f`, `o`, `e`).

**`loadState()`** reads the hash on page load and restores all values. If no hash or invalid hash, defaults are loaded.

Every input change triggers `saveState()` automatically. Copy the URL at any time to capture the full state.

---

## Grid systems

### Metric cards (`.g3`)
3-column equal grid for the bottom summary cards (Project value, Tax, Project net).

### Row grids
Two grid templates for line item rows:

| Class | Columns | Used by |
|-------|---------|---------|
| `.c-foh` | `28px 1fr 88px 58px 80px 80px` | Sections with toggles (Contractors, Expenses) |
| `.c-oh` | `1fr 88px 58px 80px 80px` | Back of House (no toggle) |

On mobile (<600px), these collapse:
- `.c-foh` → `28px 1fr auto` (toggle + name + total)
- `.c-oh` → `1fr auto` (name + total)

---

## CSS custom properties

All colors are defined as CSS variables on `:root` with a dark mode override:

| Variable | Purpose |
|----------|---------|
| `--color-background-primary` | Input/card inner backgrounds |
| `--color-background-secondary` | Card backgrounds |
| `--color-background-tertiary` | Page background |
| `--color-text-primary` | Main text |
| `--color-text-secondary` | Secondary text, labels |
| `--color-text-tertiary` | Hints, column headers |
| `--color-text-danger` | Warnings, negative values |
| `--color-text-success` | Positive values, save confirmation |
| `--color-border-*` | Three border opacity levels + danger |
| `--border-radius-md` | 8px — inputs, buttons |
| `--border-radius-lg` | 12px — cards |

---

## Mobile responsive design

**Breakpoint:** 600px

On mobile, line item rows collapse to show only the item name and total. Tapping a row expands it to reveal Rate, Qty, and Unit fields stacked vertically with labels, plus a full-width "Remove" button.

Key mobile behaviors:
- `.row-detail` — wraps Rate/Qty/Unit; `display:contents` on desktop (invisible), `display:block` on mobile when `.expanded`
- `.mob-field` / `.mob-field-label` — labeled field wrappers; `display:contents` on desktop, `display:block` on mobile
- `.mob-del` — mobile delete button; hidden on desktop, shown in expanded detail
- `.add-btn-mob` — replaces ghost row hover interaction with a visible button
- `.ghost-row` — hidden entirely on mobile
- Column headers with `.mob-hide` are hidden on mobile

A delegated click handler toggles `.expanded` on row taps (ignoring taps on inputs, toggles, selects, and buttons).

---

## Drag and drop

`setupDrag(container, array, renderFn)` enables reordering via HTML5 Drag API.

- Detects drop position (top/bottom half of target row) for accurate indicator placement
- Reorders the source array in place, then re-renders
- Visual feedback: `.dragging` (opacity), `.drag-over-top` / `.drag-over-bottom` (border indicators)

---

## Adding a new section

Follow this pattern:

1. **Data array** — `let myRows = [];`
2. **Render function** — `renderMy()` mapping array to `.cols` grid rows with `.row-detail` wrapper for Rate/Qty/Unit
3. **Add function** — `addMyRow()` pushes default row object, calls render + recalc
4. **Calc function** — `calcMy()` reduces array to total (respecting `on` toggle)
5. **Update `recalc()`** — add the new section's cost to the allocation math, update its subtotal and % pill
6. **Update `getState()` / `loadState()`** — add array to serialization under a new key
7. **HTML card** — add card with: `.card-title` + `.pct-tag`, column headers, `id="my-rows"` container, ghost row + `.add-btn-mob`, `.subtot-row`
8. **Wrap in `.card-body`** — for empty-state hide/reveal behavior
9. **Init** — call `renderMy()` and `setupDrag(...)` at the bottom of the script

---

## Local development

No build tools needed. Just serve the file:

```bash
npx serve .
# or
python3 -m http.server 8080
```

Open `http://localhost:8080` (or whatever port) in your browser.

---

## Deployment

Hosted on GitHub Pages from the `main` branch. Push to `main` and the site updates automatically within ~1 minute.
