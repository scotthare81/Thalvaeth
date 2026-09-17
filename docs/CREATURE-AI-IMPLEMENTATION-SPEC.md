# Creature AI implementation specification

This document turns the locked 90001–90010 creature roster into a server-authoritative behavioural system compatible with custom Combat, procedural run population, the Stress Director and the Creature Journal.

`CREATURES.md` remains the roster/content authority. This document defines shared AI semantics. It must not silently rewrite a creature's locked rule.

## 1. Design pillars

1. Creatures act on information they could plausibly possess.
2. Losing sight of the Remnant does not grant omniscient tracking.
3. Sound creates investigation and alert pressure, not universal radius aggro.
4. Every special keeps its one legible rule.
5. Fodder remains cheap enough for Director population use without becoming brain-dead MMO trash.
6. Combat actions use honest telegraph → commit → active → recovery semantics.
7. AI behaviour belongs to archetypes/profiles, never fixed spawn coordinates.
8. The authored map is static; run population is procedural; AI must work wherever a legal spawn node places it.
9. Run-local memory dies with the run.
10. The server owns perception, target knowledge, pathing intent, attacks and state. The addon is presentation only.

## 2. Information model

AI must distinguish **truth** from **knowledge**.

World truth may contain the player's exact transform. A creature's decision state contains only what its perception has established:

```text
PerceivedTarget
  targetGuid
  certainty
  lastSeenPosition
  lastSeenAt
  lastHeardPosition
  lastHeardAt
  identifiedDirectly
  currentlyVisible
  currentlyAudibleStimulus
```

A creature searching after LOS loss navigates/investigates remembered evidence. It must not continuously read the hidden live player position.

## 3. Canonical awareness states

Shared vocabulary:

`UNAWARE → SUSPICIOUS → INVESTIGATING → ALERTED → ENGAGED → SEARCHING → DISENGAGING`

Special archetypes may use only a subset or add a tightly scoped substate.

### UNAWARE
Normal idle/wander/post behaviour.

### SUSPICIOUS
Weak/ambiguous stimulus noticed. Creature may orient, pause or increase perception without knowing a target location precisely.

### INVESTIGATING
Creature has a meaningful stimulus position and moves/looks toward it without guaranteed target lock.

### ALERTED
Creature believes a threat is nearby and uses heightened perception/behaviour but may not have direct current sight.

### ENGAGED
Creature has sufficient target knowledge to execute combat pursuit/actions.

### SEARCHING
Direct target knowledge has been lost. Creature searches last-known evidence for a bounded period/pattern.

### DISENGAGING
Creature stops pursuit and returns/resumes authored behaviour while preserving run-persistent consequences such as HP where canon requires it.

## 4. Perception channels

V1 perception supports:

- vision;
- hearing/noise stimuli;
- direct damage/hit awareness;
- authored social/ally alert stimuli;
- special triggers such as Sleeper wake condition.

Do not create a generic magical proximity sense unless a creature's locked rule explicitly requires a proximity trigger.

## 5. Vision

Creature profiles define conceptual vision properties:

- normal range;
- peripheral/frontal weighting or cone where practical;
- close-range certainty;
- LOS requirement;
- alert-state modifier;
- movement/visibility modifier hooks for future Stealth.

V1 may use AzerothCore LOS/range primitives as sensors, but the AI layer owns the resulting knowledge state.

Vision should not instantly erase target knowledge when LOS flickers for one server tick. Use small authored grace/hysteresis to prevent jitter, without creating wallhacks.

## 6. Hearing

AI consumes semantic `NoiseEvent`s from Combat, Survival/actions and Director/world systems.

A hearing profile considers:

- noise category/intensity;
- distance/attenuation;
- current awareness state;
- creature archetype;
- optional obstruction/zone modifier where reliable;
- whether the creature is already focused on a stronger stimulus.

Noise yields evidence at the sound position. It does not automatically reveal the player's current location.

## 7. Investigation

An investigation has:

- stimulus position;
- source category if known;
- confidence;
- expiry;
- arrival behaviour;
- bounded search around the evidence location.

If the player has moved silently, the creature should be capable of reaching the old position and finding nothing.

Repeated/new stimuli can update the evidence position according to profile rules.

## 8. Target acquisition

A creature may become ENGAGED through:

- direct visual confirmation under its profile;
- being attacked/damaged and identifying the attacker;
- an authored special trigger;
- a strong social alert that provides sufficient target knowledge, where the archetype permits it.

Being merely inside a generic distance must not equal target acquisition unless that creature's locked rule says so.

## 9. LOS loss and search

On losing current visibility during engagement:

1. retain last seen position/time;
2. continue toward/act on that information for an authored short persistence window;
3. if reacquired, resume ENGAGED;
4. otherwise enter SEARCHING;
5. inspect last-known area using bounded search points/turning/listening;
6. react to new valid noise/visual evidence;
7. eventually disengage if evidence expires.

The search must not query the live hidden player transform to choose its next search point.

## 10. Search patterns

V1 search can be simple and deterministic enough to debug:

- move to last-seen/heard position;
- pause/orient/listen;
- visit a small number of nearby navigable offsets/nodes;
- abandon after bounded duration if no evidence.

Future AI may become richer, but do not require expensive general-purpose planning for v1.

## 11. Leashing and home behaviour

There is no universal MMO rubber-band leash rule.

Leash behaviour is archetype/content-specific:

- ordinary mobile fodder may disengage after losing evidence and resume local wander;
- a post/gatekeeper may have a strict authored territory;
- Patchwork Brute retains its locked 12y post leash and keeps its wounds;
- Edge Stalker uses its own flee/re-stalk behaviour and keeps its wounds;
- setpieces may use designer-authored territory constraints.

A true disengage must not silently full-heal creatures where `CREATURES.md` requires persistent run health.

## 12. Persistent run health

Creature damage belongs to the current run-world state. Disengagement, search failure, return-to-post and re-stalking do not automatically restore health.

Death/extraction run teardown discards that run's creature state. A fresh run gets fresh population/health.

This is particularly mandatory for Patchwork Brute and Edge Stalker, whose locked design explicitly preserves wounds across their disengagement behaviour.

## 13. Combat action selection

AI selects an **attack intent**, then the Combat system owns timing/hit resolution.

Selection may consider:

- target knowledge/certainty;
- current distance/angle;
- attack legal range;
- current recovery/cooldown;
- special one-shot state;
- current creature HP/state;
- target broad vulnerability/opportunity tags;
- nearby allies/space where relevant;
- authored priority/weight.

Do not select attacks purely by random spell roll every N seconds.

## 14. Honest attacks

Creature attacks should use the Combat contract:

`WINDUP/TELEGRAPH → COMMIT → ACTIVE GEOMETRY → RECOVERY`

If the Remnant leaves legal geometry before the active strike, the creature can miss.

If the creature is validly interrupted before commit, the attack can fail according to its definition.

Native stock melee may be used only where its timing/geometry honestly matches the authored action or as a controlled implementation carrier.

## 15. Opportunity behaviour

Some archetypes may react to broad player opportunity states such as heavy-weapon recovery. This must be authored and imperfect.

AI may receive semantic tags like `player.recovering_heavy`; it must not read hidden animation frames or exact future Combat timestamps beyond interfaces deliberately exposed for AI.

Fodder should not all become tactical geniuses.

## 16. Social awareness

Creatures do not share omniscient aggro tables.

Social stimuli can include:

- nearby pain/shout;
- ally death;
- Caller shriek;
- direct witnessed combat;
- explicit pack/group alert.

A recipient decides whether to ignore, investigate, alert or engage based on archetype and information content.

The Ash Caller is a special case: its locked shriek/wave rule is stronger than ordinary social awareness and remains explicit content behaviour.

## 17. Group behaviour

V1 group behaviour should remain lightweight:

- fodder can assist nearby combat when legitimately alerted;
- pairs may converge without sophisticated formation AI;
- creatures should not perfectly surround the player using shared global knowledge;
- doorway congestion/path failures need safe fallback rather than teleporting through each other;
- Director spawning and AI assistance are separate systems.

## 18. Director boundary

The Stress Director controls pressure/population/event decisions. Creature AI controls spawned creature behaviour.

Director may:

- choose/spawn eligible archetypes;
- react to noise/heat/depth;
- trigger authored encounter pressure;
- know run-level metrics unavailable to individual creatures.

Director must not make an individual creature omniscient by continuously feeding it the player's exact location.

Creature AI may publish events back to Director: engagement, loud fight, Caller event, creature death, search/alert pressure.

## 19. Procedural spawn compatibility

Every generated creature receives:

- archetype/profile key;
- run ID;
- generated spawn identity/node/zone context;
- authored territory/home anchor if required;
- initial AI state.

AI may use its generated home anchor/zone. It must not depend on a permanent DB spawn GUID or a hard-coded map coordinate.

Setpieces such as Ash Sleeper/Brute can still be constrained to authored eligible nodes; procedural does not mean unconstrained random placement.

## 20. Pathing

Movement uses server pathing/navmesh where available. AI must handle path failure explicitly:

- retry/alternate local point;
- abandon invalid investigation offset;
- return to last valid anchor;
- escalate debug diagnostics;
- never teleport to the player merely because pathing failed.

Setpiece return teleport, such as the currently locked Brute post behaviour, is a content-specific exception and must not become generic pursuit behaviour.

## 21. Idle behaviour

Idle movement supports atmosphere and encounter readability.

Fodder can wander according to locked roster behaviour. Specials preserve their distinct idle rules:

- Sleeper kneels/passive;
- Prowler waits above/passive;
- Caller waits to scream;
- Brute guards its post;
- Stalker paces at the edge;
- Snare holds choke pressure.

Generic idle code must not erase these silhouettes.

## 22. Damage awareness

A creature damaged by the Remnant receives authoritative attacker evidence even if its prior perception had not visually identified the player, subject to authored exceptions.

This prevents shooting/stabbing an unaware creature and having it remain oblivious. It does not necessarily give every ally the same attacker coordinates.

## 23. Stagger/bleed integration

AI consumes Combat-owned semantic state:

- light hit feedback;
- stagger pressure break;
- stagger protection;
- bleed change;
- death.

AI does not calculate bleed or stagger independently.

A stagger animation must not imply a gameplay interrupt unless Combat says the action was interrupted.

## 24. Flee/reposition

Fleeing is not equivalent to evade/reset.

Edge Stalker's locked break-off behaviour is the primary example: after being hit, it withdraws, wanders/repositions, then re-stalks while retaining HP.

Future cowardly creatures can use the same distinction. Never call a full-heal/reset helper just because the desired animation is "runs away."

## 25. Creature death

On authoritative death:

1. cancel future attack actions;
2. stop pursuit/search;
3. clear transient statuses through Combat;
4. emit death/social/noise event once;
5. notify Director/run state once;
6. expose loot/harvest through its own authority;
7. notify Journal observation/engagement hooks where valid;
8. persist run-world death state so reconnect/restart does not resurrect it accidentally.

## 26. Journal integration

AI/Combat can publish server-confirmed observations such as:

- creature sighted;
- creature engaged;
- distinctive action performed while observable;
- distinctive reaction/resistance witnessed.

Journal owns promotion to Sighted/Engaged/Observed knowledge.

Unknown abilities must not leak through addon payloads, internal keys, combat log names or tooltips before Journal rules permit them.

## 27. Run teardown

Terminal run teardown destroys run-local AI state:

- awareness;
- target memory;
- investigation/search positions;
- health/death state;
- combat actions;
- bleed/stagger;
- social alert state;
- temporary group relations;
- generated home anchors/spawn identities.

Persistent Journal knowledge survives.

## 28. Restart/reconnect

Disconnect does not reroll/reinitialize the run population.

For ACTIVE-run recovery, enough state must persist/reconstruct to prevent exploits such as:

- wounded Brute returning full-health;
- killed Caller respawning;
- Stalker wounds resetting;
- generated creature changing archetype/location due to reroll;
- alert/search state being abused as a guaranteed free reset if the design chooses to persist it.

Exact persistence depth for transient awareness/search can be bounded, but creature existence/death/health and generated population identity are authoritative run state.

## 29. Performance

AI must scale to Director-created populations:

- avoid full-player/full-creature world scans each tick;
- perception updates may use staggered cadences;
- noise should publish to spatially relevant consumers;
- search point generation is bounded;
- path recalculation is rate-limited;
- expensive debug traces are opt-in.

## 30. Debug tooling

Required GM/dev diagnostics:

- `.thal ai state <target>`
- `.thal ai perceive <target>`
- `.thal ai memory <target>`
- `.thal ai investigate <target>`
- `.thal ai search <target>`
- `.thal ai home <target>`
- `.thal ai trace <target> on|off`
- `.thal ai stimulus <type>` test injection where safe

Trace should show state transitions, evidence source, last-known positions, selected action, path failures, social events and disengage reason.

## 31. Authority invariants

The client cannot authoritatively provide:

- creature awareness state;
- last-known player position;
- investigation target;
- attack choice;
- path destination;
- social alert;
- search completion;
- creature HP reset;
- spawn/home anchor;
- Journal observation result.

## 32. Acceptance

Shared Creature AI is implementation-ready when tests prove:

1. LOS loss produces last-known-position search rather than wallhacking;
2. noise produces evidence/investigation rather than universal aggro;
3. attack selection respects authored geometry/recovery;
4. specials retain their one locked rule;
5. Brute/Stalker disengagement preserves wounds;
6. social awareness is local/information-based;
7. Director does not become individual-creature omniscience;
8. AI works from generated spawn/home context rather than fixed coordinates;
9. death/reconnect cannot resurrect/reset authoritative run creatures;
10. Journal observations come only from confirmed server events.