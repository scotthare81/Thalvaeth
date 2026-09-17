# Gear failure, race and recovery contract

Gear is persistent progression. A transaction bug that duplicates, deletes or rerolls it is therefore more severe than an ordinary content bug. This document defines failure handling before implementation.

## 1. Fundamental invariant

For every upgrade/repair/sharpen operation, recovery must converge to one coherent state:

- **not committed:** old gear state remains and unconsumed materials remain; or
- **committed:** new gear state exists exactly once and required materials were consumed exactly once.

Never leave the player with consumed materials and an uncommitted upgrade unless a deliberately authored destructive experimentation outcome caused it. Known gear work is not such an experiment.

## 2. Operation identity

Every mutating gear action should receive/create a server transaction ID. Repeated client packets/reconnect retries with the same logical operation must not create a second commit.

The server should be able to answer: “Was transaction T committed, rolled back, or unresolved?”

## 3. Reservation order

Recommended ordering:

1. reserve gear instance;
2. reserve required inventory inputs;
3. validate station/context/knowledge again under reservation;
4. preflight destination/equipment/bulk;
5. create durable prepared operation if needed;
6. compute one-time result;
7. commit inventory + gear mutation;
8. mark transaction committed;
9. release reservations;
10. emit presentation delta.

Do not roll hidden quality before the operation has crossed the point where cancellation/retry cannot be used to fish for another result.

## 4. Duplicate upgrade request

If request A reserves G42 and request B targets G42:

- B rejects as busy, or later sees G42 already at the new node;
- B cannot reserve a second ingredient set;
- if the client times out, it should request/resync state rather than blindly repeat as a new operation.

## 5. Upgrade vs equip/move

A reserved gear instance cannot move/equip/unequip while the mutation is prepared.

This prevents:

- source item disappearing from expected location;
- output bulk bypass;
- visible old template and new template both surviving;
- death finalization classifying the same item twice.

## 6. Upgrade vs death

Death finalization and gear mutation require deterministic ordering.

Because persistent worn gear survives death, a committed upgrade should remain committed even if death follows immediately. However, ingredients drawn from at-risk run inventory must not be double-classified or resurrected.

Recommended rule:

- run finalization freezes new inventory/gear mutations;
- an already prepared short gear transaction must either finish/reconcile before finalization consumes/deposits affected inventory, or be rolled back cleanly;
- death cannot delete the persistent gear instance itself.

V1 should normally restrict structural upgrading to Monastery stations, making upgrade-vs-death rare; still test the invariant.

## 7. Sharpen vs death/disconnect

Field sharpening can overlap run hazards.

If sharpening is channelled:

- reserve blade + whetstone at channel start;
- interruption before commit consumes nothing in v1 unless explicitly authored otherwise;
- successful completion commits restoration + whetstone use atomically;
- disconnect interrupts rather than grants free completion;
- death interrupts before run finalization.

If sharpening is instant, commit atomically in one server operation.

## 8. Repair vs wear

At home this should rarely overlap combat, but test it anyway. A gear-instance lock prevents simultaneous writes.

If future field structural repairs exist, define ordering by server commit/event sequence. Do not merge two stale durability values.

## 9. Template replacement failure

An upgrade may need to change the visible WotLK item template.

Safe sequence must preserve one persistent identity. If creating/swapping the visible representation fails:

- do not leave both old and new visible items;
- do not mark the persistent node committed unless reconciliation knows how to finish the bridge;
- retain enough durable state to retry/reconcile after restart.

Never “fix” uncertainty by granting a fresh copy.

## 10. Missing visible item

If DB says G42 exists but its expected WotLK item object is missing:

- flag reconciliation error;
- do not silently delete G42;
- do not automatically grant a replacement without verifying ownership/location/history;
- GM diagnostics should expose the mismatch.

Recovery tooling may reconstruct the visible carrier from authoritative gear metadata only through an explicit, auditable path.

## 11. Duplicate visible item

If two WotLK item GUIDs claim one persistent gear instance or equivalent persistent metadata:

- quarantine/disable mutation of the ambiguous copies;
- log loudly;
- do not allow both to be equipped/salvaged/upgraded;
- provide GM reconciliation.

## 12. Definition removed after patch

If a persisted node/material key no longer exists after a server update:

- mark gear invalid/unresolved;
- preserve DB row and visible ownership where safe;
- disable unsafe mutation/effects or fall back only to an explicitly authored compatibility profile;
- require migration mapping.

Never reset it to starter gear.

## 13. Quality corruption

If quality value is outside legal internal range:

- clamp only if the common quality contract explicitly allows safe clamping;
- otherwise quarantine/log for migration;
- never reroll randomly on login.

## 14. Durability corruption

If current > max, clamp current to max and log if this can be proven harmless. Negative/impossible max durability is a definition/data error and should not silently become a fresh item.

## 15. Station invalidation

If a station despawns/player moves away during a channelled repair/upgrade before commit, cancel and release reservations. If operation is already committed, moving away cannot undo it or refund materials.

## 16. Inventory capacity change

If destination capacity changes between initial UI preview and commit, authoritative preflight under reservation wins. Reject safely rather than overflow the bag or drop a persistent item into the world.

## 17. Server restart reconciliation

On startup/login, scan unresolved prepared gear transactions relevant to the character.

For each, determine from durable evidence whether commit happened. Reconcile to exactly one state. Never reroll quality during reconciliation.

A transaction should be safe to reconcile multiple times.

## 18. GM recovery tools

Diagnostics should support:

- inspect gear instance ↔ item GUID bridge;
- inspect last transactions;
- list unresolved prepared transactions;
- reconcile one transaction;
- identify duplicate bridge ownership;
- repair corrupted durability only through explicit GM action;
- never expose these controls to normal addon messages.

Every destructive GM recovery should log before/after state.

## 19. Test priority

Treat these as release-blocking for persistent Gear:

1. duplicate upgrade cannot duplicate/consume twice;
2. crash cannot erase persistent gear;
3. visible-template swap cannot leave two owned copies;
4. quality cannot be rerolled through retry/crash;
5. death cannot delete or repair persistent gear;
6. field sharpening cannot survive interruption incorrectly;
7. unresolved definitions never silently reset progression;
8. reconciliation is idempotent.