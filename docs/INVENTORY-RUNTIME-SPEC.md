# Inventory runtime and transaction contract

This is the lower-level runtime companion to `INVENTORY-IMPLEMENTATION-SPEC.md`.

## 1. Authoritative concepts

Runtime code should expose explicit concepts rather than scatter bag checks across scripts:

- `ContainerKind`: Worn, MainBag, Satchel, HomeStock;
- `RiskClass`: PersistentKit, RunHaul, RunConsumable, HomeStock;
- `BulkValue`: 1, 2, 4;
- `ItemProvenance`: run ID/source/timestamp;
- `InventoryReservation`: exact instance/quantity owner + expiry/transaction ID.

## 2. Service boundary

Suggested conceptual service:

- `CanPlace(item, qty, destination)`
- `GetUsedBulk(character, container)`
- `GetCapacity(character, container)`
- `ResolveAcquisitionDestination(item)`
- `MoveAtomic(...)`
- `Reserve(...)`
- `ReleaseReservation(...)`
- `CommitConsumption(...)`
- `TagRunProvenance(...)`
- `FinalizeExtraction(runId)`
- `FinalizeDeath(runId)`

Names are illustrative; centralization is mandatory.

## 3. Capacity arithmetic

All calculations use authoritative item definitions and integer quantities. UI-reported bulk is ignored.

For a normal stack: `stack_bulk = bulk_per_unit × quantity`.

Capacity check: `used_bulk - source_removed_bulk + destination_added_bulk <= capacity` for same-container transformations where applicable.

Worn gear contributes zero to carried bulk only while actually equipped in a permitted equipment slot.

## 4. Acquisition transaction

Loot/gather acquisition:

1. validate source and entitlement;
2. resolve item definition and quantity;
3. choose only legal destination;
4. preflight bulk/stack merge;
5. lock source + destination mutation scope;
6. transfer/create item once;
7. assign run provenance if active;
8. release source/lock;
9. emit client delta.

Failure before commit leaves source unclaimed where possible.

## 5. Move transaction

A move request contains only intent: source instance/quantity and destination. Server re-reads current state, validates allow-list/capacity/reservation, then commits atomically.

Stale client positions must fail safely rather than target a different item.

## 6. Reservation contract

Reservations are required for crafting and survival-use channels.

A reservation identifies:

- transaction ID;
- character;
- exact item instance/stack;
- quantity;
- purpose;
- created time;
- run ID if active.

Reserved quantity cannot be moved, consumed, traded, destroyed or reserved again by a competing transaction.

On reconnect/startup, abandoned transient reservations are reconciled and released unless a durable finalization transaction owns them.

## 7. Stack identity

Do not merge stacks when merging would erase:

- different hidden quality;
- different spoil expiry/state;
- different risk/provenance semantics relevant to finalization;
- active reservations.

Where two run-haul stacks from the same run and equivalent metadata merge, provenance remains that run.

## 8. Spoilage clock

Prefer absolute server timestamps or durable elapsed-run accounting over client timers. Moving/stack splitting cannot reset expiry. Splitting copies the original expiry to both resulting quantities.

## 9. Extraction inventory commit

Finalization operates from server state, not a client-submitted item list.

For each item present in the active run:

- PersistentKit: preserve;
- RunHaul: promote/deposit to extracted home ownership;
- RunConsumable remaining: follow its item definition; v1 defaults to extracted home ownership if it is a legitimate carried item;
- illegal/orphan metadata: quarantine/log rather than duplicate.

Mark promotion against run ID so retries are idempotent.

## 10. Death inventory commit

For each item present:

- PersistentKit: preserve;
- RunHaul: destroy/forfeit;
- RunConsumable: v1 at-risk, destroy/forfeit;
- HomeStock: must never be in the run deletion set.

Deletion set is materialized from authoritative state before commit and logged against run ID.

## 11. Error classes

Player-safe errors should distinguish only actionable categories:

- no room;
- wrong container;
- item busy;
- source changed;
- action unavailable.

Do not leak hidden quality, recipe identity, risk internals or server keys.

## 12. Concurrency tests

Must test:

- loot same node twice;
- split/move same stack twice;
- craft + move same ingredient;
- consume + craft same item;
- extraction while craft completes;
- death while loot commits;
- reconnect with stale reservation;
- repeated finalization callback.

At most one authoritative outcome may consume/create each quantity.