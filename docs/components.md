# Barbarians — Components
*Implementation specs for all UI components. Build reference — what and how, not why. Last verified against: v0.5.0.0.*

*For design rationale see `barbarians-ui-decisions-20260726.md`. For token values see `barbarians-tokens-20260726.md`.*

---

## Screen Layout

Three full-screen states. Only one is ever visible at a time.

| Screen | Stack order (top → bottom) |
|---|---|
| **Selection Screen** | Header block → Button row → Character cards (2-col grid, scrollable) |
| **Game Screen** | Status Bar → Resource Strip → Site Card → Character Cards (scrollable) → Start Day Button |
| **Journal Screen** | Day Header → Rule → Journal Entry → Return Touch |

Transition Game → Journal: fade to black (300ms), Journal fades in (300ms). No slide.

---

## 1. Character Card — Selection Screen

**Layout:** Vertical. Full-figure portrait top, card body below. Two-column grid, single-column on mobile.

### Portrait Zone

```
height: --portrait-height-full (170px)
background: --color-card-portrait-bg
object-fit: cover, object-position: center top
```

**Portrait placeholder** — shown when no portrait image is assigned. Inline SVG: generic standing figure silhouette, 72×72px, `opacity: 0.28`, centered in zone. Defined as `SILHOUETTE_SVG` constant in JS; rendered via `portraitZoneHTML(src)` helper. To add a real portrait: set `candidate.portraitSrc = "path/to/file.png"` — the helper renders an `<img>` automatically.

**Fade overlay** — absolutely positioned `div.candidate-portrait-fade` at bottom of portrait zone. Height `--portrait-fade-height` (80px). Background `--portrait-fade`. Bleeds portrait into card body.

### Card Body

| Zone | Element | Spec |
|---|---|---|
| Name | `p.cand-name` | Spectral SC 11px 700. `--color-ink-primary`. `--tracking-wide`. |
| Role | `p.cand-role` | Spectral SC 8px 400 uppercase. `--color-ink-role`. `--tracking-wider`. |
| Divider | `hr.cand-divider` | 1px, `--divider-ink`. Margin 4px top/bottom. |
| Flavor text | `p.cand-flavor` | Spectral italic 9.5px. `--color-ink-secondary`. Line-height `--leading-base`. |
| Tier hints | `p.cand-hints` | Spectral italic 9px. `--color-ink-muted`. One sentence each for Practiced and Poor hints. |
| Supplies | `p.cand-supplies` | Spectral SC 8px uppercase. `--color-supplies-parchment`. No label prefix — contents only ("4 rations"). |

### Card States

| State | Border | Background | Extra |
|---|---|---|---|
| Default | `--card-border-well` | `--color-card-bg` | — |
| Selected | `--card-border-selected` | `--color-card-bg` | `--card-shadow-selected` + filled dot 8px, `--color-selected`, top-right corner (absolute, 10px in) |

### Selection Screen Header

Sits above the button row and card grid. Max-width 680px, centered.

```
.sel-header padding: 28px 16px 16px
```

| Element | Spec |
|---|---|
| Title `p.sel-title` | Spectral SC 38px 700. `--color-ink-primary`. Letter-spacing 0.06em. |
| Rule `hr.sel-rule` | 1px, `--divider-ink`. Max-width 220px, centered. Margin-bottom 10px. |
| Instructions `#selectionInstructions` | Spectral italic 13px. `--color-ink-muted`. Updates live: "Choose 5 men — N of 5 selected." |

**Version tag** `p.sel-version` — `position: absolute`, top-right (top 14px, right 18px). 7px, `--color-ink-ornament`, opacity 0.6. Visible but not part of the composition.

### Button Row `.sel-actions`

Sits between header and card grid. Max-width 680px, centered, `justify-content: center`. Margin: 10px auto 24px.

| Button | Class | Label logic |
|---|---|---|
| Reroll | `.sel-btn.sel-btn-secondary` | "Reroll Characters" on load; "N Rerolls Remaining" / "1 Reroll Remaining" after first use. Disabled at 0 rerolls. |
| Confirm | `.sel-btn.sel-btn-primary` | "Confirm Vanguard". Disabled until exactly 5 selected. |

Button shared spec: Spectral SC 9px, `--tracking-widest`, uppercase, padding 11px 20px, border-radius 8px.
- Secondary: transparent bg, `1.5px solid --color-parchment-deep`, `--color-ink-muted` text.
- Primary: transparent bg, `1.5px solid --color-selected`, `--color-selected` text. Hover: `rgba(139,58,47,0.06)` bg.

---

## 2. Character Card — Game Screen

**Layout:** Horizontal. Single column, scrollable. Collapsed by default.

#### Collapsed

| Zone | Content | Spec |
|---|---|---|
| Portrait Zone | Face-crop portrait | 46px × 46px. `object-fit: cover`, centered on face. |
| Identity Block | Name / Role / State word | Name: Spectral SC 11px 700. Role: Spectral SC 8px uppercase `#7a5c2e`. State word: Spectral italic 8px, color per state table. |
| Right Column | Action slot | Fixed 48px wide. See Component 4. |

Card height: 46px collapsed.

#### Expanded

Tap card body (not action slot) to expand. Flavor text block appears below the collapsed row, separated by 1px gradient rule. Flavor text: Spectral italic 9.5px `#3a2f1e`, line-height 1.65, padding `5px 10px 9px 10px`. Portrait does not change size.

**States:**
- Default: `1px solid #c8b89a` border, `#f5ede0` background
- Selected: `1px solid #8b3a2f` border + `0 0 0 1px #8b3a2f33` shadow

---

## 3. Character States

Applied to the entire `.char-card` element via CSS class. No per-child overrides.

| State | Class | Background | Border | Filter | State word | State word color |
|---|---|---|---|---|---|---|
| Well | — | `#f5ede0` | `#c8b89a` | none | — | — |
| Hungry | `.hungry` | `#ddd8cc` | `#9a9080` | `saturate(0.55) brightness(0.95)` | *hungry* | `#7a6848` |
| Starving | `.starving` | `#c4beb4` | `#706860` | `saturate(0.2) brightness(0.88)` | *starving* | `#4a3e30` opacity 0.75 |
| Dead | `.dead` | `#c0bcb4` | `#888880` | `grayscale(1) brightness(0.85)` | *dead* | `#555` |

State word: Spectral italic 8px 400. Sits below Role line in Identity Block.

Dead: entire card `opacity: 0.5`. Right column empty — identity block runs full width. No action slot.

---

## 4. Action Slot & Action Tray

### Action Slot

Fixed 48px right column on every game screen card. Vertically centered.

| State | Spec |
|---|---|
| Empty | Background `#e8dbc6`. `box-shadow: inset 1px 2px 4px rgba(50,20,5,0.09)`. No border. Tappable — opens Action Tray. |
| Assigned | Background `rgba(138,106,58,0.13)`. No border. 32×32px inner slot, border-radius 6px. Icon fills slot. |

### Action Tray

Bottom sheet. Slides up from bottom of screen on action slot tap.

| Element | Spec |
|---|---|
| Surface | Parchment `#eee5d2` |
| Top edge | 1px gradient ink rule + handle pill: 36px wide, 3px tall, `#c8b89a`, centered, opacity 0.7 |
| Header | "Assign [Name]" — Spectral SC 8px `#9a8060` uppercase letter-spacing 0.2em |
| Action row | 32px icon box + action name. Row padding `7px 8px`. Border-radius 8px. |
| Icon box | 32×32px border-radius 6px. Background `rgba(138,106,58,0.10)`. Placeholder border during development. |
| Icon inner | 18×18px border-radius 3px. Background `rgba(122,92,46,0.45)`. |
| Action name | Spectral SC 9px uppercase letter-spacing 0.1em. Color `#2a1f10` default, `#4f6470` (slate) when selected. |
| Group divider | 1px gradient ink rule. No group labels. |
| Group order | Provisions → Fortification & Timber → Watch & Recon |

**Character-specific:** not all characters have access to all actions. Tray reflects available actions for the active character only.

**Dismiss and context-switching:**
- Tap outside → closes, no change
- Tap a different character's slot → stays open, context switches
- Tap same assigned slot that opened tray → closes, no change
- Auto-closes after each assignment

---

## 5. Status Bar — Game Screen

Pinned to top. Single row.

| Element | Position | Spec |
|---|---|---|
| Day number | Left | Spectral SC 10px 700 uppercase. e.g. "DAY 4" |
| Threat word | Center | Spectral italic 13px `#8b4a2f`. e.g. "uneasy" |
| Version | Right | Spectral SC 7px `#c8b89a` uppercase. Developer reference only. |

Separated from Resource Strip below by 1px gradient ink rule.

---

## 6. Resource Strip — Game Screen

Sits below Status Bar, above Site Card. Scrolls horizontally.

**Collapsed (default):** Icon + number overlaid center. Tap to expand.
**Expanded:** Labels appear below each icon (Spectral SC 6.5px uppercase).
**Number:** Lining-figure font. 11px 700. Centered over icon. `text-shadow: --resource-count-shadow`. `font-variant-numeric: lining-nums tabular-nums`.
**Icon size:** 28×28px. Journal-sketch aesthetic (ChatGPT-generated, pending).

---

## 7. Site Card — Game Screen

**Layout:** Vertical. Full width. Fixed height — not collapsible.

| Zone | Content | Spec |
|---|---|---|
| Art Zone | Stage progression illustration | `--site-art-height` (110px). `object-fit: cover`, `width: 100%`. Tappable — opens Site Modal. Placeholder: `#d8ccb4` fill, centered label. |
| Stage Name Line | Current fortification stage name | Spectral SC 9px 400 uppercase. `--color-ink-muted`. Padding `5px 12px`. Background `--color-card-bg`. |
| Divider | Ink rule | 1px, `--divider-ink`. |
| Stats Row | Site resource icons, horizontally scrollable | Padding `7px 10px 8px`. `overflow-x: auto`. Background `--color-card-bg`. Each stat tappable — opens Resource Modal. |

**Card border/radius:** same as character card — `--card-border-well`, `--card-radius`.

**Stats Row — individual stat states:**

| State | Display | Behavior |
|---|---|---|
| Hidden | Nothing rendered | Not tappable |
| Approximate | Icon only, `opacity: 0.45`, no number | Tappable → Resource Modal with vague flavor |
| Known | Icon + number centered over icon | Tappable → Resource Modal with concrete flavor |

---

## 8. Site Modal

Triggered by tapping the Site Card art zone. Covers game screen with dark scrim. Tap anywhere to dismiss.

| Zone | Spec |
|---|---|
| Scrim | `background: --modal-scrim`. Full screen. |
| Modal box | `--color-card-bg` bg. `--card-border-well`. `border-radius: --modal-radius`. `padding: --modal-padding`. Width `--modal-width`. |
| Title | Spectral SC 13px 600. `--color-ink-primary`. Uppercase. `--tracking-wider`. Centered. |
| Rule | 1px `--divider-ink`. |
| Body | Spectral italic 13px. `--color-ink-secondary`. `line-height: --leading-base`. Centered. |
| Dismiss | Spectral SC 9px. `--color-ink-muted`. Uppercase. `--tracking-widest`. Centered. |

---

## 9. Resource Modal

Triggered by tapping an approximate or known stat icon on the Site Card. Same scrim/dismiss behavior as Site Modal.

| Zone | Spec |
|---|---|
| Icon | `--modal-icon-size` (36px). Centered. No box. |
| Title | Same as Site Modal title spec. |
| Body | Spectral italic 13px. `--color-ink-secondary`. Centered. Vague or concrete per stat state. |
| Dismiss | Same as Site Modal. |

---

## 10. Start Day Button — Game Screen

Pinned to bottom. Full width. Ink rule above.

| State | Border | Text color | Tappable |
|---|---|---|---|
| Enabled | `1.5px solid #8b3a2f` | `#8b3a2f` | Yes |
| Disabled | `1.5px solid #c8b89a` | `#c8b89a` | No |

Background always `#eee5d2`. Spectral SC 9px uppercase `--tracking-widest`. Border-radius 8px. Label: "Begin the Day".

---

## 11. Journal Screen

Background `#110d08`.

| Zone | Spec |
|---|---|
| Day Header | Spectral SC 22px 600 `#c8902a`. Left-aligned. e.g. "Day Four" |
| Rule | 1px gradient `#7a4a18` → transparent. |
| Journal Entry | Spectral italic 16px `#d4b87a`. Character names in `#e8c890`. Single paragraph. |
| Return Touch | Spectral SC ~12px `#5a3e1a` centered. Whole screen tappable. Label "Dawn". |

---

## Asset Spec — Character Portraits

| Crop | Usage | Display size | Notes |
|---|---|---|---|
| Full figure | Selection screen | ~full card width × 170px | `object-fit: cover`, `object-position: center top` |
| Face crop | Collapsed game card | 46×46px | `object-fit: cover`, centered on face |

- Git paths only — never base64 inlined
- Trim ~14–15% transparent canvas margin with PIL bounding-box before committing (`extract_assets.py`)
- Set `candidate.portraitSrc = "path"` to attach a portrait; `portraitZoneHTML()` renders it automatically. No src → silhouette placeholder.

---

## Deprecated

*(none yet)*
