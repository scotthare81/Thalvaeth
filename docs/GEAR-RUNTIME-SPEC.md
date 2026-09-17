# Gear runtime specification

This document defines runtime boundaries for gear, weapons, durability, repair and upgrading. `GEAR-IMPLEMENTATION-SPEC.md` owns the authoritative rules; this file describes how code should be separated so those rules remain testable.

## 1. Runtime ownership

Recommended server components:

- `GearDefinitionRegistry` — immutable authored gear/upgrade/material profiles;
- `GearInstanceService` — persistent per-character gear instance state;
- `DurabilityService` — centralized wear, condition derivation and repair mutations;
- `GearUpgradeService` — validates and commits graph transitions;
- `GearEffectResolver` — derives combat/survival/tool effects from instance + definition + condition;
- Inventory reservation service — shared dependency, not duplicated inside Gear;
- Journal discovery service — shared authority for diagram/process knowledge.

Do not scatter authoritative durability writes through creature scripts, spell scripts and addon handlers.

## 2. Conceptual records

```text
GearInstance
  instanceId
  ownerGuid
  archetypeKey
  nodeKey
  materialKey
  qualityValue
  durabilityCurrent
  durabilityMax
  schemaVersion

UpgradeDefinition
  key
  sourceNodes[]
  targetNode
  stationKey
  requiredKnowledge[]
  requiredDiagram?
  ingredients[]
  resultProfile

WearEvent
  eventId
  gearInstanceId
  wearType
  magnitude
  sourceContext
```

Names are illustrative; stable semantics matter more than exact C++ type names.

## 3. Definition registry

Definitions load once from code/data/DB according to module conventions. Startup validation must reject or loudly log:

- duplicate stable keys;
- missing source/target nodes;
- upgrade cycles unless explicitly supported;
- missing station keys;
- unknown material/knowledge keys;
- invalid durability values;
- impossible slot/form transitions;
- graph nodes with no reachable source unless declared roots.

## 4. Derived effects

Gameplay systems ask Gear for effective values rather than reading raw item-template stats as final authority.

Conceptual API:

```text
GetWeaponProfile(player, slot)
GetArmourProfile(player)
GetToolCapability(player, capability)
GetNoiseModifier(player)
GetVigorModifier(player, action)
GetEffectiveDurability(instance)
GetConditionBand(instance)
```

The resolver applies authored base/node/material/quality/condition modifiers in a deterministic order.

Suggested order:

`definition base → upgrade node → material variant → hidden quality → current condition → temporary run effects`

Temporary poisons/buffs do not rewrite persistent gear data.

## 5. Wear pipeline

Callers submit semantic wear events; they do not directly decrement durability.

```text
ApplyWear(instanceId, wearType, magnitude, eventId)
```

The service:

1. validates instance/ownership/context;
2. rejects duplicate event IDs where the same gameplay event could replay;
3. resolves wear profile;
4. clamps at zero;
5. persists new durability;
6. derives old/new condition band;
7. emits a state delta only if useful;
8. logs significant transitions (especially Broken).

## 6. Combat integration

Weapon attack resolution requests the effective weapon profile before damage/Vigor/noise hooks. A successful attack can then emit one wear event using the same combat event identity.

Armour damage processing emits wear after authoritative incoming damage/mitigation is known. Avoid one wear call per internal damage subcomponent unless the ability explicitly represents multiple impacts.

Broken condition affects future effective profiles; it does not retroactively alter the hit that caused the break.

## 7. Tool integration

Butchering, chopping, mining and similar systems query explicit capabilities. If the equipped item provides the capability, the action may use it and emit appropriate wear.

This prevents category leakage such as every axe-shaped weapon automatically becoming a woodcutting tool.

## 8. Sharpen request

Conceptual request:

```text
Sharpen(player, gearInstanceId, whetstoneItem)
```

Validate:

- ownership;
- blade supports sharpening;
- no conflicting reservation;
- allowed context (field/home);
- valid whetstone/resource;
- item is not structurally ineligible under current one-pool rules.

Reserve consumable, compute bounded restoration, commit once, persist, release.

## 9. Repair request

```text
Repair(player, gearInstanceId, stationKey, materialSelection)
```

Validate station, repair family, exact material requirements, reservations and target state. Repair to the authored cap; v1 does not erode permanent max durability.

Repairing Fine gear should either reject as unnecessary or consume nothing. It must never become a material sink through duplicate clicks.

## 10. Upgrade request

```text
Upgrade(player, gearInstanceId, targetNodeKey, stationKey)
```

The service resolves the definition from the current node and target. Client-supplied material lists are hints at most; the server resolves legal ingredients from authoritative inventory.

The mutation is one transaction boundary across:

- gear reservation;
- ingredient reservations;
- inventory consumption;
- quality/result calculation;
- gear instance mutation;
- persistence/audit;
- Journal delta where applicable.

If the underlying database cannot provide one SQL transaction across every subsystem, use durable transaction IDs/state so crash reconciliation is idempotent.

## 11. Concurrency

A gear instance under upgrade/repair cannot simultaneously:

- move containers;
- equip/unequip;
- be consumed/salvaged;
- receive another upgrade/repair;
- be traded by future systems.

Wear arriving during a committed upgrade/repair must have deterministic ordering. Recommended v1: lock/reserve the gear instance for the short mutation; queue/reject incompatible actions rather than attempting merge semantics.

## 12. Login/restart reconstruction

On login:

1. load persistent gear instances;
2. validate referenced definition keys;
3. clamp impossible durability values and log corruption;
4. reconcile interrupted transactions before allowing gear mutation;
5. derive current effects;
6. send presentation state only after authority is stable.

Never regenerate hidden quality on login.

## 13. WotLK item-template bridge

The AzerothCore/WotLK item object is the visible/equipment carrier, but Thal'vaeth persistent metadata is authoritative for custom progression.

A bridge may map gear instances to item GUIDs/templates for visuals/equipment slots. It must detect missing/duplicate bridge records and fail safely. Replacing the visible template during an upgrade must not create a second persistent gear instance.

## 14. UI protocol needs

Normal client presentation needs only safe derived data:

- gear instance presentation ID;
- display/archetype/node key if known;
- condition band or permitted condition representation;
- legal known upgrade choices;
- required known materials/station information;
- action result/error.

Never transmit hidden quality number/roll, undiscovered node names or internal balance coefficients merely because the addon could hide them.

## 15. Error classes

Stable server errors should distinguish at least:

- unknown gear instance;
- not owner;
- busy/reserved;
- invalid transition;
- missing knowledge;
- missing diagram;
- wrong station;
- missing materials;
- destination/capacity invalid;
- already fully repaired;
- cannot sharpen;
- transaction conflict;
- internal definition error.

Player-facing text can remain diegetic and concise.

## 16. Runtime acceptance

Before adding broad gear content, automated/manual tests must demonstrate deterministic reconstruction, one-time quality resolution, wear idempotency, atomic repair/upgrade, condition-derived effects, death persistence, and no client path that mutates authoritative gear state.