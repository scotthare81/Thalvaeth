# V1 integration test vectors

These tests prove that individually documented systems form one game.

## A — complete successful run
Prepare legal loadout in Monastery; enter Rotwood; generation manifest created; Survival initialized; gather raw material into satchel; create noise and evade investigation; fight one creature; weapon wears/Vigor spent; discover valid Journal knowledge; extract.
Expected: exactly one EXTRACTED terminal outcome, haul promoted once, gear exact condition preserved, run-local AI/Director/Survival state destroyed, player arrives at home_extraction_arrival.

## B — complete death run
Enter, gather haul, damage persistent gear, die during active threat.
Expected: attack authority stops; visible bags may drop as presentation; exactly one DEAD outcome; run haul lost according to Inventory contract; persistent worn gear survives with exact damage; population/run/Director state destroyed; player wakes at home_recovery_spawn.

## C — death/extraction race
Lethal event and extraction completion contend.
Expected: first valid terminal guard wins; never both; no partial promotion/duplication.

## D — reconnect
Disconnect during ACTIVE run after generation, creature damage, suspicion and gathered haul.
Expected: same run ID/seed/population/mutations; no reroll, free repair, free extraction, duplicate noise/discovery or guaranteed AI reset.

## E — server restart
Persist a representative active run.
Expected: generation manifest and meaningful mutations reconstruct consistently; no killed creature respawn through seed replay; no duplicate Caller/Director additions.

## F — two-run variability
Complete/terminate run A, enter run B.
Expected: authored geography identical; new run ID/seed; legal population can differ; distinctive creature is not guaranteed at previous coordinate.

## G — stealth to combat
Walk quietly through dim/concealed route, create a distraction, relocate, then later get seen and fight.
Expected: evidence transitions are observer-specific; distraction never grants omniscience; Combat noise feeds nearby hearing and Director separately.

## H — heavy build consequence
Equip heavy weapon/armour; move/fight.
Expected: Gear-derived noise/Vigor/combat profiles create meaningful costs; Director may record pressure; no generic “heavy = better” score.

## I — crafting economy
Extract material needed by both repair and upgrade/barter. Spend it on one.
Expected: atomic stock deduction; competing sink remains unavailable if resources insufficient; no hidden currency conversion.

## J — Journal secrecy
Unknown creature/recipe/diagram exists in server definitions.
Expected: normal client receives no hidden name/effect/key leakage. Legitimate observation/discovery advances state once.

## K — home safety
Manipulate home stock/equipment, repair, prepare.
Expected: no run Survival/Director pressure; death recovery is not free repair; pack capacity changes preflight inventory.

## L — native WoW bypass audit
Attempt stock autoattack, threat/aggro, release spirit, hearth/unstuck, native durability and stealth paths.
Expected: each is explicitly suppressed, repurposed or proven compatible; none bypasses Thal'vaeth authority.

## M — addon failure
Disable/reload addon during run.
Expected: server game state continues; no extraction/death block; resync restores safe known UI state without hidden leakage.

## N — idempotency storm
Replay duplicate action/discovery/noise/extraction/crafting requests.
Expected: each semantic action commits at most once.

## O — first-hour comprehension
Fresh tester receives no developer coaching.
Expected after 60–90 minutes: can explain walking/noise, death risk, persistence, extraction, material progress and why next run differs.

## Definition of V1 integration pass
A build passes only when A–O have trace evidence and no subsystem depends on a contradictory deprecated contract.