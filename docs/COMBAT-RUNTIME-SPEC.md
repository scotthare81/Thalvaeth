# Combat runtime specification

This document maps the Combat contract into server runtime responsibilities. `COMBAT-IMPLEMENTATION-SPEC.md` owns behaviour; this file owns separation of concerns and transaction/event flow.

## 1. Services

Recommended module boundaries:

- `CombatDefinitionRegistry` — immutable attack/reaction/status definitions;
- `CombatActionService` — creates and advances player/creature actions;
- `CombatSpatialResolver` — server-side reach/arc/candidate resolution;
- `CombatDamageResolver` — derives authoritative damage from Gear + attack + target;
- `StaggerService` — transient poise pressure/protection;
- `BleedService` — bounded bleed state/ticking;
- `CombatNoiseService` — converts actions/impacts into semantic sound stimuli;
- `CombatReactionBridge` — publishes combat stimuli to creature AI;
- Gear `DurabilityService` — receives semantic wear events;
- Survival service — owns Vigor and validates/commits action cost;
- Director/AI systems — consume noise/heat/reaction events; Combat does not directly spawn reinforcements.

## 2. Action identity

Every accepted attack receives a unique server-owned `combat_action_id` scoped safely enough for replay detection.

A logical action may contain multiple authored strike IDs:

```text
CombatAction
  actionId
  actorGuid
  attackKey
  gearInstanceId
  runId
  state
  acceptedAt
  commitAt
  activeWindows[]
  recoveryEnd
  vigorCommitted
  resolvedStrikeIds[]
```

Paired weapon flurries use one action ID with multiple strike IDs, never independent main/off-hand auto-attacks.

## 3. Time authority

Server monotonic/runtime time determines action phases. Client animation timestamps are presentation hints only.

On request:

1. validate actor/run/alive/control state;
2. validate weapon/attack compatibility;
3. validate no incompatible active action;
4. validate preliminary Vigor availability;
5. create action with server timestamps;
6. broadcast/present start;
7. at commit, revalidate mandatory state and atomically spend Vigor;
8. advance active strike windows;
9. resolve each strike once;
10. enter recovery;
11. return READY when server recovery ends.

## 4. Vigor integration

Combat does not maintain a shadow stamina meter. It calls Survival authority.

Conceptual interface:

```text
CanCommitCombatCost(player, actionId, amount)
CommitCombatCost(player, actionId, amount)
```

The Survival side must make `CommitCombatCost` idempotent by action ID. Combat stores whether commitment succeeded.

If commitment fails because Vigor changed between request and commit, the action invalidates before damaging active windows.

## 5. Spatial resolver

Input:

- actor transform;
- attack geometry;
- current server world objects;
- active strike timestamp;
- optional primary intent/aim direction.

Output:

- deterministic ordered valid targets;
- per-target distance/angle/contact context;
- rejection diagnostics.

Do not accept client-provided cleave lists.

The resolver should support at minimum:

- frontal cone/arc;
- narrow thrust/cut arc;
- radial distance bound;
- target cap;
- deterministic ordering;
- line-of-sight/world collision hook where engine support is trustworthy.

## 6. Damage resolver

Conceptual pipeline:

```text
weapon = Gear.GetEffectiveWeaponProfile(instance)
attack = CombatRegistry.GetAttack(attackKey)
target = GetCombatDefenseProfile(creature/player)
result = Resolve(weapon, attack, target, context)
```

`result` includes authoritative damage and semantic tags used by stagger/bleed/reaction. Secondary services cannot invent a successful hit if damage resolution rejected contact.

## 7. Randomness

Where hit/quality/crit-like variance exists, randomness must be server-owned and bounded. V1 should minimize opaque combat RNG.

Prefer deterministic authored outcomes plus small controlled variance only where it improves feel. Do not reproduce stock MMO miss/dodge/crit tables simply because AzerothCore provides them.

If RNG is used, action/strike identity should make debugging/replay analysis possible. Do not allow reconnect or repeated packets to reroll the same strike.

## 8. Stagger runtime

Transient per-creature state:

```text
StaggerState
  targetGuid
  pressure
  lastPressureAt
  protectionUntil
  generation
```

`ApplyPressure` validates the target profile, protection state and hit tags. Crossing threshold emits one threshold-break event, resets/reduces pressure according to definition and applies protection.

Decay runs on a lightweight cadence/event-time calculation rather than requiring expensive per-frame updates for every creature.

## 9. Bleed runtime

Transient run-owned status:

```text
BleedState
  targetGuid
  familyKey
  intensity
  appliedAt
  expiresAt
  nextTickAt
  sourceContext
```

Bleed is run-local. Instance teardown removes it. Reconnect does not create a second state.

Tick processing is server-owned and cannot continue against dead/despawned targets.

## 10. Noise runtime

Combat publishes `NoiseEvent` records:

```text
NoiseEvent
  eventId
  runId
  sourceGuid
  position
  category
  intensity
  effectiveRadius/profile
  timestamp
```

Consumers decide reaction. Combat does not call `AggroAllWithin(radius)`.

Duplicate action replay must not create duplicate meaningful noise events.

## 11. Creature reaction bridge

Publish semantic events such as:

```text
OnCombatHit(target, source, tags, severity)
OnStaggerBreak(target, source)
OnBleedChanged(target, state)
OnNoise(event)
OnCreatureDeath(target, source, context)
```

Creature scripts/archetypes map these to behaviour. Generic Combat code must not contain `if creatureEntry == 90004 then ...` except in dedicated content data/adapters.

## 12. Interruption runtime

`InterruptAction(actor, source, strength/tags)` resolves against current action phase and definition.

Before commit:
- terminate if qualifying;
- no committed Vigor charge.

After commit:
- never refund Vigor;
- terminate/shorten active behaviour only if definition permits;
- enter authored interrupted recovery.

Duplicate interrupt events are idempotent against an already terminal action phase.

## 13. Recovery locks

A server-owned action lock prevents prohibited operations until recovery ends:

- new incompatible attack;
- Hustle/sprint where prohibited;
- weapon swap;
- crafting/survival channel;
- extraction interaction.

Do not rely on addon button greying to enforce locks.

## 14. Creature attack integration

Creature attack definitions may use the same registry/state model with a simpler AI request layer. AI selects an attack intent; Combat validates range/state and runs the authored telegraph/active/recovery timeline.

This makes creature telegraphs honest: if a claw sweep visually has a wind-up and frontal arc, damage occurs through that same definition rather than a stock instant melee swing underneath it.

## 15. Death races

Player death vs pending attack:

- Run/death authority marks player no longer eligible to commit new actions;
- uncommitted action cancels;
- committed but unresolved future strikes are cancelled in v1 once death terminal transition wins;
- no Vigor refund after commit.

Creature death vs pending status:

- mark dead once;
- later strike/status callbacks see dead and no-op;
- death reaction/noise/Journal/loot hooks emit once.

## 16. Disconnect

Disconnect is not combat resolution.

Server-owned action already committed may continue/cancel according to deterministic runtime policy, but reconnect cannot replay it. For v1, if the character remains in world during disconnect, ordinary server simulation remains authoritative. If core behaviour removes/freezes the character, reconcile action state without refunding a committed cost or duplicating a strike.

Exact disconnect-avatar behaviour should align with Run Lifecycle implementation.

## 17. Performance

Avoid full-world scans per swing. Candidate queries should use engine spatial partitions/range visitors and then apply custom geometry filters.

Stagger/bleed decay should use event-time arithmetic or bounded scheduled updates.

Trace logging is opt-in/debug; production combat must not log every candidate target at normal verbosity.

## 18. Definition validation

At startup, reject/log definitions with:

- negative timing/cost/reach;
- commit after active window;
- recovery ending before active window;
- zero/invalid target cap;
- unknown weapon capability;
- unknown damage/status/noise profile;
- bleed with invalid cap/duration;
- cleave geometry without deterministic target policy;
- follow-up cycles that bypass recovery unless explicitly authored.

## 19. Native AzerothCore bridge

Stock melee/spell systems may be used as carriers where useful, but they cannot silently reintroduce:

- uncontrolled auto-attacks;
- stock hit/miss tables contrary to custom resolution;
- native rage/energy as duplicate Vigor;
- client-authoritative spell target lists for cleave;
- automatic durability loss that double-stacks with Gear wear;
- native proc systems that bypass attack definitions.

Every reused core path must be audited for these side effects.

## 20. Debug trace example

A useful trace should read conceptually:

```text
A1042 attack.greatsword.cleave accepted gear=G17 vigor=6200
A1042 commit t=... cost=950 vigor_after=5250
A1042 strike=0 candidates=4 valid=2 cap=2
  C90008 hit damage=... stagger=... bleed=no
  C90009 rejected angle
  C90010 hit damage=... stagger=...
A1042 noise combat.heavy_impact intensity=...
A1042 wear G17 heavy_impact ...
A1042 recovery until ...
```

Exact hidden balance values are GM/debug only.

## 21. Runtime acceptance

The runtime is safe to expand when action IDs, server timing, Vigor idempotency, spatial resolution, status services, noise publishing, reaction bridge, recovery locks, death/disconnect reconciliation and Gear wear all pass deterministic test vectors without relying on addon authority.