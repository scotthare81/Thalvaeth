# Gear implementation specification

This document turns `GEAR.md` into a server-authoritative implementation contract. It is subordinate to established design canon where this file does not explicitly lock a runtime rule.

## 1. Core invariants

1. Gear is persistent kit. Equipped/upgraded gear is not lost when a run ends in death.
2. Runs do not primarily drop replacement gear. They provide materials, diagrams, knowledge and rare supplies used to improve persistent kit.
3. Gear power comes from authored upgrade definitions, discovered knowledge, material choice, hidden quality and current condition — never a generic item-level treadmill.
4. Durability creates upkeep and field decisions; it must not permanently delete a persistent weapon or armour piece merely because condition reaches Broken.
5. Upgrade, repair and salvage mutations are server-authoritative and transactional.
6. Client/addon code may present state and request actions but may not grant diagrams, improve quality, repair condition or select an upgrade result outside server validation.
7. One persistent gear instance has one authoritative owner and one authoritative current form.
8. A visual WotLK item-template swap never means the player obtained a second persistent gear instance.
9. Known gear construction is deterministic once requirements are met; it is not a gambling/reroll surface.
10. No future content should require replacing the persistence model merely to add more authored nodes.

## 2. Persistent gear identity

Every upgradeable persistent piece requires a durable gear-instance identity independent of the underlying WotLK item template. Recommended persisted fields include character GUID, gear instance ID, visible item GUID bridge, archetype key, current node key, material key, hidden quality, exact current/max durability, slot/category, timestamps/provenance where useful and schema version.

Never encode authoritative progression solely in display name, item entry, enchant slot, addon SavedVariables or tooltip text.

The persistent identity is the continuity object. `G42` can begin as a Crude Dagger and later be represented visually as an Iron Short Blade without becoming a new progression object.

## 3. Stable keys

Use stable data keys rather than player-facing names. Gear requires namespaces for archetypes/nodes, materials, diagrams, capabilities and wear profiles. Keys survive copy changes; display names can change without invalidating persistence.

**Pre-code requirement:** normalize whether persisted graph-node keys use `gear.*` or `upgrade.*`. Draft documents currently use both conceptually; implementation gets exactly one canonical convention.

## 4. Upgrade graph

Upgrades are an authored directed graph, not an unconstrained stat reroll. Each node/edge defines source(s), output form, discovery/process/diagram gates, station, material requirements/substitutions, quality policy, durability transformation, effects and legal next nodes.

The server validates the current node before allowing transition. Graphs may converge and branch. A Bone Blade and Flint Blade may both reach the first Iron form. Branches are explicit choices, not automatically selected by a calculated score.

## 5. Weapon graph

Canonical spine:

`Crude Dagger → Flint/Bone Blade → Iron Knife / Short Blade → Steel Blade → style branch`

Fast line: fighting daggers / one-handed short blades → paired/twin blades.

Heavy line: war cleaver / iron greatblade → steel greatsword or greataxe → masterwork heavy form.

Weapon definitions expose impact, cadence, reach, Vigor burden, noise, stagger/cleave/bleed hooks, explicit utility capability, wear profile and carried bulk. Exact coefficients remain balance data.

Secondary utility is explicit capability data. A dagger may satisfy butchering; a greataxe may satisfy chopping; a greatsword does not inherit either merely because it is a weapon.

## 6. Paired weapon representation

Before dual-wield code lands, choose one persistence model:

- **logical-set model:** one persistent gear instance represents the pair while the WotLK bridge renders/equips two carriers; or
- **linked-instance model:** two persistent gear instances are explicitly paired and upgraded/repaired under a linked transaction.

The chosen model must define durability, repair cost, death persistence, item-template bridging and missing-carrier recovery. Do not let WotLK main/off-hand mechanics decide this accidentally.

## 7. Material variants

Iron, steel and bronze are authored side-grade/material profiles where allowed, not a universal rarity ladder. Profiles may modify edge/impact, Vigor/weight, max durability, wear and repair family.

Current intent: iron reliable/common; steel best conventional edge retention/strong structure; bronze lighter but softer/faster wearing. Exact multipliers remain balance data. Material modifiers stay bounded so weapon form matters more than metal gimmicks.

## 8. Armour graph

Armour classes are Cloth/Rags → Leather → Mail → Plate, with Tanning, Smelting/Iron and Steelworking/Forge gates. The body establishes dominant presentation/build class, while actual pieces contribute their own effects.

Mail/plate layering over quilted base is a real dependency; implementation must not accidentally consume/delete the persistent underlayer. Hardened Leather and Splinted Mail remain valid late/endgame destinations even after Plate is known.

## 9. Supporting slots

Head, hands, feet, legs/waders, back/pack and warmth slots contribute authored survival axes rather than blindly copying body tier. Pack capacity remains Inventory authority. Shirt/tabard/neck remain warmth/weather layers independent of armour class.

Do not implement future cold/heat penalties merely because these slots reserve warmth data; environmental effects activate only when those systems are designed.

## 10. Hidden quality

Gear carries server-owned hidden quality consistent with Crafting/Items. No rarity colour, stars or numeric quality is player-facing. Quality can affect edge retention, max durability, mitigation/tool effectiveness and other authored bands.

Quality cannot be rerolled by cancelling, reconnecting, moving containers, reopening UI, repairing or logging in. An upgrade obtains its result once at transaction commit. Player-facing clues may use names/descriptions and felt performance, but strings are not authority.

**Open before code:** define quality inheritance when substantially rebuilding the same persistent item. Previous workmanship should matter somehow if continuity is meaningful, but exact weighting against new material quality must be canonical and shared with Crafting.

## 11. Durability model

Player-facing bands are:

`Fine → Worn → Damaged → Broken`

Store integer current/max durability; derive the band. Provisional test thresholds:

- Fine: >70%
- Worn: >35–70%
- Damaged: >0–35%
- Broken: 0%

Broken is severe impairment, never automatic deletion. Normal v1 presentation should prefer qualitative bands; GM/debug may show exact current/max.

## 12. Condition effects

Condition modifies the effective profile, not the base definition.

- Fine: full profile.
- Worn: mainly an upkeep signal with mild degradation.
- Damaged: clearly compromised; meaningful push/extract decision.
- Broken: severe impairment.

Condition must not accidentally grant advantages: damaged armour does not become quieter/lighter, and a damaged heavy weapon does not become cheaper in Vigor unless explicitly authored. Temporary poisons/buffs remain separate.

## 13. Wear events

Wear comes from explicit semantic server events, never elapsed time. Weapons/tools may wear from combat contacts and authored tool actions; armour from qualifying incoming impacts/abilities; environment only when explicitly authored.

One gameplay event cannot double-apply wear through multiple hooks. Centralize mutation/use idempotent event identity. Semantic classes such as light/standard/heavy/severe convert through form/material profiles into exact integer loss.

## 14. Broken behaviour

Broken gear remains owned/equipped/stored, receives severe authored effectiveness penalties, may lose advanced utility, remains repairable, survives death and cannot generate a free replacement copy. Normal feedback should indicate that proper structural work is required where applicable.

## 15. Sharpening vs repair

**Sharpening:** blade-focused, whetstone-supported, potentially field-usable, bounded restoration, cannot reconstruct structural failure.

**Structural repair:** station-appropriate, material-consuming restoration at Forge/Stitch Table/Workbench as appropriate.

V1 may use one durability pool while preserving distinct action rules. A later edge/structure split must not change gear identity. Field sharpening extends a run; it must not make home repair irrelevant.

## 16. Upgrade transaction

Canonical order:

1. receive gear instance + target intent;
2. verify ownership/reservation;
3. validate source→target edge;
4. validate Journal/process/diagram knowledge;
5. validate canonical station/context;
6. resolve/reserve exact materials;
7. preflight legal equipment/container/bulk result;
8. establish durable prepared operation if required;
9. compute one-time quality/result;
10. atomically consume materials + mutate gear;
11. update visible item-template bridge without duplicating identity;
12. persist gear/transaction outcome;
13. emit safe Journal/UI/audit delta;
14. release reservations.

Known upgrades normally consume nothing on pre-commit failure. Experimentation failure semantics belong to Crafting and must not be accidentally applied to already-known station work.

## 17. Repair transaction

Repair uses the same reservation/idempotency rules. Duplicate click, reconnect or station-close cannot consume twice or apply twice.

V1 has **no permanent max-durability erosion**. Repair inputs should semantically match the item: cloth/thread, leather/cord, rings/rivets, plate/metal/fittings, etc.

## 18. Salvage/destruction

Broken is not salvage. Automatic destruction of persistent gear is forbidden.

If voluntary dismantling arrives later it must be explicit, home/station based, confirmed, transactional and have an authored salvage return. V1 omits voluntary destruction of core persistent gear.

## 19. Death/run interaction

Persistent equipped/upgraded gear survives death with exact identity, quality and durability. Death does not repair it. A Damaged weapon is still Damaged when the Remnant wakes in personal quarters.

At-risk spare gear follows the Inventory risk-class contract, not the word “gear.” Run finalization must freeze/reconcile conflicting gear/inventory transactions before classifying contents.

## 20. Inventory/bulk interaction

Worn/wielded gear has zero carried bulk under current Inventory canon. Spare gear uses authored bulk and needs main-bag capacity. Equip/unequip/form-change preflights capacity atomically. No temporary move can bypass bulk.

Pack progression is Gear content but capacity remains Inventory authority; equipping a visual pack template alone cannot alter capacity.

## 21. Journal integration

Unknown upgrades remain silhouettes/obscured; diagrams/processes reveal only appropriate knowledge; owning materials does not reveal recipes; successful work can promote knowledge according to Journal rules; hidden quality never leaks.

Unknown nodes should not be transmitted merely so Lua can hide them.

Responsibilities remain distinct: **Journal = what I know; Gear = what I own/wear; Station = what work I can perform here.**

## 22. Station interaction

Stations are authoritative contexts, not menu flavour. Validate canonical station key, proximity/context, ownership/location, knowledge, materials/reservations, transition/repair legality and destination capacity.

Closing UI does not undo committed work. Moving away before a channelled/prepared action commits cancels safely.

## 23. Definition/version safety

Stable keys outlive builds. Balance coefficient changes normally flow through definitions; removed/renamed persisted nodes require migration mapping; unknown definitions are quarantined/logged rather than reset; quality never rerolls because definitions changed; template changes never alter persistent identity.

## 24. Diagnostics

Required before content expansion. Suggested commands:

- `.thal gear dump [target]`
- `.thal gear setdur <instance> <value>`
- `.thal gear damage <instance> <amount>`
- `.thal gear repair <instance>` dev-only
- `.thal gear grant-diagram <key>` through Journal authority
- `.thal gear upgrade-test <instance> <node>` dev-only
- `.thal gear transactions <instance>`
- `.thal gear reconcile <transaction>`
- `.thal gear bridge <instance>`

GM output may expose hidden values; normal clients never receive them. Destructive recovery logs before/after state.

## 25. Acceptance criteria

Implementation is ready to expand when tests prove:

1. identity survives relog/restart/death;
2. graph cannot be skipped;
3. missing knowledge/diagram/station/material rejects safely;
4. duplicate mutation cannot double-consume/apply;
5. quality rolls once and persists;
6. wear/bands are deterministic;
7. Broken persists and repairs;
8. death preserves exact durability;
9. sharpening cannot bypass structural repair;
10. bulk remains valid through form/equip changes;
11. Journal/UI cannot leak unknown nodes/quality;
12. restart reconciliation cannot duplicate/lose gear;
13. template swaps preserve one persistent identity;
14. invalid definitions never erase progression;
15. explicit capabilities prevent name/category tool leakage;
16. condition penalties cannot create accidental Vigor/noise advantages;
17. first fast/heavy branch children are mechanically distinct through data hooks, not damage alone.