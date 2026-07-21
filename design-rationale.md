# Barbarians — Prototype Design Rationale (v0.1.0.0)

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

**Cook** — Converts raw food into rations, and can also turn forage food into
rations at a 2:1 ratio as a way to preserve foraged food before it spoils.
Does both in the same day if both are available — a Cook working the fire
isn't limited to one pot.

**Perimeter Watch** — No resource output yet; exists purely so the Soldier's
position is tracked for the scout wave mechanic (see below — this mechanic
itself is a known-stale stand-in for the locked Watch/Threat design).

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
shortcut of always reporting it outright is looser than that.)

**Fighting back:** Weapons on hand set a ceiling on how much damage the
vanguard can do; a Soldier on watch uses them far more effectively than an
unarmed or unskilled defense. Weapons are durable equipment, not ammunition —
they don't get used up by fighting, but each one used has a random chance
(20%) of being damaged or lost in the melee.

**Hit chance:** The watcher is the exposed one — he faces the outriders
directly. Everyone else is only at risk if both the watch and the wall fail
to stop the outriders from getting further in. Hit chance scales down
slightly as more outriders are driven off (fewer attackers left = less
danger).

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
