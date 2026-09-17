# Stealth + detection v1 playable slice

The first slice proves:

**approach threat → choose pace/equipment → create or avoid evidence → creature becomes suspicious/investigates → relocate or get identified → search/escape or fight → noise also feeds run pressure without omniscient aggro**.

## 1. In scope

- observer-specific detection/evidence;
- walk/run/Hustle movement distinction;
- Still Breath hold-still action hook;
- cloth/leather/mail/plate movement-noise profiles using fixture values;
- at least three authored surface profiles;
- unified NoiseEvent service;
- distance attenuation;
- one authored obstruction/topology proof;
- visual LOS + distance + movement contribution;
- dim/normal/bright authored visibility zones;
- one concealment volume/profile;
- one thrown distraction/noisemaker fixture;
- suspicion/investigation/alert/search/decay;
- Scavenger ordinary hearing/vision fixture;
- Drifter slower-reactivity fixture;
- Ghoul aggressive-reactivity fixture;
- Ash Sleeper special-rule integration;
- Edge Stalker boundary proof;
- Director aggregate pressure bridge;
- reconnect/replay safety;
- GM evidence/noise trace.

## 2. Crouch

Crouch is included only if the WotLK client/server bridge can represent a reliable posture/movement state without pretending an emote is authoritative collision/visibility.

If not, v1 ships **Walk + Still Breath** as the deliberate stealth controls and keeps crouch as a later capability.

## 3. First playable route

1. Enter fresh Rotwood run in light/quiet kit.
2. Walk near a Scavenger in dim conditions; show marginal evidence/suspicion rather than binary aggro.
3. Stop/break LOS; allow suspicion to decay.
4. Deliberately run over timber/debris; Scavenger investigates sound position.
5. Quietly relocate; Scavenger searches stale evidence and can fail.
6. Repeat in noisy armour to demonstrate stronger movement evidence without changing the underlying rules.
7. Throw a distraction to redirect an unaware/investigating creature.
8. Prove an engaged creature does not forget direct sight because of distraction.
9. Move from concealment into bright exposed space and show faster visual identification.
10. Walk/Still Breath past Ash Sleeper safely.
11. Run past the same rule and wake it.
12. Create loud combat; nearby appropriate creatures react through sound while Director also receives pressure.
13. Break LOS during combat and prove search/escape remains possible.
14. Verify Edge Stalker does not receive hidden coordinates from Director.
15. Relog/reconnect and prove no duplicate distraction/noise or guaranteed free awareness reset.

## 4. Out of scope

- perfect acoustic simulation;
- rendered-pixel brightness sampling;
- advanced shadow casting stealth;
- camouflage clothing system;
- scent/wind detection;
- prone movement;
- lock-on stealth takedowns;
- invisibility/magic stealth;
- exact player-facing detection meter;
- sophisticated squad search tactics;
- dynamic light destruction unless separately designed;
- permanent stealth skill/stat progression.

## 5. UX target

The player should learn state primarily from creature behaviour: head turn, listening pause, investigation movement, alert posture, vocal cue and search pattern.

A normal HUD should not reveal exact certainty, hearing radius or AI thresholds. If playtesting shows the behaviour is unreadable, add the smallest restrained cue needed rather than immediately exposing a numeric meter.

## 6. Balance questions for playtest

Tune rather than lock prematurely:

- how forgiving distant dim sight is;
- how much armour noise changes safe routing;
- how quickly weak suspicion decays;
- how long investigation/search remains tense without becoming tedious;
- how much running compromises an escape after LOS break;
- whether Still Breath needs Vigor cost/channel duration;
- how strong distraction attention should be;
- how much loud combat contributes to Director pressure;
- how differently Drifter/Scavenger/Ghoul should escalate.

## 7. Done criteria

The slice is done when `STEALTH-DETECTION-TEST-VECTORS.md` passes for the selected fixtures and a player can intentionally execute both outcomes:

- **successful stealth:** create little/false evidence, relocate, let search fail;
- **failed stealth:** accumulate enough legitimate evidence to be identified and engaged;

without any creature receiving hidden coordinates it did not earn.
