# Stealth + sound runtime specification

This document maps the Stealth/Detection contract into runtime services and event flow.

## 1. Runtime services

Recommended boundaries:

- `StealthPerceptionService` — observer-specific visual/auditory evidence and certainty;
- `VisibilityService` — authored light/visibility/concealment queries;
- `NoiseService` — unified semantic sound-event authority;
- `NoisePropagationService` — spatial listener query/attenuation/obstruction;
- `MovementNoiseProducer` — player/creature locomotion events;
- `InteractionNoiseProducer` — doors/tools/harvest/world interactions;
- Combat noise producer — feeds unified `NoiseService`;
- Gear derived profile provider — noise/equipment modifiers;
- Creature AI — consumes evidence/state transitions;
- Director — consumes aggregate pressure events, never observer knowledge.

## 2. Unified NoiseEvent

```text
NoiseEvent
  eventId
  runId
  sourceGuid
  position
  categoryKey
  intensity
  propagationKey
  sourceActionId optional
  emittedAt
```

`eventId` is unique/idempotent. An action that has already emitted its semantic sound cannot emit it again through packet replay/reconnect.

## 3. Movement sound cadence

Do not create one sound event every server movement packet. Derive footsteps/locomotion at bounded cadence from server movement state and authored movement profile.

Movement producer considers current movement category, Gear noise profile, surface tag and relevant world modifiers. It snapshots those values when the event is emitted.

## 4. Propagation flow

1. validate run/source/event;
2. create immutable NoiseEvent;
3. spatially query plausible listeners;
4. attenuate by distance/profile;
5. apply authored topology/obstruction modifier where available;
6. apply listener hearing profile/category sensitivity;
7. discard below meaningful threshold;
8. submit auditory evidence to `StealthPerceptionService`;
9. publish aggregate Director pressure event separately where category qualifies.

The listener receives event position and strength, not target live transform.

## 5. Visual sampling

Visual perception can update on staggered bounded cadence rather than every frame for every creature.

For each plausible observer/target pair:

1. broad distance/spatial cull;
2. observer state/profile range;
3. server LOS;
4. facing/peripheral modifier if enabled;
5. VisibilityService query at target/context;
6. movement/posture modifier;
7. concealment modifier;
8. calculate evidence contribution over elapsed time;
9. update observer DetectionRecord;
10. notify AI only on meaningful state/evidence transitions.

## 6. VisibilityService

World designers author semantic zones/volumes/area tags rather than relying on rendered pixels.

Conceptual result:

```text
VisibilityContext
  lightBand
  concealmentKey
  concealmentStrength
  exposedModifier
  sourceLightTags
```

Server position decides membership. Client gamma/graphics settings are irrelevant.

## 7. DetectionRecord lifecycle

A record exists only while useful and run-valid. It stores observer-owned evidence, timestamps and awareness state.

Visual and auditory evidence decay independently enough to preserve last-seen vs last-heard semantics. AI receives meaningful transitions and evidence snapshots, not raw world truth.

## 8. State transition ownership

`StealthPerceptionService` evaluates evidence thresholds/profile and proposes/updates perception state; Creature AI owns behavioural response.

This avoids two systems independently deciding aggro.

Special archetypes can override generic transition policy through authored profile hooks, but not by bypassing evidence security.

## 9. Distraction action

A thrown/noisemaker action has its own server action ID. On authoritative impact/activation, emit one `noise.distraction` event.

The client may request aim/use, but server world position/impact determines sound location. Client cannot submit an arbitrary lure coordinate.

## 10. Social evidence

AI may publish `SocialStimulus` records:

```text
SocialStimulus
  eventId
  sourceCreature
  position
  type
  targetIdentityKnown
  targetGuid optional
  confidence
  createdAt
```

A social event cannot contain a target GUID/position the source creature did not legitimately know. Recipients apply their own profile.

## 11. Director pressure bridge

Noise/engagement systems publish separate aggregate records such as:

```text
DirectorPressureEvent
  eventId
  runId
  position/zone
  category
  magnitude
  createdAt
```

Director can use these for heat/stress/pacing. This record is not fed back as direct creature target evidence.

## 12. Ash Sleeper adapter

Sleeper checks authoritative player movement/combat state directly for its locked special trigger while dormant. Generic Stealth evidence cannot wake it unless `CREATURES.md` is deliberately changed later.

This adapter prevents a future NoiseService tuning change from accidentally destroying the encounter rule.

## 13. Stalker adapter

Stalker uses its dedicated pacing state machine and a reviewed perception profile. It may consume legitimate last-seen/last-heard evidence but does not receive Director truth coordinates.

When legal evidence is insufficient, reposition/search behaviour must degrade gracefully rather than teleport or maintain mathematically perfect range through obstacles.

## 14. Data registries

Need validated registries for:

- noise categories;
- propagation profiles;
- movement noise profiles;
- surface profiles;
- visibility bands;
- concealment profiles;
- creature hearing profiles;
- creature vision profiles;
- detection threshold/decay profiles.

Stable keys are mandatory. Exact numeric tuning remains data.

## 15. Performance

- spatially cull listeners before expensive work;
- event-driven sound, not continuous radius polling;
- stagger visual perception cadence across creatures;
- cache authored visibility zone lookup where safe;
- bounded search/investigation updates;
- no per-frame acoustic raytrace mesh simulation;
- debug traces opt-in.

## 16. Native AzerothCore audit

Explicitly audit/suppress/reuse:

- native stealth/invisibility aura detection;
- creature aggro radius;
- assistance/call-for-help helpers;
- threat list persistence through LOS;
- evade/reset full-heal behaviour;
- movement flags used to distinguish walk/run;
- LOS/path primitives;
- area/zone/liquid/terrain queries;
- client crouch/emote limitations.

Never leave stock aggro active underneath custom perception in a way that bypasses evidence.

## 17. Restart/reconnect

Noise events are ephemeral but action/event IDs prevent replay. Active-run recovery preserves enough observer/run state to prevent relog-as-stealth-button exploits.

Short weak suspicion may be reconstructed/expired according to bounded policy after a true server restart, but confirmed run creature identity/health and significant run state remain authoritative. Document any restart approximation explicitly.

## 18. Debug trace

Example:

```text
N884 noise.footstep source=P1 pos=(...) intensity=34 surface=timber movement=walk gear=mail
 listener C90002 distance=11 attenuated=17 hearing=scavenger.standard accepted
 observer C90002 auditory +12 lastHeard=(...)
 certainty 18 -> 30 state SUSPICIOUS -> INVESTIGATING
 DirectorPressure event=low_movement zone=rotwood.cellar
```

A visual trace should similarly show LOS, distance, visibility band, movement/posture and concealment contribution.

## 19. Runtime acceptance

Runtime is safe to code when:

- one immutable semantic sound event can serve AI and Director without sharing authority;
- movement packets cannot spam sound;
- client cannot choose sound impact/listener/detection result;
- visual evidence is server-world based;
- observer records preserve last-seen/last-heard distinction;
- stock AzerothCore aggro cannot bypass custom perception;
- Sleeper/Stalker special adapters preserve canon;
- performance scales to procedurally populated Rotwood runs.
