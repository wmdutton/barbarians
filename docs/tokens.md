# Barbarians — Design Tokens
*CSS custom properties for index.html. Last verified against: v0.5.0.0.*

---

## Usage

All tokens declared in `:root`. Reference as `var(--token-name)` throughout CSS.

Two token layers co-exist in `:root`:
- **Parchment system** — Selection Screen and all future parchment surfaces. The target visual language.
- **Legacy dark system** — Game Screen only. Will be migrated to parchment in a future pass.

---

## Parchment System Tokens

### Color

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
  --color-selected-glow:    rgba(139,58,47,0.2);

  /* Status */
  --color-threat:           #8b4a2f;
  --color-supplies-parchment: #5a7a4a;  /* supplies line on parchment cards */

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

### CSS Filters — Character Decay Arc

Applied to the entire `.char-card` element. Do not apply to individual child elements.

```css
.char-card.hungry   { filter: saturate(0.55) brightness(0.95); }
.char-card.starving { filter: saturate(0.2) brightness(0.88); }
.char-card.dead     { filter: grayscale(1) brightness(0.85); opacity: 0.5; }
```

*SVG roughen filter for ash/char border effect: designed, parked. Reintroduce as polish pass once core loop is built.*

### Typography

```css
:root {
  --font-display:  'Spectral SC', Georgia, serif;   /* headings, labels, buttons */
  --font-body-new: 'Spectral', Georgia, serif;       /* flavor text, italic passages */
  --font-numeric:  'Trebuchet MS', Arial, sans-serif; /* resource counts — lining figures */

  /* Sizes */
  --text-xs:   8px;    /* state word, resource label, role line, version tag */
  --text-sm:   9px;    /* buttons, misc meta */
  --text-base: 9.5px;  /* flavor text (game card) */
  --text-md:   11px;   /* character name (game card), resource count */
  --text-lg:   13px;   /* threat word, selection screen instructions */
  --text-xl:   16px;   /* journal entry */
  --text-2xl:  22px;   /* journal day header */

  /* Leading */
  --leading-tight: 1.2;
  --leading-base:  1.65;
  --leading-loose: 1.9;

  /* Tracking */
  --tracking-wide:   0.1em;
  --tracking-wider:  0.14em;
  --tracking-widest: 0.2em;
}
```

**Font load:** Google Fonts — add to `<head>` or `@import` at top of `<style>`:
```
https://fonts.googleapis.com/css2?family=Spectral+SC:wght@400;600;700&family=Spectral:ital,wght@0,400;0,700;1,400;1,700&display=swap
```

### Spacing & Layout

```css
:root {
  /* Card */
  --card-radius:          10px;
  --card-border-well:     1px solid var(--color-card-well-border);
  --card-border-selected: 1px solid var(--color-selected);
  --card-shadow-selected: 0 0 0 2px rgba(139,58,47,0.2);
  --card-padding:         12px;
  --card-height-collapsed: 46px;

  /* Portrait zones */
  --portrait-height-full:      170px;  /* selection screen full-figure */
  --portrait-height-selection: 120px;  /* reserved, not currently used */
  --portrait-face-size:        46px;   /* game screen collapsed card face crop */
  --portrait-fade-height:      80px;

  /* Action slot / right column */
  --card-right-width:   48px;
  --action-slot-size:   32px;
  --action-slot-radius: 6px;
  --action-slot-empty-shadow: inset 1px 2px 4px rgba(50,20,5,0.09);

  /* Resource strip */
  --resource-icon-size: 28px;

  /* Start Day button */
  --btn-start-radius: 8px;
  --btn-start-border: 1.5px solid var(--color-selected);

  /* Layout gaps */
  --gap-xs: 4px;
  --gap-sm: 8px;
  --gap-md: 12px;
  --gap-lg: 16px;

  /* Site card */
  --site-art-height:     110px;
  --site-stat-icon-size: 28px;

  /* Modals */
  --modal-radius:  12px;
  --modal-width:   290px;
  --modal-scrim:   rgba(17,13,8,0.6);
  --modal-padding: 22px 20px 18px;
  --modal-icon-size: 36px;
}
```

### Component Shorthands

```css
/* Ink rule divider — 1px-height element */
--divider-ink: linear-gradient(to right, transparent, rgba(200,184,154,0.7), transparent);

/* Portrait fade — overlay div at bottom of portrait zone (selection screen) */
--portrait-fade: linear-gradient(to bottom, rgba(238,229,210,0) 0%, rgba(238,229,210,0.6) 50%, #eee5d2 88%);

/* Resource count text-shadow — over page parchment (#eee5d2) */
--resource-count-shadow: 0 0 4px #eee5d2, 0 0 2px #eee5d2;

/* Resource count text-shadow — over card-bg surface (#f5ede0) */
--resource-count-shadow-card: 0 0 4px #f5ede0, 0 0 2px #f5ede0;
```

### Transitions

```css
:root {
  --transition-screen: 300ms ease-in-out;  /* Game → Journal fade */
  --transition-card:   150ms ease;          /* Card expand / selection state */
  --transition-tray:   200ms ease-out;      /* Action tray open */
}
```

---

## Legacy Dark System Tokens (Game Screen only)

These cover the game screen, which has not yet been migrated to the parchment system. Do not use for new surfaces — use parchment tokens instead.

```css
:root {
  --color-bg-page:   #1b1b1b;
  --color-bg-panel:  #262626;
  --color-bg-log-entry: #202020;
  --color-bg-candidate-selected: #33301f;

  --color-text-primary:  #e8e0d0;
  --color-text-muted:    #bdbdbd;
  --color-text-disabled: #999;
  --color-text-footer:   #666;

  --color-accent-gold:       #8a6d3b;
  --color-accent-gold-light: #c9a86a;
  --color-accent-gold-muted: #8a7a54;

  --color-border-default: #555;
  --color-border-subtle:  #333;

  --color-status-hungry-bg:    #6b5a1f;
  --color-status-hungry-text:  #e6d6a6;
  --color-status-starving-bg:  #6b2f1f;
  --color-status-starving-text: #e6b6a6;
  --color-status-deceased:     #6b1f1f;
  --color-status-deceased-text: #a05050;

  --color-victory-bg:     #2f4a2f;
  --color-victory-text:   #b6e6b6;
  --color-victory-border: #5a8a5a;
  --color-defeat-bg:      #4a2f2f;
  --color-defeat-text:    #e6b6b6;
  --color-defeat-border:  #8a5a5a;

  --color-supplies-text: #7a9e7a;

  --font-body: Georgia, serif;  /* legacy game screen only */
}
```

---

## Deprecated

- `--color-hunger: #8a4a1a` — retired. Hunger state uses card-level CSS decay filters.
- `--portrait-height-game: 150px` — retired. Game screen portrait is 46px square face crop.
- `--card-shadow: 1px 2px 6px rgba(30,20,10,0.22)` — retired. Cards use border only.
- `--card-border: 1px solid var(--color-parchment-deep)` — replaced by `--card-border-well` and per-state border tokens.
- `--color-supplies: #5a7a4a` — replaced by `--color-supplies-parchment` (same value, renamed for token-system clarity).
- `--card-shadow-selected: 0 0 0 1px var(--color-selected-glow)` — updated to `0 0 0 2px rgba(139,58,47,0.2)` (explicit value, wider spread).
