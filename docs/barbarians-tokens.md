# Barbarians — Design Tokens
*CSS custom properties for index.html. Last verified against: v0.5.1.0.*
*Parchment tokens are the active system for selection and game screens. Legacy dark tokens are retained for the Journal Screen only (`--color-journal-*`). Dark theme tokens for the old game screen (backgrounds, borders, status colors) remain in `:root` but are no longer used by any active component — candidates for removal once the journal screen's own token set is confirmed stable.*

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

  /* Card surfaces — well state baseline */
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
  --color-selected-glow:    #8b3a2f33;

  /* Status */
  --color-threat:           #8b4a2f;
  --color-supplies:         #5a7a4a;

  /* Character decay arc — card backgrounds */
  --color-card-hungry-bg:   #ddd8cc;
  --color-card-starving-bg: #c4beb4;
  --color-card-dead-bg:     #c0bcb4;

  /* Character decay arc — card borders */
  --color-card-well-border:     #c8b89a;
  --color-card-hungry-border:   #9a9080;
  --color-card-starving-border: #706860;
  --color-card-dead-border:     #888880;

  /* Character decay arc — state word colors */
  --color-state-hungry:   #7a6848;
  --color-state-starving: #4a3e30;  /* also apply opacity: 0.75 */
  --color-state-dead:     #555555;

  /* Action slot */
  --color-slot-assigned-bg: rgba(138,106,58,0.13);
  --color-slot-empty-bg:    #e8dbc6;

  /* Character decay arc — card backgrounds */
  --color-card-hungry-bg:   #ddd8cc;
  --color-card-starving-bg: #c4beb4;
  --color-card-dead-bg:     #c0bcb4;

  /* Character decay arc — card borders */
  --color-card-hungry-border:   #9a9080;
  --color-card-starving-border: #706860;
  --color-card-dead-border:     #888880;

  /* Character decay arc — state word colors */
  --color-state-hungry:   #7a6848;
  --color-state-starving: #4a3e30;
  --color-state-dead:     #555555;

  /* Journal screen — warm dark */
  --color-journal-bg:       #110d08;
  --color-journal-header:   #c8902a;
  --color-journal-rule:     #7a4a18;
  --color-journal-body:     #d4b87a;
  --color-journal-names:    #e8c890;
  --color-journal-return:   #5a3e1a;

  /* Accent palette — reserved for portrait art states, not UI */
  --color-accent-rust:      #8b3a2f;
  --color-accent-moss:      #6b7a4f;
  --color-accent-slate:     #4f6470;
}
```

---

## CSS Filters — Character Decay Arc

Applied to the entire `.char-card` element. Do not apply to individual child elements.

```css
.char-card.hungry  { filter: saturate(0.55) brightness(0.95); }
.char-card.starving { filter: saturate(0.2) brightness(0.88); }
.char-card.dead    { filter: grayscale(1) brightness(0.85); opacity: 0.5; }
```

*SVG roughen filter for ash/char border effect: designed, parked. Reintroduce as polish pass once core loop is built.*

---

## Typography Tokens

```css
:root {
  --font-display:   'Spectral SC', Georgia, serif;
  --font-body:      'Spectral', Georgia, serif;
  /* Resource counts: lining-figure font TBD — Georgia rejected (oldstyle figures).
     Interim: system sans with font-variant-numeric: lining-nums tabular-nums */
  --font-numeric:   'Trebuchet MS', 'Arial', sans-serif;

  /* Sizes */
  --text-xs:        8px;    /* state word, resource label, role line */
  --text-sm:        9px;    /* start day button, misc meta */
  --text-base:      9.5px;  /* flavor text (game card) */
  --text-md:        11px;   /* character name (game card), resource count */
  --text-lg:        13px;   /* threat word */
  --text-xl:        16px;   /* journal entry */
  --text-2xl:       22px;   /* journal day header */

  /* Leading */
  --leading-tight:  1.2;
  --leading-base:   1.65;
  --leading-loose:  1.9;

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
  --card-border-well:     1px solid var(--color-card-well-border);
  --card-border-selected: 1px solid var(--color-selected);
  --card-shadow-selected: 0 0 0 1px var(--color-selected-glow);
  --card-padding:         12px;
  --card-height-collapsed: 46px;

  /* Portrait zones */
  --portrait-height-full:      170px;
  --portrait-height-selection: 120px;
  --portrait-face-size:        46px;   /* game screen collapsed card */
  --portrait-fade-height:      80px;

  /* Action slot / right column */
  --card-right-width:   48px;
  --action-slot-size:   32px;
  --action-slot-radius: 6px;
  --action-slot-empty-shadow: inset 1px 2px 4px rgba(50,20,5,0.09);

  /* Resource strip */
  --resource-icon-size: 28px;

  /* Start Day button */
  --btn-start-radius:  8px;
  --btn-start-border:  1.5px solid var(--color-selected);

  /* Layout gaps */
  --gap-xs:   4px;
  --gap-sm:   8px;
  --gap-md:   12px;
  --gap-lg:   16px;

  /* Site card */
  --site-art-height:     110px;
  --site-stat-icon-size: 28px;

  /* Modals */
  --modal-radius:  12px;
  --modal-width:   290px;
  --modal-scrim:   rgba(17,13,8,0.6);
  --modal-padding: 22px 20px 18px;
  --modal-icon-size: 36px;   /* resource modal only */
}
```

---

## Component Shorthands

```css
/* Ink rule divider — paste as background on a 1px-height div */
--divider-ink: linear-gradient(
  to right,
  transparent,
  rgba(200,184,154,0.7),
  transparent
);

/* Portrait fade — paste as background on fade overlay div (selection screen) */
--portrait-fade: linear-gradient(
  to bottom,
  rgba(238,229,210,0) 0%,
  rgba(238,229,210,0.6) 50%,
  #eee5d2 88%
);

/* Resource count text-shadow — makes number legible over page parchment (#eee5d2) */
--resource-count-shadow: 0 0 4px #eee5d2, 0 0 2px #eee5d2;

/* Resource count text-shadow — on card-bg surface (#f5ede0), e.g. site card stats row */
--resource-count-shadow-card: 0 0 4px #f5ede0, 0 0 2px #f5ede0;
```

---

## Transition Tokens

```css
:root {
  --transition-screen: 300ms ease-in-out;  /* Game → Journal fade */
  --transition-card:   150ms ease;          /* Card expand / selection state */
  --transition-tray:   200ms ease-out;      /* Action tray open */
}
```

---

## Deprecated

- `--color-hunger: #8a4a1a` — retired. Hunger state now expressed via card-level CSS decay filters, not inline text color.
- `--portrait-height-game: 150px` — retired. Game screen portrait is 46px square face crop, not a tall zone.
- `--card-shadow: 1px 2px 6px rgba(30,20,10,0.22)` — retired. Cards use border only; drop shadow removed for naturalism.
- `--card-border: 1px solid var(--color-parchment-deep)` — replaced by `--card-border-well` and per-state border tokens.
- `--card-shadow-selected: 0 0 0 2px rgba(139,58,47,0.2)` — retired (v0.5.1.0). Selection screen selected state is border-only; glow shadow removed.
