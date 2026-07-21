# Barbarians — Prototype Design Rationale (v0.2.0.0)

*Companion to `index.html`, not to the locked design docs. This file
captures the "why" behind **prototype code decisions** — balance
reasoning, historical justification, and implementation intent that
isn't needed to read the code but matters if you're changing it.
The design docs (`barbarians-design-doc-*.md`) are the source of truth
for what the game *should* do; this file explains why the *current
build* does what it does, including deliberate shortcuts that haven't
caught up to locked design yet. Update the version tag above whenever
this doc is revised for a later build, and keep it in sync with
`index.html`'s version — this doc describes the state of that specific
file, not a moving target.*

---

## Vanguard Roster (Historical Framing)

The starting party is a military vanguard sent ahead of the main settlement
group to secure ground before the supply caravan arrives. Historically,
advance/garrison parties like this skewed male — women and additional trades
arrive later with the caravan. That later arrival is a planned future
milestone, not this build.

**Farmer** is defined (has a backstory pool ready) but deliberately excluded
from the vanguard's active skill pool — you don't farm contested ground. That
skill unlocks once the supply caravan arrives and the site is actually secure.

**Hunter** is a ranger-type archetype: better at both Hunt and Scout (tracking
and woodcraft cover both). **Laborer** is an honest generalist — no specialty
bonus anywhere, but reliably adequate at anything, unlike gambling on a Cook
doing Woodcutting.

> **Known gap — selection screen is stale.** The code still runs an
> 8-candidate pool with player pick-5 (`CANDIDATE_POOL_SIZE`,
> `confirmVanguard`). The character design doc locks a different model:
> five roles drawn randomly and automatically from an eight-role pool,
> no player choice, no duplicates (see "Selection: No Choice" in the
> character doc). This rewrite hasn't landed yet — the roles/archetypes
> referenced below (Hunter, Laborer, etc.) predate that doc's Role Pool
> naming (Veneur, Gatherer, Outrider, etc.) and will need remapping when
> the selection screen is rebuilt, not just the selection mechanic.

## Starting Supplies

Rolling starting supplies per-candidate (not a fixed pool) is what makes
vanguard selection a real choice — not just "which Cook" but "this Cook only
brought 3 rations, that one brought 5." Ranges are intentionally modest —
personal gear/provisions, not a warehouse.

> Superseded by design once the Founding Sequence is built: the founding
> doc locks starting rations as a flat 8–15 roll, decoupled entirely from
> which characters are drawn. Per-candidate rations are a prototype-only
> stand-in until Founding replaces the whole pre-Day-1 flow.

## Weapon Assignment

Weapon type by skill is currently flavor-only; it will matter mechanically
once the combat rework lands (see Fortification/Combat section below).
Soldier gets a proper weapon of war; everyone else carries whatever their
trade puts in their hands. `armed: true` on settlers is a placeholder for
when weapon durability/loss is built per-character.

**Weapon loss (updated 2026-07-21):** weapons no longer have any chance
of being damaged/lost from use in `resolveFightBack()` — that 20%-per-use
roll was removed. It was already a contradiction of this section's own
stated intent ("weapons are durable equipment, not ammunition") and, in
practice, guaranteed every run's weapon stock eventually hit zero on a
one-way ratchet with no way back up, which was the single biggest driver
of difficulty complaints in playtesting. The ONLY way weapons now leave
camp is theft during a raid where the camp was caught vulnerable (no
watcher + low fortification) — see `resolveResourceRisk()`. This makes
"keep weapons" purely a function of "stay watched and fortify," not a
countdown independent of player action.

## Task Skill Gating Philosophy

No task is skill-gated — any settler can attempt any task. Skill only affects
*how well* they do it, never whether they can attempt it at all. This is a
deliberate prototype-wide rule, not an oversight, and matches the "badly but
not blocked" principle in the core design doc.

## Yield Rolling (General)

Skilled and unskilled yield ranges deliberately overlap but are shifted — a
skilled worker can have a bad day, an unskilled one can get lucky. This keeps
outcomes from becoming solvable ("send the weak guy, he always gets exactly
2").

## Task-by-Task Rationale

**Woodcutting** — Carpenter and Laborer are both suited: a Carpenter for an
eye toward usable timber, a Laborer for sheer physical capability. Everyone
else manages, but less of the tree becomes usable wood.

**Build Palisade / Fortification Track** — See the dedicated section below;
this replaced the old flat-percent model referenced in earlier versions of
this doc.

**Hunt** — Skilled hunters get more raw food, reliably. Unskilled hunters get
less AND risk losing some of it — dressing a kill takes longer without
practice, and untended meat can spoil or get dragged off before it makes it
back to camp. (Future idea: a smokehouse-type building could reduce this
risk.)

**Updated 2026-07-21 — two playtesting-reported issues fixed together.**
First: `catch_` (the animal named in the log line) was previously rolled
independently of `rawYield`, so "a young boar" — which should read as a
big haul — could show up next to "2 portions," while "a brace of
pheasant" — which should read as small — could show up next to "6
portions." Game description and stated yield contradicted each other.
A new `GAME_BY_YIELD` table now buckets the catch description to match
the actual roll (hare/squirrels at the low end, up through boar/stag at
the high end). Second: Hunt's own line no longer states a portion count
at all — it only describes the catch now ("brought down a young boar").
The portion number lived on in Cook's same-day output regardless
("N rations set aside, M from the day's meat"), so a single day's log
was stating the same underlying number twice, once as "portions" and
again as "rations" — read as the game repeating itself. The number now
lives in exactly one place (Cook's line); Hunt is purely descriptive.

**Cook** — Converts raw food into rations, and can also turn forage food into
rations at a 2:1 ratio as a way to preserve foraged food before it spoils.
Does both in the same day if both are available — a Cook working the fire
isn't limited to one pot.

**Perimeter Watch** — Previously had no skill mapping at all (Soldier got
no bonus for the one task most thematically his). Updated 2026-07-21:
Soldier is now the primary skill for Perimeter Watch, via a new
`SECONDARY_PRIMARY_TASK_SKILLS` lookup (kept separate from
`SKILL_FOR_TASK` since Watch and Scout both now support a primary skill
that Laborer's "always secondary" rule and the Fortification stage
override didn't need to anticipate). A Soldier-tier watcher grants a
bigger fight-back bonus during a raid (+3 vs +1 for a non-Soldier
watcher, +0 unwatched) — see Scout Wave section below. Watch still exists
purely so position is tracked for the scout wave mechanic; this remains
a known-stale stand-in for the locked Watch/Threat design.

**Forage** — Finds food that's safe to eat without cooking (berries, roots,
greens — not hunted meat). Smaller yield than Hunt but immediately usable. A
Hunter's tracking/woodcraft helps here too. Has a smaller chance of
discovering more nearby trees while covering ground.

**Scout** — Hunter and Soldier read the signs from different angles. A Hunter
tracks — better at judging *how long* until the group arrives, less certain
of exact numbers. A Soldier reads military posture — better at judging *how
many*, less certain of timing. Both can be wrong; this is a human estimate,
not a guaranteed readout. If a Hunter AND a Soldier are both sent the same
day, that's a combo: together they get a certain reading on both stats — one
confirms what the other suspects. `trueCount`/`trueDaysOut` are rolled once
as a forecast the actual wave should closely match, though not necessarily
exactly (scouts, not oracles).

**Updated 2026-07-21 — Soldier no longer needs the Hunter combo to
scout.** Originally the strong "guaranteed accurate" read only fired if
Hunter AND Soldier scouted together, meaning Soldier's day-estimate
accuracy solo was mediocre (0.45) by design, to push players toward
pairing him with a Hunter. In practice this never happened: the food
economy needs the Hunter hunting every day, so no player was ever going
to give that up to double up on Scout, and the combo mechanic sat
unused. Soldier is now a fully valid solo scout — day-estimate accuracy
raised to 0.6, and critically, his miss is capped at "off by one day"
(`maxSpread = 1`) rather than the wider miss (`maxSpread = 2`) Hunter and
untrained scouts risk. This directly implements "he'd stake his read
within a day either way" — even a failed accuracy roll for Soldier stays
close. The Hunter+Soldier combo still exists and still gives a fully
guaranteed read on both stats — it's now a bonus for a player who
happens to have spare hands, not a requirement to get any usable Scout
result at all.

## Food Consumption Order

Whoever has gone longest without food eats first — protects the most
vulnerable rather than feeding in a fixed order. Food is drawn from Forage
food first (since it spoils and should be used before wasted), then Rations.

## Starvation

3 consecutive days without food is fatal. 1 and 2 days show as "hungry" and
"starving" states on the character card — a visible warning the player can
act on before it's fatal.

## Fortification Track (Man-Day Model)

Replaces the old flat 15%-per-attempt completion chance, which made the
palisade trivially fast to finish regardless of labor assigned (this was the
diagnosed root cause flagged in the combat design doc's "Why This System
Exists" section).

**Structure:** Six stages (`FORTIFICATION_STAGES`), each with a locked
man-day cost, summing to 50 man-days total. `stageIndex` tracks which stage
is active; `daysRemaining` tracks man-days left on the *current* stage only,
not cumulative progress. Stage costs and order are locked per the core/
combat docs — see that array's inline comments for exact numbers.

**Per-stage skill matching:** Earthwork stages (Bailey Grounds, Motte Ditch)
favor Laborer's generic hard-labor competence over Carpenter's specialized
woodworking — clearing ground and digging a ditch isn't carpentry. Timber
stages (Palisade Stakes onward) favor Carpenter. Skill tier changes how many
man-days a character contributes per day (1 / 0.75 / 0.5 for Seasoned/
Practiced/Base-or-worse) — it does not roll for guaranteed completion, and a
Seasoned worker still isn't guaranteed a full man-day's progress on any given
day.

**Wood cost is per-stage, not flat.** Earthwork stages need no wood — you
don't need timber to clear ground or dig a ditch. Wood enters the track at
Palisade Stakes and stays constant per stage after that. This corrected an
earlier version where wood cost was flat across all stages regardless of
whether the stage was earthwork or timber.

**Failure chance:** Placeholder flat chance (`FORTIFICATION_FAILURE_CHANCE`)
that a day's labor is lost entirely to bad luck (rot, weather, bad ground),
same odds at every stage/tier for now, with per-stage flavor text on the rare
failure. This is explicitly NOT tuned — the founding doc's open flags call
out per-stage odds and skill mitigation on failure as an unresolved question.
Don't read the current flat number as a locked decision.

**Updated 2026-07-21 — "work has only just begun" no longer repeats
every day.** `buildPalisadeFlavor()`'s top band (>66% of a stage's
man-days remaining) previously fired that exact line on every single
day of work while the stage sat above that threshold — for a
multi-day stage like Motte Ditch (6 man-days), that could mean the
same "only just begun" text 3-4 days running, which read as the game
not tracking its own progress. Fixed with a new `hasAnnouncedStart`
flag on `gameState.fortification`, reset to `false` whenever
`stageIndex` advances. The line now fires once — the first day worked
in the top band for that stage — then falls through to a plainer
"work continues" line for the rest of the band, before naturally
progressing to the existing "coming along" / "nearly finished" bands
as real progress is made.

**What this does NOT yet cover:** the actual raid/Threat system described in
the combat design doc. Fortification currently only gates a flat win
condition (see below) — it doesn't yet shift raid severity distribution,
because raids in this build are still the old Scout Wave event, not
Threat-driven.

## Win Condition — Known Mismatch

Code currently ends the game in victory once `stageIndex` reaches the end of
`FORTIFICATION_STAGES` — i.e., palisade completion. The core design doc locks
a different win condition: **survive until the caravan arrives**, with
palisade state being one input into what shape the game is in when the
caravan shows up, not the trigger itself. This is a known, tracked gap, not
an oversight to silently work around — the Founding Sequence and Combat/
Threat systems both need to land before a caravan-arrival win condition makes
sense to build.

## Scout Wave (Combat Event) — Superseded, Not Yet Replaced

**This entire section describes the current placeholder combat implementation
and directly contradicts the locked Combat & Threat design doc.** It's
documented here so the reasoning behind the *existing* code is legible, not
because it's the intended design going forward. Treat this as scaffolding to
be torn out, not a spec to extend.

A one-time event on day 3, per the original brief. A small enemy scout group
probes the camp. Resolution depends on whether a Soldier was on watch and how
much palisade is built. Kept deliberately separate from daily task resolvers
— it's an event, not a task outcome.

Outrider count is kept small and visible to the player — not a hidden
number, but a fact the log reports, so the player understands the scale of
the threat. (The locked combat doc keeps Threat itself hidden but does allow
raider count to be legible via Scouting-derived intel — this prototype
shortcut of always reporting it outright is looser than that.) As of
2026-07-21, this count is still cosmetic beyond capping fight-back and
scaling `pressureRatio` — a wave of 3 and a wave of 5 play out nearly
identically otherwise. This was flagged directly in playtesting as feeling
arbitrary. It's an accurate read of the current code, not a misplay, and
is exactly the gap the locked Combat doc's Threat-driven raider-count
scaling is meant to close. Not fixed in this pass — noted here so it isn't
mistaken for already-solved.

**Fighting back:** Weapons on hand set a ceiling on how much damage the
vanguard can do. **Updated 2026-07-21:** the per-weapon 20%-loss-on-use
roll has been removed entirely — see "Weapon Assignment" above for why.
The fight-back bonus is now tiered by who's watching, not a flat "any
watcher" bonus: **+3 if the watcher is Soldier-primary-tier, +1 for
anyone else on Watch, +0 unwatched.** This replaces the old flat +2.
`resolveFightBack()` now takes `(outriderCount, anyWatcher,
soldierWatching)` — two separate watch-state booleans rather than one —
sourced from the new `watchPresent()` / `soldierOnWatch()` helpers
(renamed from the misleadingly-named `hasSoldierOnWatch()`, which never
actually checked for a Soldier specifically).

**Hit chance:** The watcher is the exposed one — he faces the outriders
directly. Hit chance scales down slightly as more outriders are driven
off (fewer attackers left = less danger).

**Updated 2026-07-21 — outridersRemaining no longer floors at 1.**
Previously `outridersRemaining = Math.max(1, outriderCount - driven)`
meant that driving off every attacker still left the internal math
believing 1 was still out there, even though the log line said "drove
off ${driven} of them" with `driven === outriderCount`. This is the
root cause of a playtesting-reported bug ("5 outriders approached,
only 4 driven off, what happened to the 5th?") — the text never
actually claimed a 5th was unaccounted for, but the internal
`pressureRatio` was quietly behaving as if one remained. Floor is now
`Math.max(0, ...)`, a new `allDrivenOff` check gives explicit "raid
was repelled outright" text when it happens, and `pressureRatio`
correctly zeroes out at 0 remaining — no divide-by-zero risk since
`originalCount` (outriderCount) is always ≥3 from `rollOutriderCount()`.

**Updated 2026-07-21 — Watch now protects outdoor workers, not just
in-camp ones.** Previously, ANY character not on Perimeter Watch was
treated identically for the "caught in the open" auto-kill check,
whether they'd spent the day hunting a mile from camp or cooking by the
fire — the only thing that mattered was whether a watcher existed at
all. This didn't match the fiction (why would a cook standing at the
hearth be "caught in the open" the same as a hunter out past the
treeline?) and it meant staffing Watch felt like it was protecting
people who were never really at risk while doing nothing legible for
the people who plausibly were.

A new `OUTDOOR_TASKS` list (Hunt, Forage, Scout, Woodcutting) now
distinguishes truly exposed characters from in-camp ones:
- **Caught in the open** (`isOutdoors && !anyWatcher`): the old
  behavior — auto-kill if the camp has zero weapons, otherwise rolls at
  full open-ground risk.
- **Warned** (`isOutdoors && anyWatcher`): a manned Watch — any
  watcher, doesn't need to be Soldier-tier for this part — is assumed
  to call outdoor workers back before the raid lands. They are never an
  automatic kill regardless of weapon count, and roll at the same odds
  as an in-camp unwatched character (0.05 base, per `baseHitChance`),
  not full open-ground risk. This is the mechanical answer to a
  specific playtesting complaint: characters sent out to Hunt/Forage/
  Woodcutting were functionally unprotected all game even with a
  watcher posted, which pushed players toward never sending anyone
  outside camp at all. A `warnedAnyone` flag adds one summary log line
  when this fires, so the protection is legible, not silent.
- Perimeter Watch itself is deliberately excluded from `OUTDOOR_TASKS`
  — the watcher is the one person already braced for the raid, not
  someone who needs to be called back to safety.

**Updated 2026-07-21 — personal hit chance now also reflects Soldier
tier, not just the group-level fight-back bonus.** Previously a
Soldier standing watch faced the exact same personal risk (0.5 base)
as anyone else on watch — his new fight-back bonus (+3 driven off)
helped the whole camp, but he was mechanically no safer standing the
post himself, which read as inconsistent once Watch became a real
skill lane for him. `baseHitChance()` now takes a `soldierWatching`
flag and applies it two places: the watcher's own risk drops from 0.5
to 0.35 when he's Soldier-tier (same exposure, better trained), and a
warned outdoor worker's risk drops from 0.05 to 0.03 when the watcher
calling them back is Soldier-tier (his read on approaching danger is
the same skill underlying his tighter Scout estimate). Neither number
is precisely tuned — both are small, deliberate nudges in the direction
"Soldier-on-Watch should feel like a real upgrade everywhere Watch
matters," not a rebalance pass.

**Palisade mitigation:** Deliberately not framed as "% complete toward a
finished wall" — even a half-built line of stakes and logs blunts an attack.
The palisade's real value is defensive capacity, not completion state. (This
intent — fortification shifting outcome shape rather than being a binary gate
— is the one piece of this section that *does* survive into the locked combat
doc's "Fortification's Role in Combat," even though the mechanism around it
will be rebuilt.)

**Resource risk / looting:** If the camp's defense is weak (no watcher, low
palisade), outriders loot on the way out. Priority order: weapons first
(worth the most, easiest to grab), then rations, then forage food, then raw
food. Wood is never looted — a raiding party isn't hauling lumber.

**Updated 2026-07-21 — always reports an outcome.** `resolveResourceRisk()`
previously returned `null` (no log line at all) both when the camp wasn't
"vulnerable" and when it was vulnerable but had nothing left to steal.
This meant the player could never distinguish "nothing was taken" from
"I just didn't notice the line" — flagged directly in playtesting as
confusing, especially now that weapons can ONLY be lost this way (see
"Fighting back" above). The function now always returns a string: an
explicit "stores went untouched" line when not vulnerable, an explicit
"nothing left worth taking" line when vulnerable but empty-handed, and
the existing itemized-loss line otherwise. `resolveScoutWave()` also now
places this line in the same position every raid (immediately after
casualty resolution, before the survivor count), rather than wherever it
happened to fall — findability was part of the complaint, not just
existence.

**What the combat design doc replaces this with:** a hidden, accumulating
Threat value with threshold-based telegraphing (quiet → ambient signal →
sharpening signal → committed raid), a single camp-level severity roll with
a four-tier ladder (Minor/Setback/Injury/Death), and Watch/Scout/Fortification
all feeding that roll rather than resolving a scripted day-3 skirmish. None
of that exists in code yet. This section stays until it's rebuilt, at which
point it should move to a "Deprecated" section or be removed outright.

## Known Prototype Shortcuts (Not Yet Resolved)

- Combat currently checks the *shared* weapons pool as a stand-in for "is
  anyone armed at all," until weapons move to per-character ownership in a
  future session. Once that lands, individual armed state should gate
  individual risk instead. (Flagged inline in code at `resolveScoutWave()`.)
- Vanguard selection is still pick-5-of-8 with player choice, not the locked
  random-draw-of-5 model. See "Vanguard Roster" note above.
- Win condition is palisade completion, not caravan arrival. See "Win
  Condition" section above.
- Fortification failure chance is flat/untuned across all stages and tiers.
