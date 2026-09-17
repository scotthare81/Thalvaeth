# Gear implementation specification

This document turns `GEAR.md` into a server-authoritative implementation contract. It is subordinate to established design canon where this file does not explicitly lock a runtime rule.

## 1. Core invariants

1. Gear is persistent kit. Equipped/upgraded gear is not lost when a run ends in death.
2. Runs do not primarily drop replacement gear. They provide materials, diagrams, knowledge and rare supplies used to improve persistent kit.
3. Gear power comes from authored upgrade definitions, discovered knowledge, material choice, hidden quality and current condition — never a generic item-level treadmill.
4. Durability creates upkeep and field decisions; it must not permanently delete a persistent weapon or armour piece merely because condition reaches Broken.
5. Upgrade, repair and salvage mutations are server-authoritative and transactional.
6. Client/addon code may present state and request actions but may not grant diagrams, improve quality, repair condition or select an upgrade result outside server validation.

## 2. Persistent gear identity

Every upgradeable persistent piece requires a durable gear-instance identity independent of the underlying WotLK item template. Recommended persisted fields:

- character GUID;
- gear instance ID;
- base/archetype key;
- current upgrade-node key;
- material/variant key where relevant;
- hidden quality band/value;
- condition state and exact durability value;
- maximum durability after authored modifiers;
- bound slot/category;
- crafted/upgraded timestamp and provenance where useful;
- schema version.

Never encode authoritative progression solely in display name, item entry, enchant slot, addon SavedVariables or tooltip text.

## 3. Stable key namespaces

Use stable data keys rather than player-facing names. Suggested namespaces:

- `gear.weapon.*`
- `gear.armour.*`
- `gear.tool.*`
- `gear.pack.*`
- `gear.charm.*`
- `upgrade.*`
- `material.*`
- `diagram.*`

Keys survive copy changes. Display names can change without invalidating persistence.

## 4. Upgrade graph

Upgrades are an authored directed graph, not an unconstrained stat reroll.

Each upgrade node defines at minimum:

- stable node key;
- allowed source node(s);
- output archetype/form;
- required discovery/process gates;
- required diagram if diagram-gated;
- required station;
- exact material families/quantities;
- optional material substitutions explicitly authored;
- quality input policy;
- condition/max-durability transformation policy;
- resulting combat/survival/tool modifiers;
- next legal node(s).

The server validates the current gear node before allowing the transition. A client cannot skip directly to a later node.

## 5. Weapon graph

The first canonical spine is:

`Crude Dagger → Flint/Bone Blade → Iron Knife / Short Blade → Steel Blade → style branch`

The late branch supports:

- fast line: fighting daggers / one-handed short blades → paired/twin blades;
- heavy line: war cleaver / iron greatblade → steel greatsword or greataxe → masterwork heavy form.

Exact combat coefficients remain balance data, not hard-coded progression logic.

Weapon definitions expose authored axes such as:

- base damage profile;
- swing cadence;
- reach class;
- Vigor cost per attack/action;
- noise contribution;
- stagger/cleave/bleed hooks;
- butcher/chop/tool capability;
- durability wear profile.

Secondary utility is explicit capability data. A dagger may satisfy butchering; a greataxe may satisfy chopping; a greatsword does not inherit either merely because it is a weapon.

## 6. Material variants

Material choice may create side-grades where canon allows it. Iron, steel and bronze are not represented as a single numeric rarity ladder.

For blades, authored material profiles may modify:

- damage/edge performance;
- Vigor cost/weight;
- durability maximum;
- wear rate;
- repair material requirements.

Current canon intent: steel holds an edge well; bronze can be lighter but softer/faster-wearing; iron is reliable/common. Exact multipliers are balance data and require tests before lock.

## 7. Armour graph

Armour follows authored class progression:

- Cloth/Rags;
- Leather;
- Mail;
- Plate.

The body piece establishes the dominant class for presentation/build rules, while individual worn pieces contribute their own authored noise, Vigor and protection modifiers.

Progression gates remain:

- Leather → Tanning;
- Mail → Smelting/Iron;
- Plate → Steelworking/Forge.

Mail/plate layering over a quilted base remains a design dependency; implementation must not accidentally consume/delete the persistent underlayer when upgrading outer armour.

## 8. Hidden quality

Crafted/upgraded gear carries server-owned hidden quality consistent with `CRAFTING.md` and `ITEMS.md`.

Rules:

- no rarity colour, stars or numeric quality score player-facing;
- quality affects authored performance bands such as edge retention, maximum durability, mitigation or tool effectiveness;
- input quality biases result according to the crafting quality model;
- quality cannot be rerolled by cancelling, reconnecting, moving containers or repeatedly opening UI;
- an upgrade attempt obtains its result once at transaction commit;
- stack semantics never apply to unique persistent gear instances.

Player-facing clues may use names/descriptions and felt performance, but those strings are not authority.

## 9. Durability model

Canonical player-facing condition bands:

`Fine → Worn → Damaged → Broken`

Internally store integer current/max durability so wear is deterministic and testable. Condition bands derive from thresholds in data/config; do not persist only the label.

Recommended first thresholds for testing, explicitly provisional until balance pass:

- Fine: > 70%
- Worn: > 35% to 70%
- Damaged: > 0% to 35%
- Broken: 0%

Broken means severe loss of function/performance, **not item deletion**.

## 10. Wear events

Wear must come from explicit server events.

Weapons/tools may wear from:

- successful combat strikes;
- selected blocked/parried/heavy-impact events if authored;
- butchering/chopping/mining/tool actions;
- special high-stress actions.

Armour may wear from:

- mitigated incoming physical damage;
- selected creature attacks/abilities;
- environmental damage only where explicitly authored.

Do not apply durability loss from arbitrary elapsed time.

One gameplay event must not double-apply wear through multiple hooks. Centralize wear mutation or use an idempotent event token.

## 11. Broken behaviour

At Broken:

- item remains owned and equipped/stored unless normal slot rules require otherwise;
- weapon/tool effectiveness receives a severe authored penalty and advanced utility may be disabled;
- armour mitigation is severely reduced;
- the item remains repairable;
- death does not delete it;
- Broken cannot be exploited to obtain free replacement copies.

Exact Broken penalties are balance data. V1 tests need only prove the state transition and that the item persists.

## 12. Sharpening vs repair

Sharpening and structural repair are separate concepts.

**Sharpening**:
- primarily bladed weapons/tools;
- whetstone-supported;
- may be allowed in the field;
- restores edge/sharpness-related condition only within authored limits;
- cannot reconstruct a structurally Broken item unless an explicit field-repair recipe says so.

**Repair/mending**:
- station-appropriate structural restoration;
- Forge for metal weapons/heavy armour;
- Stitch Table for cloth/leather;
- Workbench for suitable tools/packs/components;
- consumes authored materials;
- server validates station, gear instance, reservation and output state.

V1 may use one durability pool per item while retaining distinct action rules; a later edge/structure split must not require changing gear identity.

## 13. Upgrade transaction

Canonical order:

1. receive upgrade intent for gear instance + target node;
2. verify ownership and no conflicting reservation;
3. verify current node can reach target;
4. verify required Journal/discovery/process/diagram knowledge;
5. verify correct Monastery station;
6. resolve and reserve exact materials;
7. verify resulting item can remain in/equip to a legal location;
8. compute quality/result exactly once;
9. atomically consume materials and mutate gear instance;
10. persist gear state;
11. emit Journal/UI delta and audit event;
12. release reservation.

Failure before commit does not consume inputs unless the relevant crafting failure contract explicitly defines destructive experimentation. Known gear upgrades/repairs should normally be deterministic once requirements are met.

## 14. Repair transaction

Repair follows the same reservation/atomicity rules. A reconnect, duplicate click or station-close cannot consume materials twice or apply repair twice.

Repair policy must explicitly define whether maximum durability can degrade after repeated structural repairs. **V1 default: no permanent max-durability erosion.** This avoids an unreviewed gear-destruction treadmill; such a system can be added later only deliberately.

## 15. Death/run interaction

Persistent equipped/upgraded gear survives run death with its current gear identity, quality and durability state. Death does not automatically restore condition.

If the player dies with a Damaged weapon, they wake in their Monastery quarters with that weapon still Damaged. Death is not a free repair.

At-risk spare gear semantics must follow the Inventory risk-class contract rather than being inferred from “gear” as a category. V1 should avoid introducing lootable replacement gear until that policy is explicitly exercised by tests.

## 16. Inventory/bulk interaction

Worn/wielded gear contributes zero carried bulk under the current Inventory contract. Unequipped/spare gear uses authored bulk values and requires available main-bag capacity.

An equip/unequip, upgrade or form-change that changes bulk must preflight destination capacity atomically.

No upgrade may use a temporary container move to bypass capacity.

## 17. Journal integration

Gear Journal presentation reads server-owned knowledge:

- unknown upgrade silhouettes remain obscured;
- discovered diagrams/processes reveal only the appropriate node/branch;
- owning materials does not automatically reveal an unknown upgrade;
- successfully creating/upgrading may promote the corresponding gear knowledge according to Journal rules;
- hidden quality values never leak through Journal payloads.

## 18. Diagnostics

Required before content expansion. Suggested commands:

- `.thal gear dump [target]`
- `.thal gear setdur <instance> <value>`
- `.thal gear damage <instance> <amount>`
- `.thal gear repair <instance>` dev-only
- `.thal gear grant-diagram <key>` through Journal authority
- `.thal gear upgrade-test <instance> <node>` dev-only dry-run where possible

Debug output may expose hidden values to authorized GMs but never normal clients.

## 19. Acceptance criteria

Implementation is ready to expand when tests prove:

1. gear instance identity survives relog/restart/death;
2. upgrade graph cannot be skipped;
3. missing discovery/diagram/station/material requirements reject safely;
4. duplicate upgrade/repair requests cannot double-consume or double-apply;
5. hidden quality is rolled once and remains stable;
6. wear transitions deterministically through condition bands;
7. Broken gear persists and can be repaired;
8. death preserves exact durability rather than repairing/deleting gear;
9. field sharpening cannot bypass structural repair rules;
10. inventory bulk/capacity remains valid through equip/unequip/form changes;
11. Journal/UI cannot reveal unknown nodes or hidden quality;
12. restart during mutation reconciles without item duplication or loss.