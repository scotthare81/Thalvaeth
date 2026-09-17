# Inventory + satchel implementation specification

This document turns the carrying/economy canon into a server-authoritative implementation contract.

Related canon: `ECONOMY.md`, `CONTENT.md`, `ITEMS.md`, `MATERIALS.md`, `CRAFTING-IMPLEMENTATION-SPEC.md`, `SURVIVAL-IMPLEMENTATION-SPEC.md`, `RUN-LIFECYCLE-SPEC.md`.

---

## 1. Core invariant

A Remnant has three materially different item domains:

1. **Worn/persistent equipment** — equipped gear; survives run death.
2. **Main bag** — carried finished goods, tools, spare gear, consumables and valuables; run-risk depends on item/run origin as defined below.
3. **Gather satchel** — raw-material haul; curated allow-list; always at risk during a run.

The server owns item location, capacity, eligibility and movement. The addon may render and request moves but cannot declare capacity or ownership.

## 2. Persistent kit versus haul

The game must not infer risk merely from WoW bag slot location.

Each relevant carried item instance has a server-resolvable **risk class**:

- `persistent_kit` — equipped starting/upgraded gear and explicitly protected persistent kit;
- `run_haul` — materials/items acquired during the active run;
- `run_consumable` — consumables deliberately brought into or produced during a run and designated at-risk;
- `home_stock` — Monastery storage only, never present as run inventory state.

For v1, equipped gear is persistent. The gather satchel contents are haul and lost on death. Items extracted successfully become home stock when deposited at home.

A later design may make selected carried consumables persistent or insured, but v1 must not invent insurance.

## 3. Container model

### Main bag

Carries:

- cooked food;
- water;
- medicine;
- tools;
- carried/spare weapons and armour;
- charms not equipped;
- valuables/finished goods.

### Gather satchel

Allow-list only:

- raw plant/fungal materials;
- raw creature harvests;
- raw/scavenged crafting materials;
- Raw Meat;
- ore/reclaimed metal inputs;
- other items explicitly tagged `satchel_allowed`.

Never allowed:

- finished food/drink;
- medicine;
- tools;
- finished gear;
- charms;
- recipe/diagram knowledge objects once consumed into discovery;
- arbitrary valuables unless explicitly classified as raw material.

## 4. Bulk model

Capacity is a server-checked integer bulk budget.

Canonical v1 item bulk values are **1, 2 or 4**.

- worn equipment consumes 0 carried bulk;
- main-bag items consume main-bag bulk;
- satchel items consume satchel bulk;
- stackable items consume `bulk_per_unit × quantity` unless an item definition explicitly uses stack-bulk rules;
- moving an item between legal containers never creates capacity;
- a move that would exceed destination capacity is rejected atomically.

The future footprint-grid UI must preserve the same semantic bulk values; it is presentation, not a new capacity authority.

## 5. Starting v1 capacities

Use centralized tuning constants rather than hard-coded UI assumptions.

Provisional first-play values:

- main bag capacity: **12 bulk**;
- gather satchel capacity: **16 bulk**;
- worn equipment: free;
- upgrades increase one pool at a time through explicit upgrade definitions.

These are testable starting values, not final balance.

## 6. Item routing

On acquisition, the server selects destination in this order:

1. if raw and `satchel_allowed`, try satchel;
2. otherwise try main bag;
3. if the required legal container has insufficient capacity, acquisition fails/remains in world according to source type;
4. never silently route a finished item into the satchel to bypass capacity.

For loot nodes/corpses, a capacity failure leaves the item unclaimed where the engine permits. For generated crafting outputs, capacity is preflighted before inputs commit.

## 7. Movement rules

Every move is validated for:

- source ownership;
- current run ID/character;
- destination legality;
- destination capacity;
- item lock/reservation state;
- craft/use transaction involvement;
- risk-class invariants.

A move is atomic. No intermediate duplicate may be visible to a second request.

## 8. Stacking

Stack merges must preserve:

- item definition;
- hidden quality where quality affects instance semantics;
- spoil state/timestamp where applicable;
- run provenance/risk class;
- reservation state.

Items with materially different spoil expiry or hidden quality must not merge if doing so would destroy gameplay state.

## 9. Raw Meat and spoilage

Raw Meat belongs in the satchel and has a server timestamp/expiry.

Required rules:

- spoilage advances during active run time;
- disconnect does not reset the timer;
- extraction/home transition preserves the item's remaining/derived state until it is cooked, rendered, bartered or otherwise resolved;
- spoiled meat cannot be restored by moving containers;
- stack merging cannot extend expiry;
- preservation recipes create a new finished item with its own shelf-life rules.

Exact timers remain tuning constants.

## 10. Crafting integration

Crafting may source legal ingredients from both main bag and satchel.

The craft engine must:

1. resolve required inputs;
2. reserve exact item instances/quantities;
3. preflight output destination and bulk;
4. evaluate craft;
5. commit consumption/output atomically;
6. release reservations on cancellation/failure according to recipe failure policy.

Inventory never independently decides a crafting result.

## 11. Survival-use integration

Eat/drink/treatment channels reserve exactly one intended item/quantity. On interruption, the reservation releases unless the use definition explicitly consumes at channel start. V1 survival items consume on successful channel completion.

## 12. Run provenance

Every at-risk acquisition during a run must be attributable to the active `run_id` until extraction/death finalizes it.

This provenance exists to make extraction/death idempotent and auditable; it is not necessarily a player-facing property.

Required fields conceptually:

- character GUID;
- run ID;
- item instance/stack identity;
- risk class;
- source category;
- acquired timestamp.

Implementation may store provenance in dedicated tables/metadata rather than altering AzerothCore item schema if that is safer.

## 13. Home storage boundary

Extraction commits haul into the Monastery/home economy. V1 may use existing bank/storage plumbing or a dedicated virtual stash, but the semantic contract is:

- home stock is not part of an active run;
- death cannot delete already-extracted home stock;
- starting a new run only marks items deliberately carried into that run as run-present;
- bulk checks for run containers are independent from home storage capacity.

The exact home-storage UI can be implemented later; backend semantics must exist first.

## 14. Disconnect/crash rules

Disconnect is neither death nor extraction.

During an active run:

- item positions and provenance persist;
- reservations are safely cancelled/reconciled;
- reconnect restores the same authoritative carried state;
- no item is promoted to home stock;
- no run haul is deleted merely because the client vanished.

Server crash recovery follows the same principle using persisted run state.

## 15. Anti-duplication invariants

At all times:

- one item instance has one authoritative location;
- one quantity cannot be simultaneously reserved by two transactions;
- extraction is idempotent by run ID;
- death cleanup is idempotent by run ID;
- a run cannot finalize as both extracted and dead;
- reconnect cannot replay a completed craft, loot or extraction transaction;
- capacity failure never consumes inputs without an explicitly documented destructive failure.

## 16. Client/UI contract

The addon may display:

- current/max bulk for main bag and satchel;
- legal destination cues;
- spoil state in player-facing qualitative terms;
- item risk messaging where useful.

It must not:

- decide satchel eligibility;
- calculate authoritative free capacity;
- change risk class;
- mark an item extracted;
- delete haul on death.

## 17. Diagnostics

Developer tooling should expose:

- `.thal inv dump`
- `.thal inv bulk`
- `.thal inv provenance`
- `.thal inv move <instance> <container>`
- `.thal run items`

Names are illustrative. Production commands must be permission-gated.

## 18. Acceptance criteria

Inventory foundation is ready when:

1. satchel allow-list is server-enforced;
2. bulk cannot be bypassed by moving/stacking;
3. worn gear is free;
4. crafting preflights output capacity;
5. duplicate requests cannot double-spend a stack;
6. active-run reconnect preserves items/provenance;
7. extraction and death can finalize inventory exactly once;
8. hidden quality/spoil metadata survives legal moves;
9. no client message can create, protect or extract an item.