# ClientFlow — Clients Page Design Documentation

## Overview

The Clients page is a dense, single-view dashboard for managing creative production relationships. It combines client selection, action management, financials, lead discovery, content scheduling, project workflow, and team communication into one persistent surface — designed for agency producers and project managers who need everything in peripheral vision without switching contexts.

---

## Design Language

### Philosophy

The UI follows a **dark, high-contrast editorial aesthetic** rooted in tool-first thinking. Decoration is absent. Every element earns its place by surfacing information or enabling action. The result reads closer to a developer tool or financial terminal than a typical SaaS dashboard.

Tension between density and clarity is resolved through strict typographic hierarchy, restrained color use, and consistent spatial rhythm — not whitespace.

### Typography

Two typefaces carry the entire interface:

**Hanken Grotesk** — display and body. Used for all human-readable content: names, titles, messages, labels, and CTAs. Weights used are 400, 500, 600, and 700. The typeface's slightly geometric construction gives the UI a structured, confident feel without feeling cold.

**JetBrains Mono** — metadata and system labels. Used for IDs, timestamps, status tags, unit labels, uppercase microcopy, and anything that benefits from fixed-width rendering. Its presence is a consistent signal: "this is data, not prose."

Type scale is kept deliberately tight:
- Section titles: 16px / 600
- Client names: 17px / 700
- Body / card content: 12–13px / 400–600
- Metadata / mono labels: 7–9px / 400–500, letter-spacing 0.1–0.15em, all-caps

### Color

The palette is almost entirely neutral with a single primary accent.

| Token | Value | Role |
|---|---|---|
| `--surface` | `#141313` | Page background |
| `--surface-low` | `#1c1b1b` | Cards, panels |
| `--surface-container` | `#201f1f` | Inset cells |
| `--surface-high` | `#2a2a2a` | Hover states |
| `--surface-highest` | `#353434` | Placeholder blocks |
| `--on-surface` | `#e5e2e1` | Primary text |
| `--on-surface-variant` | `#c4c7c8` | Secondary text |
| `--outline` | `#8e9192` | Muted labels, icons |
| `--outline-variant` | `#444748` | Borders, dividers |
| `--primary` | `#ffffff` | Accent — CTAs, active states, done indicators |
| `--on-primary` | `#2f3131` | Text on white buttons |

White (`--primary`) is the sole accent color. It marks selected tabs, active borders, the "done" status dot, event bars on the calendar, and all primary action buttons. This restraint makes its presence immediately meaningful — anything white is either selected, complete, or the most important action available.

### Borders and Surfaces

All cards and panels use `1px solid var(--outline-variant)` — a near-invisible boundary that defines structure without adding visual weight. Priority actions use a `3px solid var(--primary)` left border as the only decorative stroke in the layout.

Border-radius follows a four-step scale: `4px` (sm), `8px`, `12px` (md), `16px` (lg). Cards and major panels use `lg`; inline chips and status tags use `sm` or `full`.

---

## Layout

### Page Structure

The page is a three-tier vertical stack:

1. **Client selector strip** — full-width tab bar, pinned to top
2. **Body grid** — two-column layout, fills remaining height
3. *(no footer)*

The body grid splits into a fixed `300px` left column and a fluid right column. Both are set to `overflow: hidden` with internal scroll where needed, keeping the overall page chrome stable.

### Client Selector Strip

Four equal-width tabs in a CSS grid. Each tab shows a mono metadata line (either "Selected" or a numeric ID) above the client name. The selected tab gains a `2px bottom border` in white and a slightly lighter background. Unselected tabs show muted name color. The strip is purely navigational — selecting a client would populate the body with that client's data.

### Left Column

A flex column with `overflow-y: auto`. Contains four stacked sections with consistent `14px` horizontal margin:

- Actions Required
- Financial Status
- Lead Found
- New Project button

The column is fixed-width and scrolls independently of the right side.

### Right Column

A flex column split into two rows:

- **Content Schedule** — fixed height, `flex-shrink: 0`, pinned to top
- **Right Bottom** — a two-column sub-grid: Project Workflow (fluid) and Follow-ups (fixed `280px`)

---

## Components

### Actions Required

Three action cards with a header showing a count badge. The first card (highest priority) uses a `3px white left border` to visually separate it from standard cards. All cards share the same internal structure: an icon, a title, and a subtitle. Icons are hand-drawn SVG at 15px.

The count badge uses mono type, all-caps, with a faint border — a recurring pattern for numerical metadata throughout the page.

### Financial Status

A contained card showing invoice amount, number, and payment due date. The amount is rendered at 26px / weight 400 — the largest number on the page, reflecting its importance without resorting to bold. A horizontal rule separates the amount from due-date context. The Pay Now button is the only white-fill button in this section.

### Lead Found

The most complex left-column component. A card divided into four horizontal bands:

**Header band** — avatar, Instagram handle at 17px / 700, platform tag in mono, and an external link icon.

**Stats grid** — a 2×2 grid of metric cells, each separated by `1px` borders forming an internal table. Labels are 6px mono all-caps; values are 13px / 700. A supplementary sub-label (e.g. "+2.1k this week") sits below each value in 7px mono.

**Tags row** — pill-shaped chips with mono type. Used for niche categorization.

**Action row** — two equal-width buttons: a primary (white fill) "Reach Out" and a ghost "Save Lead".

### New Project Button

A full-width `white` button at the bottom of the left column. 14px / 700, 13px padding, `border-radius: lg`. Functions as a persistent CTA that persists regardless of scroll position (bottom of the scroll container).

### Content Schedule

A 7-column week grid. Each column has a mono day label above a cell. Cells have a minimum height of 120px and a subtle hover state.

The active day ("today") uses `--on-surface-variant` as its border color and highlights the date number in white. An event block uses a `2px white left border`, a short white bar, and mono all-caps text for the event name. Empty Friday cells contain a placeholder block to indicate a content slot exists but is unfilled.

Navigation chevrons sit in a paired button group with a shared border.

### Project Workflow

A free-form node canvas centered inside a dot-grid background. The grid uses `radial-gradient` dots at `22px` intervals in `--outline-variant` — present enough to give spatial context for node placement, subtle enough not to compete with node content.

Nodes are absolutely positioned inside a `560×260` inner wrapper that is flexbox-centered within the canvas container. This decouples the coordinate system from the container's actual dimensions.

**Node anatomy:**
- Header band: mono phase label (left) + status dot (right)
- Body: title, subtitle, and either action buttons or a done badge

**Three node states:**
- `done` — white border, white status dot, "Approved" badge with check icon
- `active` — `--on-surface-variant` border, filled status dot
- `pending` — `--outline-variant` border, hollow status dot

**Connector dots** sit at the left and right midpoints of each node, rendered as small circles with a surface-colored fill and a border — mimicking port handles from node editors.

**SVG connectors** are cubic bezier curves between node right-edges and left-edges. Stroke color reflects source node state: done connections use `--outline`, pending use `--outline-variant` with a `4,4` dash array.

**Drag behavior** — nodes are draggable via mouse. `mousedown` on a node (not on its buttons) begins a drag, capturing the cursor offset. Global `mousemove` updates position, clamped to the inner canvas bounds. `mouseup` ends the drag. The dragging node receives a `.dragging` class that applies `cursor: grabbing` and a lifted box-shadow.

### Follow-up Tracker

A scrollable list of communication entries, fixed at `280px` wide. Each item shows:

- A row with avatar initials circle, name, role in mono all-caps, and a timestamp
- A message body in 12px body text
- An optional status tag (urgent or done) in a pill with conditional border emphasis

Urgent tags use `--on-surface-variant` for both border and text, making them legible but not alarming. Done tags are at 50% opacity — visually receding.

An "+ Add Follow-up" button uses a dashed border, mono type, and all-caps — a ghost affordance that doesn't compete with content.

---

## Interaction Patterns

### Hover States

Every interactive element has a hover transition (120–150ms). Cards and list items lighten their background. Buttons shift color or transform slightly (`translateY(-1px)` on primary buttons). The approach is always additive — hover reveals the surface layer above the current one.

### Focus and Selection

The client tab selection model is tab-like: one selected, rest inactive. The selected state communicates through three simultaneous signals: lighter background, white bottom border, and white text color. This redundancy ensures the selection reads clearly even in peripheral vision.

### Drag and Drop (Workflow Nodes)

Node dragging is intentionally raw — no snap-to-grid, no magnetic connections. Positions are clamped within the canvas bounds. Buttons inside nodes are excluded from drag initiation via `e.target.closest('button')` guard, preserving click actions on Approve/Revise while allowing drag from any other part of the node surface.

---

## Spacing System

Spacing is not tokenized but follows consistent values in practice:

| Context | Value |
|---|---|
| Section margin (left col) | `14px` horizontal |
| Section gap (left col) | `12px` vertical |
| Card internal padding | `12–14px` |
| Body grid column gap | implicit via border |
| Week grid gap | `6px` |
| Node canvas inner padding | visual padding via centering |

---

## Scrollbar Styling

Custom scrollbars are applied globally: `4px` wide, transparent track, `--outline-variant` thumb with `2px` border-radius. This keeps scrollbars visible enough to be functional while near-invisible at rest.
