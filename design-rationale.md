# Barbarians — Prototype Design Rationale (v0.5.0.0)

*Companion to `index.html`, not to the locked design docs. Captures the
"why" behind **prototype code decisions** — balance reasoning, historical
justification, implementation intent. The design docs
(`barbarians-design-doc-*.md`) are the source of truth for what the game
*should* do; this file explains why the *current build* does what it does,
including deliberate shortcuts that haven't caught up to locked design yet.
Keep this doc's version tag in sync with `index.html` — it describes the
state of that specific file, not a moving target.*

---

## Vanguard Roster (Historical Framing)

Vanguard skews male — historically accurate for an advance/garrison party;
women and additional trades arrive later with the caravan (future
milestone, not this build).

**Farmer** has a backstory pool but is excluded from the active role pool
— you don't farm contested ground. Unlocks once the caravan arrives.

**Laborer** is retired outright — the generalist-fallback role was cut
because a reliable floor at everything removed meaningful skill gaps. See
`barbarians-build-history-20260725.md` (1b notes) for the full rationale.

**Veneur** is ranger-type: better at both Stalk beasts and Spy out
(tracking/woodcraft covers both). **Outrider** and **Woodward** are tabled
— blocked on the Threat system and Founding Sequence respectively.

## Candidate Pool and Selection

8-candidate pool, player picks 5 (`CANDIDATE_POOL_SIZE = 8`,
`VANGUARD_SIZE = 5`). `VANGUARD_SIZE` is hardcoded, not derived from
`ALL_SKILLS.length` — vanguard headcount is a design constant; adding new
roles to the pool shouldn't silently change how many men the player fields.

Pool enforces two constraints at generation:
1. **Unique full tier profiles** — no two candidates may share an identical
   role→tier map across all roles (`JSON.stringify` collision check).
2. **Headline role cap** — no single role may appear as the Seasoned
   headline more than `MAX_SAME_HEADLINE` (currently 2) times. With 7
   roles and 8 cards, one role will always appear twice; the cap prevents
   3+ clustering.

## Starting Supplies

Per-candidate rolled supplies (not a fixed pool) make selection a real
choice — "this Master Cook only brought 3 rations, that one brought 5."
Ranges are intentionally modest — personal gear, not a warehouse.

> Superseded once Founding Sequence is built: founding doc locks starting
> rations as a flat 8–15 roll, decoupled from which characters are drawn.
> Per-candidate rations are a prototype-only stand-in.

## Weapon Assignment

Weapon type by role is flavor-only until the combat rework lands.
Man-at-Arms gets a proper weapon of war; everyone else carries their
trade's tool. `armed: true` is a placeholder for future per-character
durability/loss.

**Weapon loss:** no chance of damage/loss from use in `resolveFightBack()`
— the old 20%-per-use roll contradicted "durable equipment" and guaranteed
every run's weapon stock hit zero on a one-way ratchet. Weapons now only
leave camp via raid theft when the camp is caught vulnerable (no watcher +
low fortification) — see `resolveResourceRisk()`.

## Action Skill Gating Philosophy

No action is skill-gated — any settler can attempt any action; skill
affects *how well*, never *whether*. Matches core doc's "badly but not
blocked." Poor tier is the below-floor case: a real penalty, not a hard
block.

## Tier System (v0.4.0.0)

Each character has a full `tiers` object mapping every role in `ALL_SKILLS`
to one of four values: **Seasoned / Practiced / Base / Poor**.

- **Headline role** (shown on card): always Seasoned — the one overt
  signal.
- **Remaining 6 roles**: 1 guaranteed Practiced, 1 guaranteed Poor
  (assigned to random non-headline slots before rolling), remaining 4 roll
  freely at 50% Base / 30% Practiced / 20% Poor.
- **Hints**: `practicedHint` and `poorHint` surface exactly one of each on
  the card. Additional Practiced/Poor beyond the guaranteed pair stay
  mechanically real but hidden — intended variance, not a bug.

`getSkillTier()` reads `character.tiers[role]` and maps to mechanical
bands: Seasoned→primary, Practiced→secondary, Base→base, Poor→poor. For
dual-role actions (Spy out, Keep ward), takes the better of the two tier
readings.

## Yield Rolling (General)

Skilled/unskilled yield ranges overlap but are shifted — prevents solvable
outcomes ("send the weak guy, he always gets exactly 2"). Poor tier
generally floors at 0 or near-0, making genuinely bad outcomes real rather
than cosmetic.

## Action-by-Action Rationale

**Fell timber** — Hewer (reads a tree's grain, works it efficiently).
Base tier gets some wood but wastes more of the tree; Poor tier is likely
to ruin the cut almost entirely.

**Build Palisade / Fortification Track** — see dedicated section below.

**Stalk beasts** — Veneur gets reliable yield; untrained hunting whiffs
often (base: 0–2, poor: 0–1). `GAME_BY_YIELD` buckets the catch
description (hare → boar) to match the actual yield roll — narration and
yield never contradict. Hunt's line is purely descriptive; the portion
count lives only in Master Cook's same-day output, not duplicated.

**Cook provisions** — Master Cook converts raw food into rations, and
forage food into rations at 2:1 (preserves foraged food before it spoils).
Does both in the same day if both on hand. Conversion rate is a rolled
range per tier (not a fixed multiplier) so players can't optimize against
a known number: Master Cook 1.3–1.8, Practiced 0.9–1.1, Base 0.4–0.6,
Poor 0.1–0.3. Poor tier ruins most of the food — genuinely punishing.

**Keep ward** — Man-at-Arms is primary via `SECONDARY_PRIMARY_ACTION_
SKILLS` (kept separate from `SKILL_FOR_ACTION` since Watch/Spy out support
a dual-role primary that the old single-primary model didn't anticipate).
A Man-at-Arms-tier watcher grants a bigger fight-back bonus (+3 vs +1 vs
+0 unwatched). Watch exists to track position for the scout-wave mechanic;
known-stale stand-in for the locked Watch/Threat design.

**Gather mast** — Food safe to eat raw (berries, roots, greens — not
hunted meat). Smaller yield than hunting, immediately usable. Gatherer is
primary; small chance of discovering more nearby trees regardless of tier
(luck, not skill). Poor tier almost always returns empty (0–0 range).

**Spy out** — Veneur and Man-at-Arms read different signs: Veneur judges
*how long* until arrival (tracking), less certain of numbers; Man-at-Arms
judges *how many* (military posture), less certain of timing. Both can be
wrong — human estimate, not a guaranteed readout. Veneur+Man-at-Arms same
day = combo: guaranteed accurate on both stats.

Man-at-Arms is a fully valid solo scout — day-estimate accuracy 0.6 solo,
miss capped at "off by one day" (`maxSpread = 1`) vs. the wider miss
(`maxSpread = 2`) others risk. The combo is a bonus for spare hands, not a
requirement — needed because the food economy demands Veneur hunting daily.

## Food Consumption Order

Longest-without-food eats first (protects most vulnerable, not fixed
order). Forage food drawn before rations (spoils faster, use before
wasted).

## Starvation

3 consecutive days without food = fatal. Days 1–2 show as "hungry"/
"starving" tags — visible warning before it's fatal.

## Fortification Track (Man-Day Model)

Replaces old flat 15%-per-attempt completion chance (made the palisade
trivially fast — diagnosed root cause in combat doc's "Why This System
Exists").

**Structure:** Six stages (`FORTIFICATION_STAGES`), locked man-day cost per
stage, 50 man-days total. `stageIndex` = active stage; `daysRemaining` =
man-days left on current stage only, not cumulative.

**Stage display labels** (shown in dropdown, via `FORTIFICATION_ACTION_
VERB`): Assart grounds → Delve fosse → Place palls → Erect bridge →
Construct wallwalk → Build garret. Internal stage names in
`FORTIFICATION_STAGES` are unchanged; labels are presentation only.

**Per-stage skill matching:** Earthwork stages (Bailey Grounds, Motte
Ditch) favor Fossier — not carpentry work. Timber stages (Palisade Stakes
onward) favor Carpenter. Man-days contributed per day by tier:
Seasoned 1.0 / Practiced 0.75 / Base 0.5 / Poor 0.25 (timber stages).
Earthwork uses a severity roll instead of a man-day multiplier — see below.

**Earthwork vs. timber resolution shapes differ by design:**
- *Timber (Carpenter, high-skill weight):* man-day multiplier per tier.
  A Carpenter contributes a full day's work; an untrained character
  contributes half. Poor tier at 0.25 is a real drag.
- *Earthwork (Fossier, no-skill weight):* 3-outcome severity roll
  (Minor/Partial/Lost-day), odds shifting by tier. "No-skill" means
  competent-by-default — anyone can dig a ditch — so Fossier's bonus is
  delay-avoidance (fewer partial/lost days), not extra man-days per day.
  Odds: Seasoned `{partial: 0.12, lost: 0.03}`, Practiced `{0.22, 0.07}`,
  Base `{0.30, 0.12}`, Poor `{0.45, 0.22}`. Lost-day stays a rare tail
  even at Poor — a tax, never a blocker.

**Wood cost is per-stage, not flat.** Earthwork needs no wood. Wood enters
at Palisade Stakes, same cost per stage after.

**Failure chance:** Placeholder flat chance (`FORTIFICATION_FAILURE_
CHANCE`), same odds at every stage/tier — explicitly NOT tuned. Earthwork
already uses the severity-roll model above; this flat chance applies to
timber stages only as a temporary stand-in.

**What this does NOT cover:** the raid/Threat system in the combat design
doc. Fortification only gates a flat win condition currently — doesn't yet
shift raid severity distribution, since raids are still the old Scout Wave
event, not Threat-driven.

## Win Condition — Known Mismatch

Code ends the game in victory once `stageIndex` reaches the end of
`FORTIFICATION_STAGES` (palisade completion). Core doc locks the actual
win condition as **survive until caravan arrives**, with palisade state
as one input into arrival-state, not the trigger. Known gap — Founding
Sequence and Combat/Threat systems both need to land first.

## Scout Wave (Combat Event) — Superseded, Not Yet Replaced

**Describes the current placeholder combat implementation; directly
contradicts the locked Combat & Threat design doc.** Documented so the
existing code's reasoning is legible — not the intended design going
forward. Scaffolding to be torn out, not a spec to extend.

One-time event on day 3. Resolution depends on whether Man-at-Arms was on
watch and how much palisade is built. Kept separate from daily action
resolvers — an event, not an action outcome.

Outrider count is small and visible to the player (log reports it outright)
— looser than the locked combat doc, which keeps Threat hidden but allows
raider count to surface via Spy out intel. Count is still cosmetic beyond
capping fight-back and scaling `pressureRatio`. Not fixed in this pass.

**Fighting back:** Weapons cap damage dealt. Fight-back bonus tiered by
watcher: **+3 if Man-at-Arms-primary-tier, +1 any other watcher, +0
unwatched**. `resolveFightBack(outriderCount, anyWatcher, soldierWatching)`
sources from `watchPresent()` / `soldierOnWatch()`.

**Hit chance:** Watcher is exposed — faces outriders directly. Scales down
as more outriders are driven off. `outridersRemaining` floors at 0 (not 1)
— previously `Math.max(1, ...)` left internal math believing 1 attacker
remained even after all were driven off. `pressureRatio` correctly zeroes
out; no divide-by-zero risk since `originalCount` always ≥3.

**Watch protects outdoor workers, not just in-camp ones.** `OUTDOOR_
ACTIONS` (Fell timber, Gather mast, Spy out, Stalk beasts) distinguishes
exposed characters from in-camp ones:
- **Caught in the open** (`isOutdoors && !anyWatcher`): auto-kill if camp
  has zero weapons, else rolls at full open-ground risk.
- **Warned** (`isOutdoors && anyWatcher`): watcher calls outdoor workers
  back before the raid lands — never an automatic kill, rolls at in-camp
  unwatched odds (0.05 base). `warnedAnyone` flag adds one summary log
  line.
- Keep ward itself excluded from `OUTDOOR_ACTIONS` — already braced for
  the raid, not someone needing warning.

**Personal hit chance reflects Man-at-Arms tier**, not just group-level
fight-back. `baseHitChance()` takes `soldierWatching`: watcher's own risk
0.5→0.35 when Man-at-Arms-tier; warned outdoor worker's risk 0.05→0.03
when the calling watcher is Man-at-Arms-tier. Small deliberate nudges,
not a tuned rebalance.

**Palisade mitigation:** Defensive capacity, not completion state — even a
half-built line blunts an attack. This principle survives into the locked
combat doc's "Fortification's Role in Combat," even though the mechanism
is rebuilt.

**Resource risk / looting:** Weak defense (no watcher, low palisade) →
outriders loot on the way out. Priority: weapons, then rations, then forage
food, then raw food. Wood never looted. `resolveResourceRisk()` always
returns a string — explicit "untouched" line when not vulnerable, "nothing
left worth taking" when vulnerable but empty, itemized-loss line otherwise.
Always placed in the same log position (after casualty resolution, before
survivor count).

**What the combat design doc replaces this with:** hidden accumulating
Threat with threshold telegraphing (quiet → ambient → sharpening →
committed raid), single camp-level severity roll with four-tier ladder
(Minor/Setback/Injury/Death), Watch/Scout/Fortification all feeding that
roll instead of a scripted day-3 skirmish. None of this exists in code
yet. Section stays until rebuilt, then moves to Deprecated or is removed.

## Selection Screen Visual System (v0.5.0.0)

The selection screen was reskinned to the parchment design system while the
game screen remains on the legacy dark theme. Both token layers co-exist in
`:root` — no conflict, explicit migration path.

**Why two themes in one file:** the game screen will be migrated to parchment
in a future pass, but ripping both out at once risked breaking working
gameplay. Selection screen first establishes the pattern; game screen follows
once the core loop is more stable.

**Fonts:** Google Fonts — Spectral SC (display/labels/buttons, `--font-display`)
and Spectral (italic flavor text/body, `--font-body-new`). Loaded via `@import`
at the top of `<style>`. The legacy `--font-body: Georgia` remains for game
screen elements not yet migrated.

**Portrait placeholder system:** `SILHOUETTE_SVG` is a minimal inline SVG
(generic standing figure, 72×72px, 28% opacity) defined once as a JS constant.
`portraitZoneHTML(src)` renders a real `<img>` if `src` is set, otherwise
renders the silhouette. Adding a portrait to any character is one line:
`candidate.portraitSrc = "path/to/file.png"`. No CSS changes needed — the
helper switches automatically. Portrait images will need to be committed to
the repo at whatever path is referenced (e.g. `assets/portraits/name.png`).

**`body:has()` selector:** hides the legacy page-level `<h1>` and `#versionTag`
while the selection screen is visible, without any JS. When `confirmVanguard()`
hides `#selectionScreen`, those elements reappear automatically for the game
screen. CSS-only, no state tracking needed.

**Version tag placement:** `p.sel-version` is `position: absolute` top-right
(top 14px, right 18px) inside `#selectionScreen` which is `position: relative`.
Visible for developer reference, outside the visual composition. The legacy
`#versionTag` div is kept for the game screen.

**Reroll button label logic:** shows "Reroll Characters" on first load
(before any reroll is used), then switches to "N Rerolls Remaining" /
"1 Reroll Remaining" after the first use. Logic lives in `renderCandidatePool()`
comparing `rerollsRemaining === MAX_REROLLS`.

## Known Prototype Shortcuts (Not Yet Resolved)

- Combat checks the *shared* weapons pool as a stand-in for "is anyone
  armed," not per-character ownership (flagged inline at
  `resolveScoutWave()`). Roadmap Horizon 2 item 7.
- Fortification failure chance is flat/untuned across timber stages and
  tiers (see Fortification Track section above).
