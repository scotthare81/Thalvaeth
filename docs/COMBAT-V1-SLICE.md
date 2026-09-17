# Combat v1 playable slice

The first Combat slice proves:

**see threat → choose whether to fight → commit an attack → spend Vigor → hit or whiff honestly → create damage/stagger/bleed/noise → creature reacts → recover/disengage → weapon wears → run consequences continue**.

## 1. In scope

Player weapons:

- Crude Dagger/Knife fast baseline;
- one paired fast weapon fixture;
- one Greatsword or heavy-blade fixture;
- one Greataxe fixture if needed to prove chop/stagger distinction;
- Fine/Worn/Damaged/Broken profiles.

Player actions:

- quick attack;
- committed attack;
- one paired multi-strike action;
- one heavy cleave;
- one heavy narrow/high-stagger attack.

Systems:

- action state machine;
- Vigor commit;
- server reach/facing;
- damage tags;
- stagger pressure/protection;
- bounded bleed;
- target-capped cleave;
- semantic noise;
- creature reaction bridge;
- phase-aware interruption;
- recovery locks;
- durability wear;
- death/reconnect/run teardown safety;
- trace/debug tooling.

## 2. Creature fixtures

Use a deliberately small subset of Rotwood archetypes already in scope for Journal/content work. At minimum prove:

- one ordinary bleed-susceptible creature;
- one fast/mobile creature;
- one high-stagger-resistance/brute creature;
- Ash Caller or equivalent interruptible special-action creature;
- one bleed-resistant/immune fixture if a suitable existing archetype fits canon.

Do not invent permanent anatomy traits for named creatures merely to satisfy the test. If current creature canon is insufficient, use test-only profiles until reviewed.

## 3. First combat loop

1. Enter fresh randomized Rotwood run with persistent weapon.
2. Approach ordinary creature.
3. Quick attack demonstrates short commitment and Vigor cost.
4. Deliberately whiff; cost/recovery still apply.
5. Apply bleed to eligible creature; observe bounded status.
6. Fight second creature with heavy fixture.
7. Heavy attack demonstrates larger commitment/stagger/noise.
8. Deliberately whiff heavy attack and experience recovery exposure.
9. Use cleave against two valid targets; third/out-of-arc target is safe.
10. Trigger nearby AI response to combat noise without universal instant aggro.
11. Interrupt one creature special through valid stagger condition.
12. Drive weapon through condition bands to Broken.
13. Confirm Broken degraded combat without item deletion.
14. Die/reconnect/run-reset cases prove no action/status replay and gear condition persistence.

## 4. Out of scope

- shields/block/parry system;
- ranged weapons;
- firearms;
- spellcasting/magic combat;
- elaborate combo trees;
- lock-on targeting;
- player dodge-roll unless separately designed;
- locational limb damage;
- executions/finishers;
- PvP;
- large boss movesets;
- complex critical-hit table;
- elemental damage matrix;
- dozens of status effects;
- perfect weapon/world collision simulation.

## 5. Balance approach

Use fixture values first. Tune relationships rather than final numbers:

- fast cost < heavy cost per action;
- fast recovery < heavy recovery;
- heavy stagger > fast stagger;
- heavy noise > fast noise;
- heavy cleave has meaningful target cap/commitment;
- fast bleed pressure > heavy baseline bleed where anatomy allows;
- Broken profile is clearly worse without becoming unusably bugged;
- Vigor never becomes irrelevant in sustained combat.

Do not publish exact numbers as permanent canon until playtesting.

## 6. UX requirements

The player should be able to understand combat without MMO combat-text dependence.

Need presentation hooks for:

- attack commitment/recovery through animation;
- low Vigor through existing Survival presentation;
- hit impact;
- meaningful stagger break;
- bleed/wound where appropriate;
- Broken weapon degradation;
- creature investigation/alert from noise.

Debug overlays may show exact geometry/meters; normal play should not.

## 7. Integration gates

Combat implementation cannot proceed safely until it uses:

- Gear effective weapon/condition profile;
- Equipment logical weapon identity;
- Survival Vigor authority;
- Run ID/active-state authority;
- creature archetype/profile data independent of spawn coordinates;
- Director/AI semantic noise consumer or temporary trace consumer;
- Journal observation hooks only through server-confirmed events.

## 8. Done criteria

The slice is complete when all relevant `COMBAT-TEST-VECTORS.md` cases pass and playtesting demonstrates a visibly different fast/heavy decision rhythm without relying on arbitrary item-level scaling.