# Stealth + detection coding plan

Implement after/alongside the shared Creature AI foundation and before broad Rotwood encounter tuning.

## PR 1 — unified NoiseEvent authority

- stable noise registry;
- immutable/idempotent event IDs;
- spatial listener query;
- distance attenuation;
- AI auditory-evidence bridge;
- Director pressure bridge kept separate;
- GM trace.

Exit: sound creates evidence positions, never universal aggro.

## PR 2 — movement + Gear noise

- server walk/run/Hustle classification;
- bounded footstep cadence;
- Gear derived noise profile;
- surface profile hook;
- anti-packet-spam/idempotency;
- first cloth/plate fixtures.

Exit: movement/Gear/surface tests pass.

## PR 3 — visual evidence

- observer visual sampling;
- LOS/distance/facing hooks;
- evidence buildup/decay;
- last-seen memory;
- AI state transition bridge;
- no native aggro bypass.

Exit: close sight, marginal sight, LOS break/search tests pass.

## PR 4 — authored visibility + concealment

- visibility bands/zones;
- concealment profiles;
- world query service;
- movement/posture modifiers;
- client gamma/graphics irrelevance tests.

Exit: dim/bright/concealment cases pass.

## PR 5 — Still Breath + optional crouch

- Still Breath server action/state;
- interruption on movement/action;
- evidence/noise modifier;
- Survival Vigor hook only if approved;
- crouch only if reliable client/server representation is proven.

Exit: no invisibility toggle; Sleeper remains correct.

## PR 6 — distractions + social evidence

- thrown/noisemaker authoritative impact;
- `noise.distraction`;
- competing evidence selection;
- social stimulus data model;
- bounded information propagation;
- anti-ping-pong attention rules if needed from playtest.

Exit: distraction redirects investigation but not direct engagement.

## PR 7 — special integrations

- Ash Sleeper special adapter;
- Edge Stalker perception/pacing boundary;
- Scavenger/Drifter/Ghoul reviewed perception fixtures;
- Caller social escalation compatibility;
- Director pressure audit.

Exit: named-archetype tests pass without hard-coded generic-system contamination.

## PR 8 — reconnect/restart/performance hardening

- observer state reconciliation;
- event replay protection;
- staggered perception cadence;
- spatial performance profiling;
- run teardown;
- debug trace completeness.

Exit: no relog stealth exploit and acceptable Rotwood population cost.

## Before code

Resolve/verify:

1. exact AzerothCore movement flags for walk/run and how Hustle/Short Burst will be represented;
2. whether v1 crouch is technically honest or omitted;
3. canonical `noise.*`, visibility, concealment and perception-profile keys;
4. world surface tagging strategy;
5. authored visibility-zone representation;
6. Director pressure API;
7. exact Still Breath action semantics/Vigor policy;
8. which native aggro/threat helpers must be suppressed;
9. reviewed first hearing/vision profiles for Scavenger, Drifter and Ghoul;
10. restart persistence depth for short-lived suspicion/search.

Do not solve these by baking provisional values or creature-entry switch statements into generic Stealth code.
