# Barbarians — Design Tokens
*CSS custom properties for index.html. Last verified against: v0.2.3.0.*
*Parchment tokens are the active system for all screens. No dark-theme journal screen exists — the journal is a parchment tab pane inside the game screen. Legacy `--color-journal-*` tokens have been removed.*

---

## Usage

All tokens declared in `:root`. Reference as `var(--token-name)` throughout CSS.

---

## Color Tokens

```css
:root {
  /* Parchment surfaces */
  --color-parchment:        #eee5d2;
  --color-parchment-mid:    #d8ccb4;
  --color-parchment-deep:   #c8b89a;

  /* Card surfaces */
  --color-card-bg:          #f5ede0;
  --color-card-portrait-bg: #d8ccb4;

  /* Ink — text on parchment */
  --color-ink-primary:      #2a1f10;
  --color-ink-secondary:    #3a2f1e;
  --color-ink-role:         #7a5c2e;
  --color-ink-ornament:     #b8a07a;
  --color-ink-muted:        #9a8060;

  /* Accent — selected / danger / primary action */
  --color-selected:         #8b3a2f;
  --color-selected-glow:    rgba(139,58,47,0.2);

  /* Status */
  --color-threat:           #8b3a2f;
  --color-supplies-parchment: #5a7a4a;
  --color-weapon-parchment:   #4f6470;

  /* Accent palette */
  --color-accent-slate:         #4f6470;   /* recon / water / skill chips / selected action */
  --color-accent-ochre:         #8b6914;   /* ledger category heads / quarry rows */
  --color-accent-ochre-bg:      rgba(139,105,20,0.12);
  --color-accent-verdigris:     #4a7a6a;   /* building actions in action tray */
  --color-accent-verdigris-bg:  rgba(74,122,106,0.15);

  /* Action slot */
  --color-slot-assigned-bg: rgba(138,106,58,0.13);
  --color-slot-empty-bg:    #e8dbc6;
}
```

**Note:** `--color-accent-rust` and `--color-accent-moss` from the style bible are reserved for portrait art states and are not in `:root` — they are not used by any current CSS component.

---

## CSS Filters — Character Decay Arc

Applied to the entire `.char-card` element. No per-child overrides.

```css
.char-card.hungry   { filter: saturate(0.55) brightness(0.95); }
.char-card.starving { filter: saturate(0.2)  brightness(0.88); }
.char-card.dead     { filter: grayscale(1) brightness(0.85); opacity: 0.5; }
```

Character decay arc is expressed through CSS filters only. No separate card background or border tokens per hunger state exist in the current build — the decay filter handles the visual shift.

---

## Typography Tokens

```css
:root {
  --font-display:   'Spectral SC', Georgia, serif;
  --font-body:      'Spectral', Georgia, serif;
  --font-numeric:   'Trebuchet MS', 'Arial', sans-serif;
  /* Resource counts and ledger numbers: font-variant-numeric: lining-nums tabular-nums */

  /* Sizes */
  --text-xs:   11px;   /* role line, state word, labels, ledger rows, chip labels */
  --text-sm:   12px;   /* start day button, day log, misc meta */
  --text-base: 13px;   /* flavor text (selection card) */
  --text-md:   14px;   /* character name (game card), resource count */
  --text-lg:   16px;   /* site state name, journal day label */

  /* Leading */
  --leading-tight:  1.2;
  --leading-base:   1.65;

  /* Tracking */
  --tracking-wide:   0.1em;
  --tracking-wider:  0.14em;
  --tracking-widest: 0.2em;
}
```

---

## Spacing & Layout Tokens

```css
:root {
  /* Card */
  --card-radius:          10px;
  --card-border-well:     1px solid #c8b89a;
  --card-border-selected: 1px solid #8b3a2f;
  --card-shadow-selected: 0 0 0 2px rgba(139,58,47,0.2);
  --card-padding:         12px;
  --card-height-collapsed: 64px;   /* game screen char card collapsed */

  /* Right column / action slot */
  --card-right-width:   64px;
  --action-slot-size:   52px;
  --action-slot-radius: 6px;
  --action-slot-empty-shadow: inset 1px 2px 4px rgba(50,20,5,0.09);

  /* Portrait */
  --portrait-height-full:  170px;   /* selection screen full-figure zone */
  --portrait-fade-height:  80px;

  /* Layout gaps */
  --gap-xs:   4px;
  --gap-sm:   8px;
  --gap-md:   12px;
  --gap-lg:   16px;

  /* Transitions */
  --transition-screen: 300ms ease-in-out;
  --transition-card:   150ms ease;
  --transition-tray:   200ms ease-out;
}
```

---

## Component Shorthands

```css
/* Ink rule divider */
--divider-ink: linear-gradient(
  to right,
  transparent,
  rgba(200,184,154,0.7),
  transparent
);

/* Portrait fade — selection screen bottom gradient */
--portrait-fade: linear-gradient(
  to bottom,
  rgba(238,229,210,0) 0%,
  rgba(238,229,210,0.6) 50%,
  #eee5d2 88%
);
```

---

## Deprecated

- `--color-journal-*` (all) — removed. Journal screen is now a parchment tab pane; no dark journal palette.
- `--color-hunger: #8a4a1a` — retired. Hunger expressed via card-level CSS decay filters.
- `--color-supplies: #5a7a4a` — renamed to `--color-supplies-parchment`.
- `--color-card-hungry-bg / -starving-bg / -dead-bg` — removed. Decay via filter only.
- `--color-card-hungry-border / -starving-border / -dead-border` — removed. Decay via filter only.
- `--color-state-hungry / -starving / -dead` — removed. Decay via filter only.
- `--portrait-height-selection`, `--portrait-face-size` — removed; not used.
- `--portrait-height-game: 150px` — removed.
- `--card-shadow: 1px 2px 6px rgba(30,20,10,0.22)` — removed. Cards use border only.
- `--site-art-height`, `--site-stat-icon-size` — removed. Site card no longer uses illustration art zone.
- `--modal-radius / -width / -scrim / -padding / -icon-size` — removed. Modals use inline styles.
- `--resource-icon-size` — removed. Resource strip replaced by resource ledger.
- `--resource-count-shadow / -shadow-card` — removed. Not used by ledger.
- `--btn-start-radius / -border` — removed. Start Day button uses border-image and hardcoded values.
- `--leading-loose`, `--text-xl`, `--text-2xl` — removed. Not used in current build.
