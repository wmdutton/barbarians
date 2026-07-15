# Barbarians — Design Tokens

*Reference for every color and font value used in `index.html`. These are
real CSS custom properties (not just a naming convention) — defined once in
the `:root { ... }` block at the top of the `<style>` tag, referenced
everywhere else via `var(--token-name)`. See [components.md](./components.md)
for which components use which tokens.*

---

## What a token is (quick primer)

A **design token** is a named value (a color, a font, a spacing amount) that
stands in for a raw hardcoded value, so the whole codebase references the
*name* instead of repeating the literal.

In this file, tokens are implemented as **CSS custom properties**:

```css
:root {
  --color-accent-gold: #8a6d3b;
}

.character {
  border-left: 3px solid var(--color-accent-gold);
}
```

`:root` is a CSS selector matching the `<html>` element — defining a
variable there makes it available to every element on the page. `--name` is
the custom property declaration; `var(--name)` is how any CSS rule reads it.
This is a native CSS feature, not a preprocessor trick — it works in every
modern browser with no build step, which matters for this project since
`index.html` is deliberately dependency-free.

**Why this matters as the project grows:** without tokens, "make the accent
gold darker" means finding and editing every hardcoded `#8a6d3b` in the file
by hand, hoping you caught them all. With tokens, it's one line in `:root`.

---

## How to talk to Claude about tokens

Reference tokens by name, not by hex code, once they exist:

- ✅ "Make `--color-accent-gold` a bit darker."
- ✅ "The starving hunger tag background feels too similar to deceased — check `--color-status-starving-bg` vs `--color-status-deceased`."
- 🚫 Avoid: "Make the `#8a6d3b` color darker" — once it's tokenized, referring to it by hex code makes Claude go find where that hex code is *defined* rather than just editing the token directly. Not wrong, just an extra step.

If you want a *new* color added to the system (not a variation of an
existing token), say so explicitly — e.g. "I want a new token for a blue
status color for the Scout warning banner" — rather than "make it blue,"
so Claude adds it to `:root` properly instead of hardcoding a one-off hex
value that falls outside the system.

---

## Color Tokens

### Backgrounds

| Token | Value | Used for |
|---|---|---|
| `--color-bg-page` | `#1b1b1b` | Page background; also used as text-on-gold color for buttons (dark text on a light gold background) |
| `--color-bg-panel` | `#262626` | Character cards, candidate cards, resource summary bar |
| `--color-bg-log-entry` | `#202020` | Day log entries — one shade darker than panel, to sit visually "behind" the roster |
| `--color-bg-candidate-selected` | `#33301f` | Background tint on a candidate card once selected |

### Text

| Token | Value | Used for |
|---|---|---|
| `--color-text-primary` | `#e8e0d0` | Default body text color (off-white/parchment) |
| `--color-text-muted` | `#bdbdbd` | Backstory flavor text — dimmer than primary |
| `--color-text-disabled` | `#999` | Text on disabled buttons |
| `--color-text-footer` | `#666` | Version footer text |

### Accent (bronze/gold — the game's signature color)

| Token | Value | Used for |
|---|---|---|
| `--color-accent-gold` | `#8a6d3b` | Primary buttons, selected candidate border, character card left-border |
| `--color-accent-gold-light` | `#c9a86a` | Primary skill labels, italic emphasis text, scout warning line |
| `--color-accent-gold-muted` | `#8a7a54` | Secondary skill labels — deliberately dimmer/smaller than primary gold |

### Borders / Dividers

| Token | Value | Used for |
|---|---|---|
| `--color-border-default` | `#555` | Default card and input borders, disabled button background |
| `--color-border-subtle` | `#333` | Footer top divider |

### Status — Hunger Tags

| Token | Value | Used for |
|---|---|---|
| `--color-status-hungry-bg` | `#6b5a1f` | "Hungry" tag background |
| `--color-status-hungry-text` | `#e6d6a6` | "Hungry" tag text |
| `--color-status-starving-bg` | `#6b2f1f` | "Starving" tag background |
| `--color-status-starving-text` | `#e6b6a6` | "Starving" tag text |
| `--color-status-deceased` | `#6b1f1f` | Deceased character card left-border |
| `--color-status-deceased-text` | `#a05050` | Deceased character name text |

### Status — Game Over Banners

| Token | Value | Used for |
|---|---|---|
| `--color-victory-bg` | `#2f4a2f` | Victory banner background |
| `--color-victory-text` | `#b6e6b6` | Victory banner text |
| `--color-victory-border` | `#5a8a5a` | Victory banner border |
| `--color-defeat-bg` | `#4a2f2f` | Defeat banner background |
| `--color-defeat-text` | `#e6b6b6` | Defeat banner text |
| `--color-defeat-border` | `#8a5a5a` | Defeat banner border |

### Status — Misc

| Token | Value | Used for |
|---|---|---|
| `--color-supplies-text` | `#7a9e7a` | "Brings: ..." supply line on candidate cards (greenish, distinct from gold accent family) |

---

## Font Tokens

| Token | Value | Used for |
|---|---|---|
| `--font-body` | `Georgia, serif` | The only font family in the file. Tokenized even though there's currently just one, so a future need (e.g. a monospace debug/dev view) is a one-line addition rather than a find-and-replace. |

---

## What's *not* tokenized yet — and why

**Spacing (padding, margin, gap) is intentionally left as hardcoded pixel
values for now** — e.g. `padding: 10px`, `margin-top: 18px`. This was a
deliberate choice, not an oversight:

- Colors and fonts were already repeated 3+ times each across the file
  with an obvious shared identity (e.g. "the gold accent," "the body font")
  — a clear signal they should be tokenized now.
- Spacing values in the current file are mostly one-off, per-component
  choices (`10px` here, `14px` there, `18px` elsewhere) that haven't yet
  settled into an obvious shared *scale* (like a 4/8/12/16/24px system).
  Tokenizing spacing before that pattern is visible risks locking in an
  arbitrary scale that doesn't match what the game actually needs once
  more screens exist.

**When to revisit this:** once more screens/components are built (Phase 2
UI, the Ring/Scouting map, etc.) and it becomes clear which spacing values
actually repeat with intent, that's the signal to add a `--space-*` token
scale the same way colors were done here. Flagging this as a deliberate
open item, not a gap.

---

*Last verified against `index.html` v0.0.1.1. Update this tag whenever
`index.html`'s version bumps AND the change touches tokens (colors, fonts,
or the token block itself) — see project instructions for the sync rule.*
