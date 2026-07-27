# Barbarians — Components
*Implementation specs for all UI components. Build reference — what and how, not why. Last verified against: v0.5.1.0.*

*For design rationale see `barbarians-ui-decisions-20260726.md`. For token values see `barbarians-tokens-20260726.md`.*

## Implementation Notes (v0.5.1.0)

- **Action Tray (§4) is built.** Bottom sheet slides up from `translateY(100%)`. Shared single instance — context-switches on slot tap without closing. Scrim replaced with document-level click listener so character slots remain tappable while tray is open. Tapping the same open slot closes.
- **Flavor text hints are embedded in the flavor paragraph**, not a separate element. `HINT_POOL` in `index.html` holds role/tier-keyed sentences written as character observation. Sentences are picked once at `createSettler` time (`practicedSentence`, `poorSentence` on the character object) and read at render time — never re-rolled on re-render.
- **Actions persist across days.** `clearAllTasks()` removed. Characters carry their last assignment until the player changes it via the tray.
- **Game screen layout:** Status bar + resource strip wrapped in `#gameHeader` with `position: sticky; top: 0`. Start Day button `position: fixed; bottom: 0`. `#gameScreen` has `padding-bottom: 56px` to clear the button. `body` background switches to parchment when game screen is active via `body:has()`.
- **Status bar:** "Barbarians" title + interpunct separator + day counter grouped in `.sb-left` flex cluster left. Threat word centered. Version tag right.
- **Selection screen:** Selected-state dot indicator removed. Box-shadow on selected card removed (border only).

---



Three full-screen states. Only one is ever visible at a time.

| Screen | Stack order (top → bottom) |
|---|---|
| **Selection Screen** | Character cards (2-col grid, scrollable) |
| **Game Screen** | Status Bar → Resource Strip → Site Card → Character Cards (scrollable) → Start Day Button |
| **Journal Screen** | Day Header → Rule → Journal Entry → Return Touch |

Transition Game → Journal: fade to black (300ms), Journal fades in (300ms). No slide.

---

## 1. Character Card — Selection Screen

**Layout:** Vertical. Full-figure portrait top, card body below. Two-column grid on screen, scrollable.

| Zone | Content | Spec |
|---|---|---|
| Portrait Zone | Full-figure portrait | 170px height. `object-fit: cover`. No visible placeholder. Fade gradient bottom 80px → parchment (`--portrait-fade`). |
| Name Row | Character name | Spectral SC 12px 700. |
| Role Line | Role label | Spectral SC 9px 400 uppercase. `#7a5c2e`. |
| Divider | Ornamental rule | 1px gradient (`--divider-ink`). |
| Flavor Block | Flavor text | Spectral italic 10.5px. Scrollable — no truncation. |
| Supplies Line | Resource contents | Spectral SC 9px uppercase. `#5a7a4a`. No label prefix — contents only ("5 rations · salt"). |

**States:**
- Default: `1px solid #c8b89a` border, `#f5ede0` background
- Selected: `1px solid #8b3a2f` border + `0 0 0 1px #8b3a2f33` shadow + small filled dot indicator top-right corner

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
- Tap a different character's slot → stays open, context switches (header + selected state update)
- Tap same assigned slot that opened tray → closes, no change
- Auto-closes after each assignment — action pins to character's slot immediately

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

**Number:** Lining-figure font (TBD — not Georgia). 11px 700. Centered over icon. `text-shadow: 0 0 4px #eee5d2, 0 0 2px #eee5d2`. `font-variant-numeric: lining-nums tabular-nums`.

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
| Hidden | Nothing rendered — no slot, no placeholder | Not tappable |
| Approximate | Icon only, `opacity: 0.45`, no number | Tappable → Resource Modal with vague flavor |
| Known | Icon + number centered over icon | Tappable → Resource Modal with concrete flavor |

- Icon sits directly on parchment. No containing box or square.
- Icon size: `--site-stat-icon-size` (28px).
- Number: `--font-numeric`, 11px 700, centered over icon. `text-shadow: --resource-count-shadow-card`.
- Label below icon: Spectral SC 7px uppercase `--color-ink-muted`. Shown when stat is approximate or known.
- Stats row scrolls horizontally as additional stats reveal. No scroll indicator shown.

---

## 8. Site Modal

Triggered by tapping the Site Card art zone. Covers game screen with dark scrim. Tapping anywhere dismisses.

| Zone | Content | Spec |
|---|---|---|
| Scrim | Dark overlay | `background: --modal-scrim`. Covers full screen. Entire surface tappable to dismiss. |
| Modal box | Parchment card, centered | `--color-card-bg` bg. `--card-border-well` border. `border-radius: --modal-radius`. `padding: --modal-padding`. Width `--modal-width`. |
| Title | Stage name | Spectral SC 13px 600. `--color-ink-primary`. Uppercase. Letter-spacing `--tracking-wider`. Centered. |
| Rule | Ink divider | 1px, `--divider-ink`. Full modal width. |
| Body | Stage flavor / progress description | Spectral italic 13px. `--color-ink-secondary`. `line-height: --leading-base`. Centered. |
| Dismiss | "Tap to return" | Spectral SC 9px. `--color-ink-muted`. Uppercase. Letter-spacing `--tracking-widest`. Centered. |

No close button. No icon.

---

## 9. Resource Modal

Triggered by tapping an approximate or known stat icon on the Site Card. Same scrim/dismiss behavior as Site Modal.

| Zone | Content | Spec |
|---|---|---|
| Scrim | Dark overlay | Same as Site Modal. |
| Modal box | Parchment card, centered | Same container spec as Site Modal. |
| Icon | Resource icon, large | `--modal-icon-size` (36px). Centered. No box. |
| Title | Resource name | Same as Site Modal title spec. |
| Rule | Ink divider | Same. |
| Body | Resource flavor text — vague or concrete per stat state | Spectral italic 13px. `--color-ink-secondary`. `line-height: --leading-base`. Centered. |
| Dismiss | "Tap to return" | Same. |

Flavor text varies by stat state: approximate → character-voiced vague read ("the slopes above camp hold good standing oak — how much, none can yet say"); known → concrete count read ("forty-seven trees still standing within felling distance of camp").

---

## 10. Start Day Button — Game Screen

Pinned to bottom. Full width. Ink rule above: `linear-gradient(to right, transparent, #c8b89a, transparent)`, 1px, margin-bottom 7px.

| State | Border | Text color | Tappable |
|---|---|---|---|
| Enabled (all assigned) | `1.5px solid #8b3a2f` | `#8b3a2f` | Yes |
| Disabled (any unassigned) | `1.5px solid #c8b89a` | `#c8b89a` | No |

Background always `#eee5d2`. Spectral SC 9px uppercase letter-spacing 0.2em. Border-radius 8px. Label: "Begin the Day".

---

## 11. Journal Screen

Background `#110d08`.

| Zone | Spec |
|---|---|
| Day Header | Spectral SC 22px 600 `#c8902a`. Left-aligned. e.g. "Day Four" |
| Rule | 1px gradient `#7a4a18` → transparent. Minimal gap below header. |
| Journal Entry | Spectral italic 16px `#d4b87a`. Character names in `#e8c890`. Single paragraph. |
| Return Touch | Spectral SC ~12px `#5a3e1a` centered. Label only — whole screen is tappable. "Dawn" |

---

## Asset Spec — Character Portraits

| Crop | Usage | Display size | Notes |
|---|---|---|---|
| Full figure | Selection screen, expanded game card | ~220px wide × 170px tall | `object-fit: cover`, centered |
| Face crop | Collapsed game card | 46×46px | `object-fit: cover`, centered on face |

- Git paths only — never base64 inlined
- Trim ~14–15% transparent canvas margin with PIL bounding-box before committing (`extract_assets.py`)

---

## Deprecated

*Components moved here when replaced or removed. Kept for reference during build.*

*(none yet)*
