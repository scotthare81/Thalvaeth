# Stealth + detection implementation specification

This document defines the server-authoritative Stealth layer shared by player movement, Gear, Combat, Creature AI, the Stress Director and world interactions.

The goal is not a binary MMO stealth flag. Stealth is the player's management of **evidence**: what creatures can see, what they can hear, how certain they become, and whether the player can break or redirect that evidence before it becomes a confirmed engagement.

## 1. Core invariants

1. The player is never globally "stealthed" or "not stealthed" as an authoritative binary state.
2. Detection belongs to each observer. Two creatures may have different evidence/certainty about the same Remnant at the same moment.
3. Vision and hearing are separate evidence channels that can reinforce one another.
4. Sound reveals a sound position, not the player's live hidden coordinates.
5. Visual evidence requires legal server LOS and visibility conditions.
6. Losing LOS does not erase knowledge instantly; it converts current sight into last-known evidence.
7. Investigation is not engagement. A creature can hear something, inspect it and find nothing.
8. Armour, movement, actions, combat and environment modify evidence; they do not directly call generic aggro.
9. The Director may consume aggregate pressure/noise but cannot grant an individual creature omniscient target knowledge.
10. Native WotLK stealth/detection mechanics may be used only as implementation carriers if they do not violate these rules.

## 2. Detection model

Each observer tracks target evidence conceptually:

```text
DetectionRecord
  observerGuid
  targetGuid
  visualEvidence
  auditoryEvidence
  certainty
  lastSeenPosition
  lastSeenAt
  lastHeardPosition
  lastHeardAt
  currentVisibility
  identifiedDirectly
  awarenessState
```

Exact numeric certainty is server/debug data. Normal UI does not expose a Skyrim-style detection meter unless separately approved.

Detection is continuous enough to support buildup/decay, but authored thresholds produce the AI vocabulary already locked:

`UNAWARE → SUSPICIOUS → INVESTIGATING → ALERTED → ENGAGED → SEARCHING → DISENGAGING`

A threshold crossing is not enough by itself to invent information. ENGAGED still requires sufficient target identification/current evidence according to the observer profile.

## 3. Evidence and certainty

Evidence contributions are semantic and bounded. Conceptually:

```text
certainty += visualContribution
certainty += auditoryContribution
certainty += directHitContribution
certainty += validSocialContribution
certainty -= decayWhenEvidenceStops
```

Do not hard-code one universal rate. Creature perception profiles own sensitivity, threshold and decay characteristics.

Repeated weak evidence can make a creature increasingly suspicious. A single strong event can escalate quickly. Direct sight at close range can identify the Remnant rapidly. A faint distant sound should usually create investigation, not instant combat.

## 4. Movement states

Canonical movement/action categories for Stealth:

- `movement.still`
- `movement.walk`
- `movement.run`
- `movement.hustle`
- `movement.short_burst`
- `movement.crouch` if/when the client presentation supports a genuine crouched locomotion mode
- `movement.still_breath`

Movement state is server-derived from authoritative movement/action state. The addon cannot claim the player is walking quietly while the server sees a run.

### Still

Lowest ordinary movement noise and lowest movement visibility contribution. It is not invisibility.

### Walk

The default deliberate stealth locomotion. Walking should be materially quieter than running and must preserve the Ash Sleeper counterplay.

### Run

Creates stronger footfall/equipment noise and stronger visual motion evidence. Running near an Ash Sleeper satisfies its locked wake condition.

### Hustle

A sustained high-output movement state owned by Survival/Combat movement rules. It creates stronger noise and visual motion than ordinary walk and generally stronger pressure than run, while exact tuning remains data.

### Short Burst

Very strong brief movement evidence/noise. Useful to escape danger but poor for remaining unnoticed.

## 5. Crouch

Crouch is a **posture modifier**, not an invisibility toggle.

If implemented, crouch may:

- reduce silhouette/visual contribution where geometry/cover makes that meaningful;
- reduce footfall intensity at compatible speeds;
- reduce movement speed;
- make concealment behind low authored cover possible where the engine can support reliable LOS/height semantics.

Crouching in full view at close range does not prevent detection. Crouch must not become a universal percentage stealth buff independent of environment.

If reliable crouch collision/visibility cannot be implemented on the WotLK client, v1 may omit locomoting crouch rather than fake it badly.

## 6. Still Breath

Still Breath is a deliberate **hold-still stealth action**, not permanent crouch.

Intent:

- player stops moving;
- suppresses/reduces avoidable body/equipment noise;
- minimizes movement-derived visual evidence;
- can help let weak suspicion decay or allow a creature to pass;
- does not erase existing last-known evidence;
- does not make the player invisible in direct clear LOS;
- ends on incompatible movement/action/combat input.

Still Breath must preserve the Ash Sleeper fantasy: careful walking or stopping/holding still can pass it; running or being in combat nearby wakes it.

Exact Vigor cost, channel behaviour and whether Still Breath itself consumes Vigor remain balance/system decisions and are not locked here.

## 7. Unified sound model

All meaningful sound enters one semantic `NoiseEvent` pipeline regardless of source system.

```text
NoiseEvent
  eventId
  runId
  sourceGuid optional
  position
  category
  baseIntensity
  spectrum/material tags optional
  propagationProfile
  createdAt
  expiry
```

Sources include:

- footsteps/movement;
- armour/equipment rattle;
- weapon swings/impacts;
- creature attacks/calls/deaths;
- doors/containers;
- harvesting/crafting/tool use;
- breaking objects;
- thrown/distraction objects;
- environmental authored events;
- Survival actions where appropriate.

Combat's existing `CombatNoiseService` becomes a producer into this unified sound authority rather than a separate incompatible sound universe.

## 8. Sound categories

Stable semantic families should include at minimum:

- `noise.footstep`
- `noise.equipment`
- `noise.combat_light`
- `noise.combat_heavy`
- `noise.impact`
- `noise.creature_call`
- `noise.interaction`
- `noise.tool`
- `noise.distraction`
- `noise.environment`

Exact keys must be registry-validated before persisted/logged tooling depends on them.

Category matters because creatures can react differently to equal raw intensity. A Caller shriek, metal impact and footstep are not necessarily equivalent stimuli.

## 9. Noise intensity composition

A player-generated movement sound is derived from components rather than a single armour-class switch:

```text
movement base
× speed/state modifier
× footwear/surface modifier
× armour/equipment modifier
× action modifier
× environmental modifier
```

The result is a semantic intensity consumed by propagation/hearing profiles.

Do not expose exact radii as player-facing guarantees. "Heavy plate can be heard at 18.4 metres" is not the desired UX.

## 10. Armour and equipment noise

Gear publishes a derived noise profile. Stealth consumes it.

Intent:

- cloth/rags: very quiet baseline;
- leather: quiet;
- mail: noticeable movement/rattle;
- plate: loudest movement burden;
- packs/tools/spare gear can add authored equipment noise where useful;
- condition does not accidentally make heavy armour quieter unless explicitly authored.

Armour noise should matter most while moving. Standing still in plate is not equivalent to running in plate.

Equipment noise is a build tradeoff alongside Vigor/protection, not a punitive random proc.

## 11. Surfaces

World surfaces may modify footfall noise where reliable data exists:

- soft earth/ash;
- timber;
- stone;
- metal;
- shallow water/wet ground;
- debris/glass-like authored hazards.

Do not require perfect material detection for v1. Prefer authored area/surface tags where engine data is unreliable.

A loud authored debris patch can be a stealth obstacle and can generate a meaningful NoiseEvent when crossed.

## 12. Propagation

Sound is not simply `distance < radius`.

The runtime should support bounded attenuation and authored obstruction/zone modifiers where practical. At minimum:

1. source creates event;
2. spatial query finds plausible listeners;
3. distance attenuates stimulus;
4. hearing profile/category modifies significance;
5. optional world/portal/zone obstruction modifies it;
6. listener receives evidence at the sound position.

Do not raycast every sound against every creature every frame. Sound events are discrete and spatially queried.

Closed walls/major separation should reduce or block sound when reliable topology can represent it. Do not promise physically perfect acoustics.

## 13. Hearing outcomes

A heard sound can produce:

- ignore;
- orientation/listen;
- suspicion increase;
- investigation of source position;
- alert state;
- engagement only when the stimulus legitimately identifies the target strongly enough under that creature's profile.

The common stealth loop should therefore be possible:

**make noise → creature investigates → player relocates quietly → creature reaches old position → searches → gives up**.

## 14. Distraction sounds

Deliberate distractions use the same NoiseEvent pipeline. They are not scripted "move enemy here" commands.

A thrown object/noisemaker creates sound at its actual impact/activation position. Eligible creatures independently decide whether to react according to hearing profile/current evidence/state.

Rules:

- distraction never overwrites stronger direct sight;
- an ENGAGED creature does not forget the player merely because a pebble lands nearby;
- an INVESTIGATING/SUSPICIOUS creature can be redirected by stronger/new evidence;
- repeated identical distraction abuse may be limited through AI attention/confidence rules rather than arbitrary immunity;
- source identity should remain unknown unless the creature legitimately sees/infers it.

## 15. Investigation propagation

Creatures may create social stimuli from their reactions, but information content is bounded.

Examples:

- one Scavenger hears a crash and moves to inspect it; another nearby may notice its alert/pain/shout depending on profile;
- a creature that directly sees the Remnant may emit a meaningful alert stimulus;
- Ash Caller shriek is an explicit authored escalation event;
- an investigating creature does not psychically broadcast the player's exact position when it never saw them.

Propagation must carry **evidence quality**, not merely an aggro target GUID.

## 16. Vision model

Vision contribution considers:

- legal LOS;
- distance;
- observer vision profile;
- facing/peripheral relationship where practical;
- player movement state;
- player posture;
- authored concealment/cover;
- local visibility/light modifier;
- observer alert state.

No single factor grants invisibility except actual blocked LOS or explicitly authored concealment.

## 17. Light and visibility

Light modifies visual evidence; it does not replace LOS.

V1 should use **authored visibility zones/volumes/tags** rather than attempting to infer exact rendered pixel brightness from the client.

Possible semantic levels:

- `visibility.dark`
- `visibility.dim`
- `visibility.normal`
- `visibility.bright`
- `visibility.exposed`

Darkness slows/reduces visual evidence at suitable distances but does not make a player invisible at arm's length. Bright/exposed areas make movement easier to notice.

Portable/player light sources can increase local visibility and may themselves create a detectable visual cue where implemented.

## 18. Concealment

Concealment is authored world state that reduces visual evidence without necessarily blocking physical LOS.

Examples may include:

- dense brush;
- hanging cloth;
- heavy fog/smoke;
- deep shadow volumes;
- low cover when crouched if reliable.

Concealment has a semantic strength/profile. It does not suppress hearing.

A player crashing through brush or moving loudly behind a curtain may still be located through sound.

## 19. Visual detection buildup

At long/marginal conditions, sight should often build evidence rather than snap directly from invisible to combat.

Example behaviour:

- distant movement in dim cover → SUSPICIOUS;
- continued sight/movement → ALERTED;
- player stops/breaks LOS → evidence decays/search may begin;
- player walks into bright close clear view → rapid identification/ENGAGED.

This is observer-specific. Ghoul may escalate faster than Drifter. Stalker uses its special relationship rather than generic acquisition rules where canon says so.

## 20. Evidence decay

Evidence decays only when no confirming stimulus sustains it.

- weak suspicion can decay back to UNAWARE;
- investigation confidence expires;
- direct identification/engagement becomes SEARCHING after LOS loss rather than instant forgetting;
- repeated recent evidence may extend search/alert duration;
- run-local evidence never persists into a fresh run.

Decay must not become a visible countdown exploit unless deliberately exposed through creature behaviour.

## 21. Player feedback

Normal play should communicate detection diegetically:

- creature head/body orientation;
- pause/listen behaviour;
- vocalization;
- movement toward evidence;
- change in stance/pace;
- search behaviour;
- environmental audio feedback;
- optional restrained UI cue only if playtesting proves body-language insufficient.

Do **not** ship an exact numeric eye meter by default. GM/debug needs exact certainty and contribution traces.

## 22. Ash Sleeper integration

Ash Sleeper's locked trigger remains authoritative:

- within 10y + player running → wake;
- within 10y + player in combat → wake;
- walking does not satisfy the trigger;
- Still Breath/still does not satisfy the trigger;
- generic noise/detection must not silently replace this rule.

The unified sound system may still let Sleeper presentation react to authored ambient cues only if that does not wake it or invalidate the rule.

## 23. Edge Stalker integration

Edge Stalker remains a special pressure archetype rather than generic stealth acquisition.

Stealth affects whether/how confidently it maintains awareness where geometry permits, but its locked 22–26y pacing, hit-breakoff, withdrawal and re-stalk behaviour wins over ordinary ENGAGED chase logic.

The Stalker must not receive player coordinates from Director merely to maintain perfect pacing through walls. Its repositioning must use legal perception/pathing and authored pressure logic.

Exact interaction between deep concealment and Stalker tracking should be playtested and locked with its perception profile rather than assumed globally.

## 24. Stress Director integration

The Director may consume **aggregate semantic events** such as:

- loud movement/combat;
- repeated disturbances;
- confirmed engagements;
- creature calls;
- prolonged high-pressure actions;
- district/depth context.

The Director may respond by adjusting future pressure/spawn/event choices.

It cannot:

- mark all creatures as ENGAGED;
- provide exact hidden player coordinates to individual AI;
- override a creature's special perception rule;
- turn a distraction into omniscient global knowledge.

Noise can therefore have two consequences simultaneously: a nearby creature may hear/investigate it, and the Director may record broader run pressure.

## 25. Combat integration

Combat produces unified noise events for:

- swings where authored;
- impacts;
- heavy misses hitting world if detectable;
- creature/player pain/death where appropriate;
- heavy cleave/stomp/call events.

Combat state also affects stealth directly: being in combat satisfies the Ash Sleeper wake condition and generally makes stealth recovery harder through ongoing movement/noise/evidence.

A creature losing LOS during combat can still transition to SEARCHING. "In combat" must not mean permanent wallhack target lock.

## 26. Survival integration

Survival owns Vigor and movement-state costs. Stealth reads movement/action state and emits evidence/noise consequences.

Hunger/Thirst/Infection do not automatically change stealth unless an explicit authored consequence exists. Do not invent random coughs, groans or noisy penalties merely because a survival meter is high.

## 27. Gear integration

Gear exposes derived stealth-relevant properties:

- movement/equipment noise profile;
- footwear/surface modifier hooks;
- armour silhouette/visibility hooks only where explicitly authored;
- carried/worn noisy equipment tags;
- light-source state if Gear owns the object.

Stealth never parses item names or WotLK armour class directly.

## 28. Run lifecycle

Stealth/detection state is run-local:

- observer evidence;
- suspicion/certainty;
- investigation positions;
- sound-event history;
- distraction state;
- concealment temporary state;
- Director pressure derived from current run.

Death/extraction teardown clears it. Reconnect to the same ACTIVE run must not reroll creature population or create an exploitable guaranteed full-awareness reset.

Exact persistence of short-lived suspicion/search across server restart may be bounded, but confirmed creature existence/health and major run state remain authoritative under Run Lifecycle.

## 29. Anti-exploit rules

Reject designs that allow:

- crouch toggling to erase detection instantly;
- walking/running packet spoofing to suppress sound;
- relogging to clear an engaged creature for free;
- throwing a distraction to force an engaged creature to forget direct sight;
- armour swap while prohibited to retroactively alter an emitted sound;
- duplicate network/action replay to emit duplicate distraction/noise events;
- client-reported light level or concealment;
- Director heat leaking exact spawn/target information to addon.

## 30. Debug tooling

Required development diagnostics:

- `.thal stealth state`
- `.thal stealth observers`
- `.thal stealth trace <observer> on|off`
- `.thal stealth visibility`
- `.thal stealth concealment`
- `.thal noise emit <category> <intensity>`
- `.thal noise trace on|off`
- `.thal noise listeners <event>`
- `.thal ai evidence <target>`

Trace should explain contributions, for example:

```text
observer=C90002 target=P1
vision: LOS=yes distance=14m visibility=dim movement=walk contribution=low
hearing: event=N104 footstep attenuated=weak
certainty: suspicious -> investigating
lastHeard=(...)
action=investigate sound position
```

## 31. Acceptance criteria

Stealth is implementation-ready when deterministic tests prove:

1. walk/run/Hustle create distinct evidence/noise without client authority;
2. armour modifies movement noise without becoming a binary stealth stat;
3. sound produces positional evidence rather than hidden target lock;
4. LOS loss enables real escape/search;
5. visual detection can build/decay in marginal conditions;
6. close clear sight identifies rapidly;
7. light/concealment modify sight without overriding LOS/hearing;
8. distraction can redirect investigation but not erase direct engagement;
9. social propagation cannot upgrade information beyond what was actually communicated;
10. Sleeper's locked wake rule survives the generic system;
11. Stalker remains its own pressure archetype;
12. Director consumes pressure without granting omniscience;
13. death/reconnect/restart cannot cheaply reset or duplicate evidence events;
14. normal clients receive no exact hidden certainty/radii/profile values;
15. debug traces can explain every meaningful detection transition.