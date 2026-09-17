# Gear + weapons + durability — v1 playable slice

The first slice proves:

**persistent weapon → use it → wear it down → repair/sharpen it → discover an upgrade → spend extracted materials → improve the same persistent weapon → die → wake with that exact upgraded, still-worn weapon**.

## 1. In scope

- durable gear-instance identity;
- starting Crude Dagger;
- one pre-forge blade upgrade;
- Iron Knife / Short Blade;
- Steel Blade;
- one fast branch child;
- one heavy branch child;
- Rag Armour → Quilted Coat → Boiled Leather body path;
- hidden quality persistence;
- one integer durability pool per gear instance;
- Fine/Worn/Damaged/Broken bands;
- weapon/tool/armour wear hooks;
- field sharpening with Whetstone;
- structural repair at correct Monastery station;
- diagram/process/material gating;
- Inventory reservations/bulk integration;
- Journal knowledge integration;
- death/relog/restart persistence;
- GM/debug tooling.

## 2. Out of scope

- complete endgame weapon catalogue;
- all armour slots/classes;
- permanent max-durability decay;
- separate edge vs structural durability pools;
- reforging/respec between late branches;
- magical affixes/random stat rolls;
- rarity colours/item levels;
- lootable replacement-gear treadmill;
- PvP balance;
- sophisticated repair animation/UI;
- warmth/cold implementation beyond preserving existing slot design.

## 3. First player flow

1. Start with persistent Crude Dagger and Rag Armour.
2. Enter Rotwood and use dagger for combat/butchering.
3. Dagger condition falls from Fine toward Worn/Damaged.
4. Field Whetstone demonstrates bounded sharpening.
5. Extract materials/knowledge needed for first upgrade.
6. At Monastery station, server validates and upgrades the same gear instance.
7. Hidden quality is resolved once and persists.
8. Continue to Iron/Steel transition.
9. Learn one diagram-gated branch option.
10. Choose either first fast or first heavy branch child in test tooling/content.
11. Damage weapon to Broken; verify it remains owned and heavily impaired.
12. Repair it at the correct station.
13. Deliberately die in a run.
14. Wake in personal quarters with exact weapon node, quality and durability preserved.
15. Relog/restart and verify no rerolls, duplicates or free repairs.

## 4. Armour proof

Armour v1 only needs the body path:

`Rag Armour → Quilted Coat → Boiled Leather`

It must prove:

- upgrade graph gating;
- Stitch/appropriate station validation;
- armour wear;
- protection/noise/Vigor derived profile plumbing;
- death persistence;
- hidden quality persistence.

Do not build Mail/Plate content until this path is mechanically sound.

## 5. Balance values

Exact damage, swing time, Vigor costs, mitigation, noise, wear magnitudes, repair costs and durability maxima remain data/balance values during this slice.

The implementation must avoid baking provisional numbers into control flow. Tests should use fixture definitions where exact arithmetic matters.

## 6. Required integration

- Inventory owns location/bulk/reservations.
- Crafting/knowledge owns legal material/process discovery semantics.
- Journal owns persistent diagram/gear knowledge and silhouettes.
- Survival consumes derived Vigor/noise modifiers where relevant.
- Combat consumes derived weapon/armour profiles.
- Run lifecycle preserves persistent gear on death and must not repair it.

## 7. Definition of done

Do not expand to the full gear catalogue until:

- `GEAR-TEST-VECTORS.md` passes for the v1 subset;
- persistent gear identity is proven through death/restart;
- upgrade/repair transactions are idempotent;
- Broken is recoverable and never deletes the item;
- hidden quality cannot be rerolled or leaked;
- branch/diagram gating cannot be bypassed;
- equip/unequip cannot bypass bulk;
- no known race can duplicate gear or materials.