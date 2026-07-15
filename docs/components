# Barbarians — Components

*Every named UI piece in `index.html`: what it's called, what it looks like,
what JS function renders/controls it, and what it in turn affects. Use this
doc to tell Claude "change the X component" and have it land on the right
CSS selector and the right function on the first try. Colors referenced
below are tokens — see [tokens.md](./tokens.md) for values.*

---

## How to talk to Claude about components

Refer to a component by the name in **bold** at the start of each entry
below (e.g. "the **candidate card**" or "the **hunger tag**"). That name
maps directly to a CSS selector, so Claude can go straight to the right
rule instead of guessing which `<div>` you mean.

If you want to change *behavior* (not just appearance), mention the
function name if you know it (e.g. "change how `resolveHunt` picks flavor
text") — otherwise just describe what you want changed about the component
and Claude will find the controlling function from this doc.

---

## Screen 1 — Vanguard Selection

This is the first screen shown. Controlled by `#selectionScreen` (hidden
once a vanguard is confirmed, via `confirmVanguard()`).

### Selection instructions
- **Selector:** `#selectionInstructions`
- **What it is:** The line of italic text above the candidate pool ("Choose 5 men for the vanguard. Selected: X / 5.")
- **Rendered by:** `renderCandidatePool()` — rewritten every time selection changes
- **Controls:** Nothing downstream; pure status display.

### Candidate pool
- **Selector:** `#candidatePool` (container), `.candidate` (each card)
- **What it is:** The flex-wrap grid of up to 8 selectable candidate cards.
- **Rendered by:** `renderCandidatePool()`, which loops over the `candidatePool` array (module-level variable, not part of `gameState` — it only exists pre-game) and builds one `.candidate` div per entry.
- **Controlled by:** `generateCandidatePool()` (initial 8), `rerollCandidates()` (regenerates unselected ones)
- **Controls:** Clicking a card calls `toggleCandidateSelection()`, which updates `selectedIds` and re-renders.

### Candidate card
- **Selector:** `.candidate` (base), `.candidate.selected` (selected state)
- **What it is:** One candidate's info block — name, primary skill, secondary skill (if any), backstory, and starting supplies.
- **Sub-parts:**
  - `.candidate .name` — candidate's name, bold
  - `.candidate .skill` — primary skill, gold italic (`--color-accent-gold-light`)
  - `.candidate .secondary-skill` — secondary skill, smaller/dimmer gold (`--color-accent-gold-muted`). **Omitted entirely for Laborer candidates** — Laborer has no secondary skill by design (see design-rationale.md).
  - `.candidate .backstory` — flavor text sentence
  - `.candidate .supplies` — "Brings: X rations, Y wood" line, green-tinted (`--color-supplies-text`)
- **Rendered by:** `renderCandidatePool()`, built from a `createSettler()` object
- **Controls:** Click toggles selection (see above); selection count gates the Confirm button.

### Reroll button
- **Selector:** `#rerollBtn`
- **What it is:** "Reroll Remaining Candidates" button.
- **Rendered by:** Static HTML; enabled/disabled state set by `renderCandidatePool()` based on `rerollsRemaining`
- **Controls:** Click calls `rerollCandidates()` — regenerates unselected candidates, decrements `rerollsRemaining` (starts at `MAX_REROLLS` = 3).

### Reroll count
- **Selector:** `#rerollCount`
- **What it is:** "Rerolls remaining: N" text next to the reroll button.
- **Rendered by:** `renderCandidatePool()`

### Confirm Vanguard button
- **Selector:** `#confirmVanguardBtn`
- **What it is:** Locks in the 5 selected candidates and starts the game.
- **Rendered by:** Static HTML; enabled only when `selectedIds.length === 5` (set in `renderCandidatePool()`)
- **Controls:** Click calls `confirmVanguard()` — hides `#selectionScreen`, shows `#gameScreen`, applies starting supplies, does first render of the game screen.

---

## Screen 2 — Game Screen

Controlled by `#gameScreen` (hidden until `confirmVanguard()` runs).

### Resource summary bar
- **Selector:** `#resourceSummary`
- **What it is:** The top strip showing Day, Rations, Raw Food, Forage Food, Wood, Weapons, Trees Left, Palisade %, and (if active) the scout warning line.
- **Rendered by:** `renderResources()` — pure read of `gameState.resources`, `gameState.environment`, and `gameState.scoutWarning`. Called after every `startDay()`.
- **Controls:** Nothing; read-only display. The ⚠ warning line only appears when `gameState.scoutWarning` is non-null (set by `resolveScoutForEnemiesForDay()`, cleared after each wave resolves).

### Game over banner
- **Selector:** `#gameOverBanner`, with modifier classes `.game-over.victory` / `.game-over.defeat`
- **What it is:** The full-width victory/defeat message shown once the game ends, plus the "Start a New Vanguard" reset button.
- **Rendered by:** `renderGameOverIfNeeded()`, called at the end of `startDay()`. Only renders content once `gameState.gameOver` is true (set by `checkGameOver()`).
- **Controls:** Its "Start a New Vanguard" button (`#resetBtn`) calls `location.reload()` — a full page refresh, not a soft reset. Also disables `#startDayBtn` so no further days can be played.

### Roster
- **Selector:** `#roster` (container), `.character` (each card)
- **What it is:** The list of all 5 vanguard members (or fewer, as they die), each with a task-assignment dropdown.
- **Rendered by:** `renderRoster()`, called after every `startDay()` and after `confirmVanguard()`. Loops over `gameState.characters`.

### Character card (alive)
- **Selector:** `.character` (base)
- **What it is:** One living settler's card — name, primary skill, secondary skill (if any), hunger tag (if any), backstory, and a task dropdown.
- **Sub-parts:**
  - `.character .name` — bold name
  - `.character .skill` — primary skill, gold italic
  - `.character .secondary-skill` — secondary skill, smaller/dimmer. **Omitted for Laborer**, same rule as the candidate card.
  - `.hunger-tag` (`.hungry` or `.starving` modifier) — only rendered when `hungerStage()` returns non-null
  - `.character .backstory` — flavor text
  - `.character .task-row` — contains the `<select>` task dropdown
- **Rendered by:** `renderRoster()`
- **Controls:** The task `<select>` fires `handleTaskChange()` on change, which sets `character.currentTask` and calls `updateStartDayAvailability()`.

### Character card (deceased)
- **Selector:** `.character.deceased`
- **What it is:** A simplified card for a dead settler — dimmed (`opacity: 0.6`), red-tinted border (`--color-status-deceased`), name struck through in red text (`--color-status-deceased-text`), no dropdown.
- **Rendered by:** `renderRoster()`, when `character.alive` is `false`

### Hunger tag
- **Selector:** `.hunger-tag`, with `.hungry` or `.starving` modifier
- **What it is:** Small uppercase pill shown on a character card when they've missed a meal.
- **Rendered by:** `renderRoster()`, driven by `hungerStage(character)` — returns `"hungry"` at 1 day without food, `"starving"` at 2+, `null` (no tag) at 0. 3 days is fatal (handled by `checkStarvation()`, not this display function).

### Task dropdown
- **Selector:** `select` (inside `.task-row`)
- **What it is:** The per-character task assignment control. Options come from `TASK_POOL` (Woodcutting, Build Palisade, Hunt, Cook, Perimeter Watch, Forage, Scout).
- **Rendered by:** `renderRoster()`
- **Controls:** `handleTaskChange()` on change → `updateStartDayAvailability()`, which enables/disables `#startDayBtn`.

### Start Day button
- **Selector:** `#startDayBtn`
- **What it is:** The main turn-advance button.
- **Rendered by:** Static HTML; enabled/disabled by `updateStartDayAvailability()` (only enabled once every living character has a task)
- **Controls:** Click calls `startDay()` — the top-level function that runs `runDay()` (all task/combat/consumption resolution), then re-renders resources, log, roster, and checks for game over.

### Assignment status
- **Selector:** `#assignmentStatus`
- **What it is:** The italic status line below Start Day ("All settlers assigned..." or "N settler(s) still need a task.")
- **Rendered by:** `updateStartDayAvailability()`

### Day log
- **Selector:** `#dayLog` (container), `.day-entry` (each day's block)
- **What it is:** The scrolling history of every resolved day, most recent first.
- **Rendered by:** `renderLog()`, called after every `startDay()`. Reads `gameState.log` (array of `{ day, entries }` objects built up by `runDay()`).
- **Sub-parts:**
  - `.day-entry .day-title` — "Day N" header, gold
  - `.day-entry ul` — bullet list of that day's narrative log lines

---

## Global / Shared

### Version footer
- **Selector:** `#versionFooter`
- **What it is:** Small centered text at the bottom of the page showing version, author, date.
- **Rendered by:** Static HTML — manually updated by hand each version bump (not JS-driven).

### Task select (shared style)
- **Selector:** `select` (bare tag selector — applies to every dropdown in the app)
- **What it is:** The shared visual style (dark background, bordered) for all `<select>` elements, currently just the per-character task dropdown.

---

## Mobile responsiveness

All of the above components get layout adjustments (not new components) at
viewports ≤600px, defined in the `@media (max-width: 600px)` block near the
bottom of the `<style>` tag. This block changes sizing/stacking only — it
introduces no new colors or components, so nothing here needed updating
when tokens were added.

---

*Last verified against `index.html` v0.0.1.1. Update this tag whenever
`index.html`'s version bumps AND the change adds/renames a component or
changes what function controls it — see project instructions for the sync
rule.*
