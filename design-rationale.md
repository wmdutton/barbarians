# Barbarians — Prototype Design Rationale (v0.2.0.1)

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

**Farmer** has a backstory pool but is excluded from the active skill pool
— you don't farm contested ground. Unlocks once the caravan arrives.

**Veneur** is ranger-type: better at both Hunt and Scout (tracking/woodcraft
covers both). **Laborer** is an honest generalist — no specialty bonus, but
reliably adequate at anything.

> **Known gap — selection screen is stale.** Code runs an 8-candidate pool
> with player pick-5 (`CANDIDATE_POOL_SIZE`, `confirmVanguard`). Character
> doc locks a different model: five roles drawn randomly and automatically
> from an eight-role pool, no player choice, no duplicates. Not yet rebuilt
> — see roadmap Horizon 1 step 1c.

## Starting Supplies

Per-candidate rolled supplies (not a fixed pool) make selection a real
choice — "this Master Cook only brought 3 rations, that one brought 5."
Ranges are intentionally modest — personal gear, not a warehouse.

> Superseded once Founding Sequence is built: founding doc locks starting
> rations as a flat 8–15 roll, decoupled from which characters are drawn.
> Per-candidate rations are a prototype-only stand-in.

## Weapon Assignment

Weapon type by skill is flavor-only until the combat rework lands.
Man-at-Arms gets a proper weapon of war; everyone else carries their
trade's tool. `armed: true` is a placeholder for future per-character
durability/loss.

**Weapon loss:** no chance of damage/loss from use in `resolveFightBack()`
— the old 20%-per-use roll contradicted "durable equipment" and guaranteed
every run's weapon stock hit zero on a one-way ratchet (biggest difficulty
complaint in playtesting). Weapons now only leave camp via raid theft when
the camp is caught vulnerable (no watcher + low fortification) — see
`resolveResourceRisk()`.

## Task Skill Gating Philosophy

No task is skill-gated — any settler can attempt any task; skill affects
*how well*, never *whether*. Matches core doc's "badly but not blocked."

## Yield Rolling (General)

Skilled/unskilled yield ranges overlap but are shifted — prevents solvable
outcomes ("send the weak guy, he always gets exactly 2").

## Task-by-Task Rationale

**Woodcutting** — Carpenter (eye for usable timber) and Laborer (raw
capability) both suited. Everyone else manages, less wood usable.

**Build Palisade / Fortification Track** — see dedicated section below.

**Hunt** — Skilled hunters get more raw food, reliably; unskilled hunters
get less AND risk losing some (untended kills spoil/get dragged off).
`GAME_BY_YIELD` buckets the catch description (hare → boar) to match the
actual `rawYield` roll, so narration and yield never contradict (previously
independent rolls could pair "a young boar" with "2 portions"). Hunt's line
is purely descriptive — the portion count lives only in Master Cook's
same-day output, not duplicated.

**Cook (task)** — Master Cook converts raw food into rations, and forage
food into rations at 2:1 (preserves foraged food before it spoils). Does
both in the same day if both on hand.

**Perimeter Watch** — Man-at-Arms is primary skill (`SECONDARY_PRIMARY_
TASK_SKILLS`, kept separate from `SKILL_FOR_TASK` since Watch/Scout support
a primary skill Laborer's "always secondary" rule didn't anticipate). A
Man-at-Arms-tier watcher grants a bigger fight-back bonus (+3 vs +1 vs +0
unwatched) — see Scout Wave section. Watch exists only to track position
for the scout-wave mechanic; known-stale stand-in for the locked
Watch/Threat design.

**Forage** — Food safe to eat raw (berries, roots, greens — not hunted
meat). Smaller yield than Hunt, immediately usable. Veneur's tracking helps
here too; small chance of discovering more nearby trees while covering
ground.

**Scout** — Veneur and Man-at-Arms read different signs: Veneur judges
*how long* until arrival (tracking), less certain of numbers; Man-at-Arms
judges *how many* (military posture), less certain of timing. Both can
be wrong — human estimate, not guaranteed readout. Veneur+Man-at-Arms
same day = combo: guaranteed accurate on both stats.

Man-at-Arms is a fully valid solo scout (no longer needs the Veneur combo)
— day-estimate accuracy 0.6 solo, miss capped at "off by one day"
(`maxSpread = 1`) vs. the wider miss (`maxSpread = 2`) Veneur/untrained
scouts risk. Reflects "he'd stake his read within a day either way." The
combo is now a bonus for spare hands, not a requirement for a usable Scout
result — needed because the food economy demands Veneur hunting daily, so
players never paired the two under the old all-or-nothing combo design.

## Food Consumption Order

Longest-without-food eats first (protects most vulnerable, not fixed
order). Forage food drawn before Rations (spoils, use before wasted).

## Starvation

3 consecutive days without food = fatal. Day 1/2 show as "hungry"/
"starving" tags — visible warning before it's fatal.

## Fortification Track (Man-Day Model)

Replaces old flat 15%-per-attempt completion chance (made the palisade
trivially fast regardless of labor assigned — diagnosed root cause in
combat doc's "Why This System Exists").

**Structure:** Six stages (`FORTIFICATION_STAGES`), locked man-day cost per
stage, 50 man-days total. `stageIndex` = active stage; `daysRemaining` =
man-days left on current stage only, not cumulative.

**Per-stage skill matching:** Earthwork stages (Bailey Grounds, Motte
Ditch) favor Laborer over Carpenter — not carpentry work. Timber stages
(Palisade Stakes onward) favor Carpenter. Tier changes man-days contributed
per day (1 / 0.75 / 0.5 for Seasoned/Practiced/Base-or-worse) — no roll for
guaranteed completion; even Seasoned isn't guaranteed a full day's progress.

**Wood cost is per-stage, not flat.** Earthwork needs no wood. Wood enters
at Palisade Stakes, constant per stage after.

**Failure chance:** Placeholder flat chance (`FORTIFICATION_FAILURE_
CHANCE`), same odds at every stage/tier — explicitly NOT tuned. Founding
doc's open flags call out per-stage odds/skill mitigation as unresolved.

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
watch and how much palisade is built. Kept separate from daily task
resolvers — an event, not a task outcome.

Outrider count is small and visible to the player (log reports it
outright) — looser than the locked combat doc, which keeps Threat hidden
but allows raider count to surface via Scouting-derived intel. Count is
still cosmetic beyond capping fight-back and scaling `pressureRatio` — a
wave of 3 and 5 play out nearly identically. Flagged in playtesting as
arbitrary; exactly the gap the locked Combat doc's Threat-driven
raider-count scaling closes. Not fixed in this pass.

**Fighting back:** Weapons cap damage dealt. Fight-back bonus tiered by
watcher: **+3 if Man-at-Arms-primary-tier, +1 any other watcher, +0
unwatched** (replaces old flat +2). `resolveFightBack(outriderCount,
anyWatcher, soldierWatching)` sources from `watchPresent()` /
`soldierOnWatch()`.

**Hit chance:** Watcher is exposed — faces outriders directly. Scales down
as more outriders are driven off. `outridersRemaining` floors at 0 (not 1)
— previously `Math.max(1, ...)` meant driving off every attacker still
left internal math believing 1 remained, contradicting "drove off all of
them" log text. `allDrivenOff` now gives explicit "repelled outright" text;
`pressureRatio` correctly zeroes out (no divide-by-zero risk —
`originalCount` always ≥3 from `rollOutriderCount()`).

**Watch protects outdoor workers, not just in-camp ones.** `OUTDOOR_TASKS`
(Hunt, Forage, Scout, Woodcutting) distinguishes exposed characters from
in-camp ones:
- **Caught in the open** (`isOutdoors && !anyWatcher`): auto-kill if camp
  has zero weapons, else rolls at full open-ground risk.
- **Warned** (`isOutdoors && anyWatcher`): any watcher (not Man-at-Arms-
  tier specifically) calls outdoor workers back before the raid lands —
  never an automatic kill, rolls at in-camp unwatched odds (0.05 base) not
  full open-ground risk. `warnedAnyone` flag adds one summary log line.
- Perimeter Watch itself excluded from `OUTDOOR_TASKS` — already braced
  for the raid, not someone needing to be called back.

**Personal hit chance reflects Man-at-Arms tier**, not just group-level
fight-back. `baseHitChance()` takes `soldierWatching`: watcher's own risk
0.5→0.35 when Man-at-Arms-tier (same exposure, better trained); warned
outdoor worker's risk 0.05→0.03 when the calling watcher is Man-at-Arms-
tier (same skill underlying his tighter Scout estimate). Both numbers are
small deliberate nudges, not a tuned rebalance.

**Palisade mitigation:** Not "% complete toward a finished wall" — even a
half-built line blunts an attack. Defensive capacity, not completion
state. (This principle — fortification shifts outcome shape, not a binary
gate — is the one piece surviving into the locked combat doc's
"Fortification's Role in Combat," even though the mechanism is rebuilt.)

**Resource risk / looting:** Weak defense (no watcher, low palisade) →
outriders loot on the way out. Priority: weapons, then rations, then
forage food, then raw food. Wood never looted (not hauling lumber).
`resolveResourceRisk()` always returns a string — explicit "untouched"
line when not vulnerable, explicit "nothing left worth taking" when
vulnerable but empty-handed, itemized-loss line otherwise. Always placed
in the same log position (after casualty resolution, before survivor
count).

**What the combat design doc replaces this with:** hidden accumulating
Threat with threshold telegraphing (quiet → ambient → sharpening →
committed raid), single camp-level severity roll with four-tier ladder
(Minor/Setback/Injury/Death), Watch/Scout/Fortification all feeding that
roll instead of a scripted day-3 skirmish. None of this exists in code
yet. Section stays until rebuilt, then moves to Deprecated or is removed.

## Known Prototype Shortcuts (Not Yet Resolved)

- Combat checks the *shared* weapons pool as a stand-in for "is anyone
  armed," not per-character ownership (flagged inline at
  `resolveScoutWave()`). Roadmap Horizon 2 item 7.
- Fortification failure chance is flat/untuned across all stages and
  tiers (see Fortification Track section above).
