# Barbarians — Prototype Design Rationale (v0.0.0.0)

*Extracted from inline code comments in `index.html` (v0.0.0.0) on 2026-07-12,
to keep the production file lean. This doc captures the "why" behind
prototype systems — balance reasoning, historical justification, and design
intent that isn't needed to read/maintain the code itself. Pair with
`barbarians-design-doc.md` for the broader locked design; this file is
prototype-specific implementation rationale that hasn't necessarily been
promoted to that doc. Update the version tag above when this doc is revised
for a later build.*

---

## Wave Scheduling

The first outrider wave and the gap between waves are both randomized (1–3
days), so the player can't just ignore watch/scouting until a known day.
`nextWaveDay` is rerolled fresh after each wave resolves.

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

## Starting Supplies

Rolling starting supplies per-candidate (not a fixed pool) is what makes
vanguard selection a real choice — not just "which Cook" but "this Cook only
brought 3 rations, that one brought 5." Ranges are intentionally modest —
personal gear/provisions, not a warehouse.

## Weapon Assignment

Weapon type by skill is currently flavor-only; it will matter mechanically
once the combat rework lands (see build backlog). Soldier gets a proper
weapon of war; everyone else carries whatever their trade puts in their
hands. `armed: true` on settlers is a placeholder for when weapon
durability/loss is built per-character.

## Task Skill Gating Philosophy

No task is skill-gated — any settler can attempt any task. Skill only affects
*how well* they do it, never whether they can attempt it at all. This is a
deliberate prototype-wide rule, not an oversight.

## Yield Rolling (General)

Skilled and unskilled yield ranges deliberately overlap but are shifted — a
skilled worker can have a bad day, an unskilled one can get lucky. This keeps
outcomes from becoming solvable ("send the weak guy, he always gets exactly
2").

## Task-by-Task Rationale

**Woodcutting** — Carpenter and Laborer are both suited: a Carpenter for an
eye toward usable timber, a Laborer for sheer physical capability. Everyone
else manages, but less of the tree becomes usable wood.

**Build Palisade** — Real skill gap by design. A trained Carpenter makes real
progress; anyone else is slow, clumsy improvised work. Wood cost is the same
regardless of skill — the waste shows up in progress made, not material
consumed.

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
position is tracked for the scout wave mechanic.

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

## Scout Wave (Combat Event)

A one-time event on day 3, per the original brief. A small enemy scout group
probes the camp. Resolution depends on whether a Soldier was on watch and how
much palisade is built. Kept deliberately separate from daily task resolvers
— it's an event, not a task outcome.

Outrider count is kept small and visible to the player — not a hidden number,
but a fact the log reports, so the player understands the scale of the
threat.

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
The palisade's real value is defensive capacity, not completion state.

**Resource risk / looting:** If the camp's defense is weak (no watcher, low
palisade), outriders loot on the way out. Priority order: weapons first
(worth the most, easiest to grab), then rations, then forage food, then raw
food. Wood is never looted — a raiding party isn't hauling lumber.

## Known Prototype Shortcuts (Not Yet Resolved)

- Combat currently checks the *shared* weapons pool as a stand-in for "is
  anyone armed at all," until weapons move to per-character ownership in a
  future session. Once that lands, individual armed state should gate
  individual risk instead.
