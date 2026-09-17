# Gear & weapons — upgrade-only

**You never find gear in a run — only materials to make what you have better** ([MATERIALS.md](MATERIALS.md)). All power progression is *crafted and discovered*, not looted.

- **Gear persists.** It is **not** lost on death — only the run haul + satchel are ([SURVIVAL.md](SURVIVAL.md)). You keep your upgraded kit; a bad run costs you materials and time, not your sword.
- **Upgrade at Monastery stations** (forge, workbench, stitch table) with **materials + a discovered diagram** for the next tier ([CRAFTING.md](CRAFTING.md)).
- **Tiers are gated by the discovery tree** — leather needs Tanning, mail needs Smelting, plate needs Steelworking (the Charcoal→Forge spine).
- Look holds L15: patchwork → worn-but-better. Even late gear reads as survived-in, not parade armour.

## Implementation-ready contracts

This file owns the broad design and progression fantasy. The implementation-ready contracts are:

- [GEAR-IMPLEMENTATION-SPEC.md](GEAR-IMPLEMENTATION-SPEC.md) — authority, identity, durability, upgrade/repair transactions;
- [GEAR-RUNTIME-SPEC.md](GEAR-RUNTIME-SPEC.md) — runtime services, effect resolution and WotLK bridge;
- [GEAR-DATA-CONTRACT.md](GEAR-DATA-CONTRACT.md) — registry/persistence/data shapes and versioning;
- [GEAR-UPGRADE-GRAPH.md](GEAR-UPGRADE-GRAPH.md) — stable progression graph semantics;
- [GEAR-CONTENT-MATRIX.md](GEAR-CONTENT-MATRIX.md) — progression gates/material intent/authoring checklist;
- [GEAR-BALANCE-FRAMEWORK.md](GEAR-BALANCE-FRAMEWORK.md) — weapon/armour tradeoff and upkeep tuning framework;
- [GEAR-PLAYER-EXPERIENCE.md](GEAR-PLAYER-EXPERIENCE.md) — player-facing knowledge, station and durability experience;
- [GEAR-FAILURE-RECOVERY.md](GEAR-FAILURE-RECOVERY.md) — crash/race/dupe/loss recovery contract;
- [EQUIPMENT-CONTRACT.md](EQUIPMENT-CONTRACT.md) — logical equipment roles, paired weapons, tools, layering, packs and swapping;
- [EQUIPMENT-TEST-VECTORS.md](EQUIPMENT-TEST-VECTORS.md) — deterministic equipment/paired-weapon acceptance cases;
- [COMBAT-IMPLEMENTATION-SPEC.md](COMBAT-IMPLEMENTATION-SPEC.md) — attack lifecycle, Vigor, reach, damage, stagger, bleed, cleave, noise, interruption and Broken combat;
- [COMBAT-RUNTIME-SPEC.md](COMBAT-RUNTIME-SPEC.md) — server action IDs, timing, spatial resolver, status/noise/reaction services and AzerothCore bridge;
- [COMBAT-WEAPON-BEHAVIOUR.md](COMBAT-WEAPON-BEHAVIOUR.md) — dagger/paired/greatsword/greataxe combat identity and move grammar;
- [COMBAT-CREATURE-REACTIONS.md](COMBAT-CREATURE-REACTIONS.md) — anatomy, stagger personality, hearing, AI reactions and creature attack grammar;
- [COMBAT-TEST-VECTORS.md](COMBAT-TEST-VECTORS.md) — deterministic combat acceptance cases;
- [COMBAT-V1-SLICE.md](COMBAT-V1-SLICE.md) — first playable combat proof;
- [COMBAT-CODING-PLAN.md](COMBAT-CODING-PLAN.md) — staged combat implementation sequence;
- [GEAR-TEST-VECTORS.md](GEAR-TEST-VECTORS.md) — deterministic Gear acceptance tests;
- [GEAR-V1-SLICE.md](GEAR-V1-SLICE.md) — first playable Gear proof;
- [GEAR-CODING-PLAN.md](GEAR-CODING-PLAN.md) — staged Gear implementation sequence.

Where an older conceptual statement here conflicts with a later implementation-ready contract, resolve the contradiction in documentation before C++ rather than choosing silently in code.

## Starting loadout

The Remnant begins with a Crude Dagger, Rag Armour, Rag Hood and small pack/gather satchel. Hands, feet, improved pack, charms and serious weapons are earned through crafting and discovery.

## Weapons

Everyone starts with the Crude Dagger and climbs the shared early line:

`Crude Dagger → Flint/Bone Blade → Iron Knife / Short Blade → Steel Blade`

At Steel the combat line can commit toward fast paired weapons or heavy two-handed weapons.

### Fast style

Paired daggers/shortswords are a **single matched persistent weapon set**. They have one Gear identity, one quality result, one durability pool and one upgrade/repair path even though the WotLK client may need two visible weapon carriers.

This is now canonical. The player upgrades *their paired weapon set*, not two independent daggers.

Fast weapons emphasize cadence, mobility and bleed at lower per-hit impact. Dagger forms also retain explicit butcher capability. Combat depth comes from close spacing, low individual commitment, bounded bleed pressure and the temptation to overspend Vigor through repeated attacks.

### Heavy style

Greatswords, greataxes and heavy cleavers are one persistent two-handed weapon each. They emphasize impact, reach, stagger/cleave and higher Vigor/noise cost.

A Greatsword is combat-only by default. A Greataxe may chop wood. A War Cleaver may butcher heavy. Heavy builds therefore still benefit from carrying a dedicated skinning knife.

Heavy combat is deliberately committed: missing still spends Vigor, recovery remains, and loud impacts can make the surrounding district more dangerous.

### Combat authority

Combat is not stock WotLK auto-attack progression. Attacks are server-authoritative actions with wind-up, commit, active and recovery phases. Vigor is spent at commitment whether the attack later hits or whiffs. The server resolves reach, facing, target geometry, damage, stagger, bleed, cleave, noise and durability wear.

Fast and heavy are different decision rhythms rather than simple DPS tiers. See the Combat contracts above.

### Weapon bulk

Worn/wielded gear is free under the Inventory bulk contract. Carried spares use authored bulk. Current intent is dagger 1, one-handed 2, paired set 2 and heavy/two-handed 4, all tunable data values.

## Armour

Armour progresses through Cloth/Rags, Leather, Mail and Plate, but heavier is a build choice rather than a universal upgrade. Protection trades against noise, Vigor burden and mobility.

The body establishes dominant armour-class identity, while actual worn pieces contribute their own effects. Mixed equipment remains legal unless explicitly prohibited.

Quilted/gambeson layers remain meaningful under Mail/Plate and are persistent logical gear. Native WotLK slot limitations must not cause the underlayer to be consumed or forgotten.

## Equipment roles

Thal'vaeth uses logical equipment roles rather than allowing native WotLK slots to dictate game design. These include Head, Body, Shirt, Tabard, Neck, Hands, Legs/Waist, Feet, Back/Pack, Weapon, Utility/Tool and two Charm roles.

Shirt/Tabard/Neck remain the independent warmth layer. Back/Pack controls authoritative capacity. Pack visuals alone never alter capacity, and a downgrade/unequip cannot create inventory overflow.

Weapon roles distinguish single, paired and heavy weapons. A dedicated skinning knife can be available as a logical tool without requiring it to remain visibly equipped beside a two-handed weapon.

See [EQUIPMENT-CONTRACT.md](EQUIPMENT-CONTRACT.md) for the authoritative mapping and transaction rules.

## Durability

Player-facing condition remains:

`Fine → Worn → Damaged → Broken`

Broken means severe impairment, not deletion. Gear persists through death with its exact condition. Death and extraction are never free repairs.

Blades may be sharpened with a whetstone where allowed; structural repair uses the appropriate Monastery station. Paired weapons are sharpened/repaired as one logical set.

In combat, Broken is a severe degraded profile: edge/bleed/stagger/cleave/tool capability can fall sharply or selected advanced attacks can lock out. A Broken heavy weapon does not conveniently become cheap to swing simply because it performs badly.

## Upgrading

Upgrades require the correct authored combination of current gear node, discovered process/diagram, station and materials. Hidden quality is resolved server-side and persists; it is not a rarity colour or client-visible numeric roll.

The Gear instance survives a form/template change. Upgrading a dagger into a later weapon does not destroy the persistent identity and create an unrelated loot item.

## Open decisions before code

The paired-weapon identity question is closed: **one logical persistent matched set**.

Combat now also locks the behavioural architecture while deliberately leaving numeric tuning as data.

Still requiring explicit cross-system resolution before implementation:

- exact canonical `station.*` key spellings;
- exact stable weapon/armour/attack keys before persisted data ships;
- quality inheritance weighting across major rebuilds;
- Inventory death-risk semantics for carried spare persistent gear;
- exact combat-time weapon-swap/channel rule;
- final per-weapon Vigor/timing/reach/damage/stagger/bleed/noise values;
- first reviewed Rotwood creature anatomy/stagger/hearing profiles;
- exact native AzerothCore combat paths to suppress/reuse;
- which upgrade edges are discoverable vs diagram-only;
- whether cold becomes a tracked environmental factor rather than only future-fit equipment data.

## Related

- [CRAFTING.md](CRAFTING.md) — recipes, discovery tree, stations, durability
- [MATERIALS.md](MATERIALS.md) — upgrade materials + diagrams
- [SURVIVAL.md](SURVIVAL.md) — noise / Vigor / Infection that gear and combat consume
- [APTITUDES.md](APTITUDES.md) — charm-slot aptitude items
- [CONTENT.md](CONTENT.md) — starting/content kit