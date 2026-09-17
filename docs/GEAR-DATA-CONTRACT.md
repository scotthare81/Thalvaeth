# Gear data contract

This document pins the data shape needed to implement Gear without hard-coding individual weapons and armour pieces throughout C++.

## 1. Separation of concerns

Keep four concepts separate:

1. **Archetype** — what the thing fundamentally is: dagger, greatsword, leather body, boots.
2. **Upgrade node** — where this persistent instance sits in its authored progression graph.
3. **Material profile** — iron/steel/bronze/etc. where relevant.
4. **Gear instance** — the player's unique persistent object with quality and durability.

A fifth concept, **presentation**, maps safe known state to names/icons/tooltips. Presentation is not authority.

## 2. Definition keys

Examples:

```text
gear.weapon.dagger
gear.weapon.greatsword
gear.armour.body
gear.armour.feet

upgrade.weapon.crude_dagger
upgrade.weapon.iron_short_blade
upgrade.weapon.steel_blade
upgrade.weapon.fast.fighting_dagger
upgrade.weapon.heavy.war_cleaver

material.iron
material.steel
material.bronze
```

The exact registry should avoid duplicating the same semantic key under both `gear.*` and `upgrade.*`. Before coding, settle whether graph-node keys use `gear.*` or `upgrade.*` as the persisted node namespace and use one convention consistently. Existing draft docs currently use both conceptually; this file explicitly flags that as a pre-code normalization task.

## 3. Suggested gear definition

```text
GearArchetypeDefinition
  key
  category                 weapon | armour | tool | pack | charm
  equipSlot
  handedness               none | one_hand | two_hand
  carriedBulk
  capabilities[]
  baseProfileKey
  presentationFamilyKey
```

Capabilities are stable semantic keys such as:

```text
capability.butcher
capability.butcher_heavy
capability.chop
capability.skin
capability.mine
```

Never infer capability from localized item name or display subclass.

## 4. Suggested upgrade definition

```text
GearUpgradeDefinition
  nodeKey
  archetypeKey
  sourceNodeKeys[]
  gatePolicy
  requiredKnowledgeKeys[]
  requiredDiagramKey?
  stationKey
  ingredientRequirements[]
  allowedMaterialProfiles[]
  requiredMaterialProfile?
  qualityPolicyKey
  durabilityPolicyKey
  effectProfileKey
  nextNodeKeys[]
```

`nextNodeKeys` may be derived from sources at registry validation time; storing both is optional. If both are authored, startup validation must ensure they agree.

## 5. Ingredient requirement

```text
IngredientRequirement
  materialFamilyKey
  quantity
  minimumKnownQuality?
  consumed = true
  substitutionGroup?
```

The server resolves actual inventory instances/stacks. Client UI never supplies authoritative ingredient identity.

## 6. Material profile

```text
GearMaterialProfile
  key
  allowedArchetypeFamilies[]
  impactModifier
  vigorModifier
  maxDurabilityModifier
  wearModifier
  repairFamilyKey
  presentationKey
```

These fields should preferably be fixed-point integers/basis points rather than floating-point persistence values where exact deterministic behaviour matters.

## 7. Effect profile

Weapon example:

```text
WeaponEffectProfile
  impact
  cadence
  reachClass
  vigorCost
  noise
  stagger
  cleave
  bleed
  wearProfileKey
```

Armour example:

```text
ArmourEffectProfile
  mitigation
  infectionResistance
  vigorBurden
  noise
  warmth
  footing
  wearProfileKey
```

Not every field applies to every slot. Use explicit defaults rather than magic zero semantics when ambiguity matters.

## 8. Persistent gear instance

Canonical persistence requires at least:

```text
thalvaeth_player_gear
  gear_instance_id
  owner_guid
  item_guid
  archetype_key
  node_key
  material_key
  quality_value
  durability_current
  durability_max
  schema_version
  created_at
  updated_at
```

Recommended constraints:

- primary key `gear_instance_id`;
- unique `item_guid` when bridged to one visible WotLK item;
- owner index;
- durability current/max non-negative validation in service layer;
- no duplicate persistent instance for one visible item.

Exact SQL types/migration naming follow repository conventions when implementation starts.

## 9. Quality representation

Store an internal deterministic quality value/band sufficient for effect resolution. Do not store only the player-facing adjective.

If Crafting ultimately standardizes a common quality scale, Gear must reuse it rather than inventing another scale. The gear registry maps common quality into gear-specific effects.

## 10. Durability representation

Store exact integer current and maximum durability. Condition is derived.

Do not store Fine/Worn/Damaged/Broken as the only source of truth. A cached condition may exist for diagnostics but must be recalculable.

## 11. Transaction record

Upgrade/repair crash safety should use a durable operation record if one SQL transaction cannot cover all dependencies.

```text
GearTransaction
  transactionId
  ownerGuid
  gearInstanceId
  type                 upgrade | repair | sharpen
  sourceNode?
  targetNode?
  state                prepared | committed | rolled_back
  runId?
  createdAt
  committedAt?
```

Ingredient reservation IDs/audit detail may live in the shared Inventory transaction system rather than duplicate tables.

## 12. Definition versioning

Balance changes will alter definitions after gear already exists.

Rules:

- stable keys never change casually;
- changing damage/Vigor/noise coefficients should normally affect all instances through current definitions without DB migration;
- changing graph identity/material meaning requires explicit migration/reconciliation;
- removing a persisted node key requires a migration map;
- never silently reset an unknown node to Crude Dagger;
- startup/login should quarantine/log invalid gear rather than destroy it.

## 13. Item-template bridge

The visible WotLK item template may change as form changes. The persistent `gear_instance_id` does not.

Upgrade example:

```text
G42: Crude Dagger item-template A
→ valid upgrade commit
G42: Iron Short Blade item-template B
```

There must still be exactly one G42. The bridge operation cannot create B and leave A as a second owned persistent weapon.

## 14. Presentation contract

Normal client may receive:

- safe display key/name/icon for known current form;
- condition band;
- capability descriptions that the player has legitimately learned/experienced;
- known legal next upgrades;
- known requirements.

Normal client must not receive:

- raw hidden quality;
- RNG state;
- unknown diagram/node names;
- exact hidden balance multipliers intended to remain experiential;
- GM transaction IDs/debug provenance unless needed for support mode.

## 15. Registry validation tests

Startup/unit validation must catch:

- duplicate keys;
- unknown archetype/material/profile references;
- invalid station keys;
- impossible graph edges;
- unintended cycles;
- unreachable non-root nodes;
- diagram-required edge with missing diagram key;
- material-required edge with no legal material;
- negative durability/wear definitions;
- capability keys not registered;
- two definitions claiming the same persisted key with incompatible categories.

## 16. Pre-code normalization decision

Before C++ lands, normalize the draft graph-key convention across `GEAR-IMPLEMENTATION-SPEC.md`, `GEAR-UPGRADE-GRAPH.md` and this file. The system needs one stable persisted node namespace. This is deliberately documented now rather than allowing an implementation to choose accidentally.