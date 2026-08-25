# Barbarians — Components
*Implementation specs for all UI components. Build reference — what and how, not why. Last verified against: v0.2.3.0.*

*For design rationale see `barbarians-ui-decisions-20260730.md`. For token values see `barbarians-tokens.md`.*

---

## Implementation Notes (v0.2.3.0)

- **Two-tab game screen.** Game screen is `#gameScreen` (flex column, `overflow: hidden`). Always-visible `#gameHeader` sits above `#tabContent`. `#tabContent` fills remaining space. `#tabBar` pins to bottom.
- **Journal is a tab pane, not a screen.** No dark journal screen. `#tabJournal` is a parchment-background scrollable pane alongside `#tabMen`. The old full-screen Journal Screen is retired.
- **Resource ledger replaces the resource strip.** `#resourceLedger` is a horizontally-scrollable parchment card with category columns, dot leaders, and a collapsible Quarry drawer. No separate resource strip or icon row exists.
- **Site card has no art zone.** Site card (`#siteCard`) shows stage name text + fortification stage dots + projects area. Site art is not yet implemented.
- **Structures card is a separate sibling card.** `#structuresCard` sits beside `#siteCard` in `#siteCardRow` (flex row). Hidden until first structure is built.
- **Action tray uses a pipeline pattern.** Actions are grouped by industry into horizontal pipeline rows with arrow connectors between nodes. No icon-grid layout.
- **Die-face skill chips on game cards.** Collapsed card shows the row only. Expanded card reveals `.char-die-row` — a strip of `.die-chip` elements (one per action), each containing a `.die-face` with authentic pip layout plus a `.die-label`.
- **Portrait modal on character face tap.** Tapping the silhouette area of a game card opens `.portrait-modal-backdrop` — a playing-card-shaped modal (75vw/vh, 2:3 ratio) with a placeholder SVG face zone (top 55%) and backstory text (bottom 45%).
- **Actions persist across days.** No `clearAllTasks()`. Characters carry their last assignment until the player changes it.
- **Start Day button in normal flow.** `#startDayBtn` sits at the bottom of `#tabMen` pane, above `#tabBar`. Not fixed-position.
- **Status bar no longer shows threat word or day counter.** Current status bar: "Barbarians" title (`.sb-left`) left, version tag right. Day counter and threat word are surfaced in the site card / day label instead.

---

## Screens

Three full-screen states. Only one visible at a time.

| Screen | ID |
|---|---|
| Selection Screen | `#selectionScreen` |
| Game Screen | `#gameScreen` |

*(Journal Screen retired — journal is now `#tabJournal` inside the game screen.)*

---

## 1. Character Card — Selection Screen

**Layout:** Vertical. Full-figure portrait top, card body below. Two-column grid, scrollable.

| Zone | Element | Spec |
|---|---|---|
| Portrait Zone | `.cand-portrait` | `170px` height. `object-fit: cover`. Fade gradient bottom 80px → parchment (`--portrait-fade`). |
| Name Row | `.cand-name` | Spectral SC `--text-md` 700. |
| Role Line | `.cand-role` | Spectral SC `--text-xs` 400 uppercase `--tracking-wider`. `--color-ink-role`. |
| Divider | `hr` | 1px `--divider-ink`. |
| Flavor Block | `.cand-flavor` | Spectral italic `--text-base`. Scrollable. |
| Supplies Line | `.cand-supplies` | Spectral SC `--text-xs` uppercase. `--color-supplies-parchment`. Contents only (e.g. "5 rations · salt"). |
| Die Grid | `.die-grid` | 3×2 grid of `.skill-chip` tiles. See §7. |

**States:**
- Default: `--card-border-well` border, `--color-card-bg` background
- Hover: `--color-parchment-deep` border
- Selected: `--card-border-selected` border + `--card-shadow-selected` box-shadow

---

## 2. Character Card — Game Screen

**Layout:** Horizontal. Single column inside `#rosterPanel`, scrollable. Collapsed by default.

### Collapsed Row (`.char-card-row`)

Height: `52px` (flex row, `align-items: stretch`).

| Zone | Element | Spec |
|---|---|---|
| Face Zone | `.char-face` | `52px` square. `--color-card-portrait-bg` background. SVG silhouette placeholder (140×140px, 28% opacity). Tappable — opens Portrait Modal. |
| Identity Block | `.char-identity` | Flex column, flex: 1. |
| — Name | `.char-name` | Spectral SC `--text-md` 700. `--color-ink-primary`. `--tracking-wide`. |
| — Role | `.char-role` | Spectral SC `--text-xs` uppercase. `--color-ink-role`. `--tracking-wider`. |
| — Level | `.char-level` | Spectral SC `--text-xs`. `--color-ink-ornament`. Roman numeral (I–VI). |
| Right Column | `.char-right` | Fixed `--card-right-width` (64px). Contains action slot. |

Tapping the card row (not the action slot, not the face) toggles expansion.

### Expanded (`.char-card.expanded`)

`.char-die-row` appears below the collapsed row, separated by `1px solid --color-parchment-deep`. Background `rgba(0,0,0,0.018)`. Contains `.die-chip` elements. See §6.

---

## 3. Character States

Applied to the entire `.char-card` element via CSS class and CSS filter.

| State | Class | Filter |
|---|---|---|
| Well | — | none |
| Hungry | `.hungry` | `saturate(0.55) brightness(0.95)` |
| Starving | `.starving` | `saturate(0.2) brightness(0.88)` |
| Dead | `.dead` | `grayscale(1) brightness(0.85)` + `opacity: 0.5` |

No per-state background or border token — filter handles the visual shift. Dead card: right column empty, identity block runs full width.

---

## 4. Action Slot

Fixed `--card-right-width` (64px) right column on every live game screen card. Vertically centered.

| State | Spec |
|---|---|
| Empty | Background `--color-slot-empty-bg`. `box-shadow: --action-slot-empty-shadow`. No border. Tappable — opens Action Tray. |
| Assigned | Background `--color-slot-assigned-bg`. No border. `--action-slot-size` (52px) square, `--action-slot-radius` (6px). Icon/label fills slot. |

---

## 5. Action Tray

Bottom sheet. Slides up from bottom of screen on action slot tap. Fixed position, `max-height: 70vh`, `overflow-y: auto`.

| Element | Spec |
|---|---|
| Surface | `--color-parchment` |
| Border radius | `14px 14px 0 0` |
| Handle zone | `.tray-handle-zone` — sticky top. Handle pill: 36×3px, `--color-parchment-deep`, centered. Ink rule below. |
| Header | `.tray-header` — "Assign [Name]". Spectral SC `--text-xs` `--color-ink-muted` uppercase `--tracking-widest`. Sticky below handle. |
| Industry section | `.tray-industry` — one per action group (Provisions, Fortification & Timber, Watch & Recon). Divided by ink rule. Label: `.tray-industry-label` — 9px, `--tracking-wider`, uppercase, `--color-ink-ornament`. |
| Pipeline row | `.tray-pipeline` — horizontal flex, `overflow-x: auto`, no scrollbar. Nodes connected by `.tray-pipe-arrow` (24px wide, font-size 22px ornament arrow). |
| Action node | `.tray-node` — flex column, centered. Tap to assign. |
| Node icon | `.tray-node-icon` — 84×62px, `border-radius: 8px`. Default: `rgba(138,106,58,0.08)` bg, `rgba(122,92,46,0.25)` border. Building actions: `--color-accent-verdigris-bg` bg, `--color-accent-verdigris` border. |
| Node label | `.tray-node-icon-text` — Spectral SC 8.5px 700, `--color-ink-role`, uppercase, centered, max 2 lines. |
| Selected node | `.tray-node-selected` — node icon gets `--color-accent-slate` border + `rgba(79,100,112,0.15)` bg. Label: `--color-accent-slate`. |
| Dimmed node | `.tray-node-dimmed` — 35% opacity, pointer-events none. Inputs unavailable. |

**Behavior:**
- Tap outside → closes, no change
- Tap different character's slot → stays open, context switches
- Tap same open slot → closes
- Auto-closes after each assignment

---

## 6. Die Chip (Game Card Expanded State)

`.char-die-row` — flex row, `flex-wrap: wrap`, `gap: 6px`, `padding: 8px 10px 10px`. One `.die-chip` per action the character has a skill for.

### Die Chip (`.die-chip`)
Flex column, `align-items: center`, `gap: 5px`.

### Die Face (`.die-face`)
36×36px. `border-radius: 8px`. `--color-card-bg` background. `border: 1px solid --color-parchment-deep`. Inset shadow. Nine-position pip grid (l/c/r × t/m/b), each position absolutely placed at 28%/50%/72%.

### Die Pip (`.die-pip`)
6×6px circle. `--color-accent-slate` fill. Inset shadow. Rendered at authentic d6 face positions (1–6 pips).

### Die Label (`.die-label`)
Spectral SC 6.5px. `--color-accent-slate`. Uppercase. Action name below the face.

---

## 7. Skill Chip (Selection Screen Die Grid)

Used in `.die-grid` (3×2 grid, `gap: 8px`) on the selection card. One per action in the character's skill profile.

### Skill Chip (`.skill-chip`)
`border-radius: 6px`. `--color-slot-assigned-bg` background. Flex column, centered. `padding: 4px 3px 3px`.

### Chip Name (`.skill-chip-name`)
Spectral SC 7.5px. `--color-accent-slate`. Uppercase. Centered.

### Pip Row (`.skill-pips`)
Flex row, `gap: 2.5px`. One `.skill-pip` per tier level.

### Pip (`.skill-pip`)
4×4px circle. Off: `rgba(79,100,112,0.22)`. On (`.skill-pip.on`): `--color-accent-slate`.

---

## 8. Portrait Modal

Triggered by tapping `.char-face` on a game card. Fixed overlay, `z-index: 200`.

| Element | Spec |
|---|---|
| Backdrop | `.portrait-modal-backdrop` — `rgba(30,18,6,0.55)` scrim. Flex centered. Display none → flex on `.open`. |
| Card | `.portrait-modal-card` — `width: min(75vw, 75vh × 0.667)`, `height: min(75vh, 75vw × 1.5)`. `border-radius: 14px`. `border: 1px solid --color-parchment-deep`. |
| Face zone | `.portrait-modal-face` — top 55% of card. `--color-card-portrait-bg`. SVG placeholder: 140×140px, 28% opacity. |
| Text zone | `.portrait-modal-text` — bottom 45%. `--color-card-bg`. `padding: 16px 18px`. |
| Name | `.portrait-modal-name` — Spectral SC 18px 700. `--color-ink-primary`. `--tracking-wide`. |
| Role | `.portrait-modal-role` — Spectral SC 11px. `--color-ink-role`. Uppercase. `--tracking-wider`. |
| Backstory | `.portrait-modal-backstory` — Spectral italic 14px. `--color-ink-secondary`. `line-height: 1.65`. Scrollable. |

Dismiss: tap backdrop (no close button).

---

## 9. Status Bar — Game Screen

`#statusBar` — pinned inside `#gameHeader`. Single row, space-between.

| Element | Spec |
|---|---|
| `.sb-left` | Flex row, `align-items: baseline`, `gap: 6px` |
| `.sb-title` | Spectral SC `--text-sm` 700. `--color-ink-primary`. Uppercase. `--tracking-wide`. "Barbarians" |
| `.sb-version` | Spectral SC 7px. `--color-ink-primary`. Uppercase. `--tracking-wide`. Opacity 0.5. Developer reference. |

*(Threat word and day counter not currently in status bar — day label appears in `#journalDayLabel` inside the Journal tab.)*

---

## 10. Site Card — Game Screen

`#siteCard` — flex child inside `#siteCardRow`. Parchment card, same border/radius as character cards.

**No art zone in current build.** Site art not yet implemented.

| Zone | Element | Spec |
|---|---|
| Stage name | `#siteState` | Spectral SC `--text-lg` uppercase. `--color-ink-primary`. `--tracking-wide`. Centered. |
| Stage dots | `#siteStage` | Flex row, `gap: 14px`, centered. Six `.stage-dot` elements. |
| Fosse enhancement | `#fosseEnhancement` | Hidden by default. Contextual. |
| Projects area | `#siteProjects` | Flex column, `gap: 4px`. `.site-section-label` (9px ornament) + `.site-projects-row` (flex wrap chips). |

### Stage Dots (`.stage-dot`)
8×8px circle. Default: `--color-parchment-deep` fill, `--color-ink-ornament` border. In-progress (`.in-progress`): `--color-accent-slate` border. Complete (`.on`): `--color-accent-slate` fill and border.

### Building Chip (`.building-chip`)
Spectral SC `--text-xs` uppercase. `--color-ink-role`. Transparent background, `1px solid --color-ink-ornament` border, `border-radius: 4px`. Hover tooltip via `::after` pseudo-element.

---

## 11. Structures Card — Game Screen

`#structuresCard` — sibling to `#siteCard` inside `#siteCardRow`. Hidden (`.visible` toggles `display: flex`) until first structure is built.

| Element | Spec |
|---|---|
| Label | `.structures-card-label` — Spectral SC 9px uppercase `--tracking-wide`. `--color-ink-ornament`. |
| List | `#structuresList` — flex column, `gap: 5px`, centered. |

Structure entries (from JS): Spectral SC `--text-xs` uppercase `--color-accent-slate`.

---

## 12. Resource Ledger — Game Screen

`#resourceLedger` — full-width parchment card in `#gameHeader`. `margin: 0 14px`. Horizontally scrollable, no scrollbar shown.

### Ledger Inner (`.ledger-inner`)
`padding: 10px 14px 12px`. Flex row, `gap: 0`. `width: max-content; min-width: 100%`.

### Category Column (`.ldg-cat`)
`min-width: 110px`. `padding-right: 16px`. `border-right: 1px solid rgba(200,184,154,0.45)`. Last child: no border.

### Category Head (`.ldg-cat-head`)
Spectral SC 8px, `letter-spacing: 0.2em`, uppercase. `--color-accent-ochre`. Bottom border: `1px solid rgba(139,105,20,0.22)`. `margin-bottom: 5px`.

### Standard Row (`.ldg-row`)
Flex row, `align-items: baseline`, `padding: 2px 0`.

- **Name** (`.ldg-name`): Spectral italic 10px. `--color-ink-muted`. Fresh state (`.fresh`): `--color-ink-secondary`.
- **Dot leader** (`.ldg-dots`): flex: 1. 1px tall. Radial-gradient dot pattern, `6px` repeat. `margin: 0 5px 3px`.
- **Tail** (`.ldg-tail`): flex row, `gap: 4px`.
- **Number** (`.ldg-num`): Spectral italic 10px 400. `--color-ink-muted`. `lining-nums tabular-nums`. Fresh: `--color-ink-secondary`.
- **Delta** (`.ldg-delta`): Spectral italic 8px. `--color-ink-secondary`. `lining-nums tabular-nums`.

### Quarry Toggle Row (`.ldg-quarry-toggle`)
Same layout as `.ldg-row`. Cursor pointer.

- **Quarry name** (`.ldg-quarry-name`): Spectral italic 10px **700**. `--color-accent-ochre`.
- **Quarry num** (`.ldg-quarry-num`): Spectral italic 10px **700**. `--color-accent-ochre`.
- **Chevron** (`.ldg-chevron`): 7px. `--color-ink-ornament`. Rotates on open (`transition: transform 120ms ease`).

### Quarry Drawer (`.ldg-quarry-drawer`)
`max-height: 0`, `overflow: hidden`. Open state (`.open`): `max-height: 300px`. `transition: max-height 200ms ease`.

### Quarry Indent (`.ldg-quarry-indent`)
`padding: 2px 0 3px 9px`. `border-left: 1px solid rgba(200,184,154,0.6)`. `margin: 1px 0 2px 2px`.

### Animal Row (`.ldg-animal-row`)
Flex row, `gap: 3px`, `padding: 1px 0`, `white-space: nowrap`.

- **Animal name** (`.ldg-animal-name`): Spectral italic 9px. `--color-ink-muted`. Fresh: `--color-ink-secondary`.
- **Multiplier** (`.ldg-animal-mult`): Spectral italic 9px. `--color-ink-ornament`. Fresh: `--color-ink-secondary`.

---

## 13. Journal Tab — Game Screen

`#tabJournal` — parchment tab pane. `overflow-y: auto`. `padding: 0 14px 16px`.

| Element | Spec |
|---|---|
| Day label | `#journalDayLabel` — Spectral SC `--text-lg` uppercase `--tracking-wide`. `--color-ink-primary`. Centered. `padding: 14px 0 8px`. Sticky top, `--color-parchment` background, `z-index: 5`. |
| Day log | `#dayLog` — Spectral `--text-sm`. `--color-ink-primary`. `line-height: 1.6`. |
| Day block | `.log-day-block` — `margin-bottom: 12px`. |
| Day header | `.log-day-header` — Spectral SC `--text-xs` uppercase `--tracking-wide`. `--color-ink-muted`. `margin: 0 0 4px`. |
| Log entry | `.log-entry` — `margin: 2px 0`. |
| Spoil entry | `.log-spoil` — `--color-selected` italic. Spoilage events. |
| Fail entry | `.log-fail` — `--color-selected` opacity 0.85. Total action failure. |
| Hunger (hungry) | `.hunger-hungry` — `#8b6914` italic. |
| Hunger (starving) | `.hunger-starving` — `--color-threat` italic. |

---

## 14. Tab Bar — Game Screen

`#tabBar` — `flex-shrink: 0`. Flex row. `border-top: 1px solid --color-parchment-deep`. `--color-parchment` background. `padding-bottom: env(safe-area-inset-bottom, 12px)`.

### Tab Button (`.tab-btn`)
Flex: 1. Flex column, centered. `padding: 10px 4px 8px`. No background, no border.

| Element | Default | Active (`.active`) |
|---|---|---|
| `.tab-btn-label` | Spectral SC 9px `--tracking-widest` uppercase. `--color-ink-ornament`. | `--color-ink-primary` |
| `.tab-btn-pip` | 4×4px circle. Transparent. | `--color-selected` fill |

---

## 15. Start Day Button — Game Screen

`#startDayBtn` — in normal flow at bottom of `#tabMen`. `flex-shrink: 0`. Full width. `border-top` via `border-image: --divider-ink 1`.

| State | Color | Opacity | Display |
|---|---|---|---|
| Hidden | — | — | `display: none` |
| Visible / disabled | `--color-ink-muted` | 0.5 | `display: block` |
| Ready (`.ready`) | `--color-selected` | 1 | `display: block` |

Spectral SC `--text-sm` uppercase `--tracking-widest`. `--color-parchment` background. No explicit border-radius (uses border-image).

---

## Deprecated

- **Journal Screen (full-screen dark)** — retired. Journal is `#tabJournal` parchment pane.
- **Resource Strip** — retired. Replaced by `#resourceLedger`.
- **Site Modal / Resource Modal** — retired. No stat icons or art zone on site card currently.
- **Site Card art zone** (`#site-art`, 110px illustration) — not yet implemented.
- **Unassigned warning modal** — inline styled `#unassignedModal`. Not a component spec — to be tokenized when redesigned.
