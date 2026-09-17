# Equipment contract

This document locks how persistent Gear maps onto player equipment, weapon sets, tools and carried spares. It is an implementation contract for the Gear package.

## 1. Core rule

A player's equipped loadout is a set of **logical persistent gear instances**. The WotLK equipment slots and visible item objects are presentation/carrier mechanisms; they do not define Thal'vaeth ownership or progression.

The server owns:

- which logical gear instance is equipped;
- which logical slot/loadout role it occupies;
- which visible WotLK item carrier(s) represent it;
- whether the gear can be equipped/unequipped/swapped;
- durability, quality, upgrade node and capabilities;
- carried bulk when not worn/wielded;
- persistence through death/relog/restart.

## 2. Canonical dual-wield decision

**Paired Daggers, paired Shortswords and Twin Blades are one logical persistent gear instance representing a matched weapon set.**

They are **not** two independently progressing weapons.

The matched pair has:

- one `gear_instance_id`;
- one upgrade-node key;
- one material profile unless a later authored node explicitly supports mixed materials;
- one hidden quality result;
- one durability pool in v1;
- one repair/sharpen transaction target;
- one persistent ownership record;
- one carried bulk definition for the complete set;
- one Journal gear entry.

The client may require two visible item carriers to render/equip both hands. Those carriers are children/projections of the single logical instance and cannot exist as independently owned gear.

## 3. Why matched pairs are one object

This preserves the intended progression fantasy: the player chooses a **fast weapon build**, not two separate item-maintenance projects.

It also prevents:

- one dagger having unrelated hidden quality from its mate;
- one half of the pair being upgraded while the other is left behind;
- two repair transactions for one combat style;
- losing or duplicating one half during template swaps/relog;
- awkward branch progression requiring two copies of every diagram/material gate;
- dual-wield receiving twice the persistent-upgrade opportunities of a two-handed build.

The pair can still animate alternating hands and combat code can distinguish left/right strike presentation without creating separate progression objects.

## 4. Paired weapon carrier invariant

For a logical paired weapon G:

```text
G = authoritative persistent instance
  ├─ main-hand carrier (presentation/equipment)
  └─ off-hand carrier (presentation/equipment)
```

Carrier invariants:

1. both carriers resolve to the same G;
2. neither carrier may be independently traded, destroyed, repaired, upgraded or persisted as a second Gear instance;
3. if one carrier is missing/corrupt on login, reconstruction repairs the presentation from G;
4. if duplicate carriers exist, reconciliation selects the authoritative expected set and removes/quarantines extras safely;
5. deleting a carrier must never delete G;
6. upgrading G may replace both visible templates atomically without changing `gear_instance_id`.

## 5. Paired durability

V1 uses one durability pool for the matched pair.

Combat may visually alternate hands, but each qualifying attack emits wear against the pair's logical instance. The wear definition for a paired weapon is tuned for its faster attack cadence so it does not degrade twice as fast merely because two models are displayed.

At Broken, the **pair** is Broken. We do not model one snapped dagger and one healthy dagger in v1.

A future per-hand condition system would be a deliberate schema/gameplay expansion, not an accidental consequence of WotLK's two weapon slots.

## 6. Paired sharpening and repair

Sharpening/repair targets the logical pair.

A matched pair may require more repair material or whetstone wear than a single dagger; that is authored cost data. The player performs one action and receives one resulting condition state.

The system must not allow sharpening the main-hand carrier and off-hand carrier separately.

## 7. Weapon equipment roles

Canonical logical weapon roles:

- `weapon.single` — dagger, knife, one-handed early blade;
- `weapon.paired` — matched fast-style set occupying both combat hands;
- `weapon.heavy` — two-handed greatblade/greataxe/cleaver occupying the combat weapon role;
- `tool.skinning` — dedicated harvest knife where required;
- future dedicated utility tools use explicit roles/capabilities rather than weapon-slot inference.

The exact mapping to WotLK main-hand/off-hand/ranged/bag/etc. slots is an adapter detail.

## 8. Heavy weapon rule

A heavy weapon is one logical persistent gear instance. While equipped it is worn/wielded and contributes zero main-bag bulk under the Inventory contract.

When unequipped/carried, its authored bulk applies. Current design intent is bulk 4 for two-handed weapons.

A heavy weapon does not automatically grant butchering because it occupies a weapon slot. Capabilities are explicit:

- Greatsword: combat only by default;
- Greataxe: may grant chopping;
- War Cleaver: may grant heavy butchering;
- dedicated skinning knife: grants reliable butchering/skinning.

## 9. Dedicated skinning knife

Heavy builds retain the survival-economy cost of needing a dedicated skinning/butchering knife.

The knife is a persistent tool/gear instance if it is part of the player's retained kit. It does **not** need to remain visibly equipped in an off-hand while the heavy weapon is in use.

For v1, tool actions may resolve an eligible knife from the player's legal equipped/tool-access state rather than requiring a permanent visible off-hand slot.

This avoids forcing the WotLK paper-doll model to dictate the design.

Exact quick-access/tool-slot presentation can be decided with ThalvaethUI; the server capability contract remains stable.

## 10. Weapon switching

Switching between weapon configurations is a server-validated inventory/equipment mutation.

A switch must validate:

- ownership;
- active run state;
- item not reserved/busy;
- target loadout compatibility;
- destination bulk for the weapon being stowed;
- no prohibited channel/action state;
- carrier reconstruction can complete safely.

No switch may silently delete or overflow the stowed weapon.

If the main bag cannot hold the weapon being unequipped, the switch fails and the current weapon remains equipped.

## 11. Mid-combat switching

The equipment contract supports switching in a run, but combat pacing rules may impose a channel/lock/cooldown later.

V1 implementation must not make swapping an instantaneous exploit merely because WotLK permits equipment changes. Combat design owns the final swap time/lock rule.

The authoritative equipment transaction should therefore support `pending → commit/cancel` rather than assuming every switch is immediate.

## 12. Broken weapon during a run

Broken does not force automatic unequip and does not conjure a replacement.

If the active weapon becomes Broken:

- it remains the same equipped persistent instance;
- its Broken combat/capability profile applies;
- the player may continue using it at severe disadvantage where allowed;
- field sharpening cannot bypass required structural repair;
- the player may switch to a carried spare/tool only if normal switching and bulk rules permit;
- death preserves the Broken state.

This creates a genuine run decision: press on with compromised kit, use a legal spare, or extract.

## 13. Spare weapons

A weapon not currently worn/wielded is carried inventory and uses authored bulk.

Current bulk intent:

- dagger/small knife: 1;
- one-handed weapon: 2;
- paired weapon set: authored as a complete set, normally 2 unless balance changes it;
- heavy/two-handed weapon: 4.

These are balance values and must come from data.

**Risk on death for carried spare persistent gear remains governed by the Inventory risk-class contract.** Do not infer loss merely because it is unequipped. This remains an explicit cross-document decision before implementation if PR #14 does not already settle it.

## 14. Armour slots

Canonical logical armour/equipment roles:

- Head — cowl/hood/head protection;
- Body — dominant armour body piece;
- Shirt — warmth under-layer;
- Tabard — warmth/weather outer wrap;
- Neck — scarf/muffler/warmth accessory;
- Hands — wraps/gloves;
- Legs/Waist — waders/leg protection/carry utility;
- Feet — boots;
- Back — pack;
- Weapon — single/paired/heavy logical weapon;
- Utility/Tool — contextual tool access;
- Charm 1;
- Charm 2.

These are Thal'vaeth logical roles. They may map onto different native WotLK equipment slots internally.

## 15. Body armour class

The body piece establishes the player's dominant armour-class presentation/build identity, but other pieces retain their own authored effects.

Mixing is legal unless an explicit definition forbids it.

The server calculates noise, Vigor burden, mitigation and environmental effects from actual worn pieces. It must not automatically turn every slot into the body's class.

## 16. Armour layering

Quilted/gambeson underlayers are persistent gear, not disposable ingredients by default.

Mail/plate requirements may declare a required underlayer. The equipment resolver verifies that required layer is present/valid.

A plate upgrade cannot consume the player's quilted underlayer merely to make the native item-slot model simpler.

If two logical layers map to one visible WotLK slot, the bridge must preserve both logical identities and choose an authored combined visual carrier.

## 17. Warmth layer

Shirt, tabard and neck remain independent of armour class and are reserved for warmth/environmental gear as established in `GEAR.md`.

They may stack authored warmth/weather effects. Future heat/cold mechanics consume the derived equipment profile; they do not redefine ownership.

These pieces persist through death under normal persistent-kit rules.

## 18. Pack equipment

The Back/Pack gear instance controls authored carrying-capacity modifiers.

Rules:

- capacity is server-owned;
- changing the visible backpack model alone cannot change capacity;
- unequipping/downgrading a pack requires the resulting inventory to fit;
- if the lower capacity would overflow current contents, the mutation fails atomically;
- upgrading a pack changes capacity only when the authoritative gear transaction commits;
- pack durability, if enabled, must not silently delete inventory or reduce capacity in a way that strands items without an explicit overflow policy.

V1 recommendation: pack condition may affect secondary handling/noise later, but **does not reduce hard capacity dynamically**. Capacity remains stable until an explicit pack upgrade/downgrade.

## 19. Charm slots

V1 reserves two logical Charm slots, consistent with existing Gear/Aptitude design.

A charm is a persistent equipped gear/aptitude object and worn gear contributes zero carried bulk. Swapping a charm is an equipment mutation and may be restricted during an active run by later Aptitude rules.

The Gear system only owns slot occupancy/persistence. Aptitude/Charm systems own the effect semantics.

## 20. Equip transaction

Canonical equip/swap transaction:

1. receive intent with logical gear instance and target role/loadout;
2. validate owner and current run/equipment state;
3. acquire reservation/lock on involved gear instances;
4. determine all gear being displaced;
5. preflight legal inventory destination and resulting bulk/capacity;
6. validate role/class/layer dependencies;
7. prepare carrier/template changes;
8. atomically commit logical equipment state + inventory locations;
9. reconstruct/update visible carriers;
10. recompute derived gear profile;
11. emit safe UI state;
12. release reservations.

Presentation failure after authoritative commit triggers carrier reconstruction; it must not roll back into duplicated ownership.

## 21. Unequip transaction

Unequip follows the same atomic rules. The player cannot drag a worn bulk-4 heavy weapon into a full main bag and create overflow.

If destination validation fails, the gear remains equipped and no authoritative state changes.

## 22. Death interaction

Death does not alter the logical equipped loadout except where the Inventory risk contract explicitly says an at-risk carried item is lost.

Persistent worn gear:

- retains gear instance ID;
- retains upgrade node;
- retains hidden quality;
- retains durability/condition;
- is reconstructed on wake/reconnect if visible carriers are missing;
- is not repaired by death.

The visible bags dropped beside the corpse belong to the run-loss presentation contract and must not imply that persistent worn weapons/armour were dropped with the haul.

## 23. Run entry and extraction

Run entry validates the complete equipped loadout and carried bulk before ACTIVE.

Extraction preserves the authoritative equipment state and promotes/deposits haul according to Inventory/Run rules. It does not reroll, repair or otherwise mutate persistent gear merely because the run succeeded.

## 24. Reconnect/restart

On reconnect/restart:

1. load logical gear instances;
2. load authoritative equipment-role assignments;
3. reconcile interrupted equip/upgrade/repair transactions;
4. validate inventory capacity;
5. rebuild expected WotLK carrier objects/visuals;
6. remove/quarantine duplicate presentation carriers;
7. recompute derived equipment profile;
8. only then expose equipment UI/control.

Never infer authoritative equipped state solely from whatever native item happened to survive in a WotLK slot.

## 25. V1 acceptance

Before combat implementation depends heavily on equipment, tests must prove:

1. paired weapons are one logical instance with two safe carriers;
2. deleting/reconstructing one paired carrier does not lose/duplicate the weapon set;
3. pair durability/quality/upgrade state is singular and persistent;
4. heavy weapon + skinning knife capability works without impossible slot requirements;
5. Broken weapon remains equipped/persistent;
6. weapon swap fails safely if the displaced weapon cannot fit in the bag;
7. armour layering preserves both logical pieces;
8. mixed armour derives effects from actual pieces;
9. pack downgrade cannot create overflow;
10. death preserves worn loadout exactly and does not repair it;
11. restart reconstructs logical equipment independently of native carrier corruption;
12. no addon/native-slot action can bypass server authority.