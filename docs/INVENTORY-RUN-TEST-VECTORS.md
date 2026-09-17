# Inventory + run lifecycle deterministic test vectors

These vectors are mandatory for the first implementation.

## A. Capacity and routing

### A1 — raw gather routes to satchel
Given active run, Ashbloom bulk 1, satchel has room. Gather one. Expect satchel +1 bulk, run provenance assigned, main bag unchanged.

### A2 — finished item cannot use satchel
Attempt to move Ash Tea into satchel. Expect rejection, no mutation.

### A3 — satchel full
Satchel used 16/16. Gather bulk-1 raw material. Expect no acquisition/overflow and source remains available where possible.

### A4 — main bag bulk
Main bag 10/12. Acquire bulk-2 legal item: success to 12/12. Next bulk-1 finished item fails.

### A5 — worn gear free
Equip carried bulk-2 legal armour from main bag. Expect main-bag used bulk decreases by 2 and worn contribution is zero. Unequip requires 2 free main-bag bulk or fails.

### A6 — illegal unequip when bag full
With no free capacity, attempt to unequip bulk-2 worn item. Expect rejection; gear stays equipped.

## B. Stacks and metadata

### B1 — spoil split
Split Raw Meat stack with expiry T. Both resulting stacks retain expiry T.

### B2 — no expiry laundering
Try merge Raw Meat expiring T1 with T2. Expect no metadata-destroying merge; expiry cannot become later for T1 quantity.

### B3 — quality separation
Two otherwise identical crafted items with materially different hidden quality do not merge if quality affects effects/state.

### B4 — provenance preserved
Split run-haul stack then recombine equivalent halves. Both remain associated with same run ID.

## C. Reservations/concurrency

### C1 — craft versus move
Craft reserves final Ashbloom. Simultaneous move request for it fails `item busy`. Craft may commit once.

### C2 — consume versus craft
Drink/use channel reserves an item. Craft cannot reserve same quantity.

### C3 — duplicate loot
Two requests target same one-use gather source. At most one item is created.

### C4 — output capacity preflight
Craft output needs bulk 2 but destination has 1 free. Expect craft does not commit and ingredients remain.

## D. Run entry

### D1 — valid start
No active run. Entry creates one run ID, initializes Survival, validates inventory, phases/ports player, marks ACTIVE.

### D2 — duplicate start
Second start request while ACTIVE does not create another run.

### D3 — failed entry
Teleport/phase setup fails before ACTIVE. PREPARING state rolls back; no item loss and no phantom active run.

## E. Extraction

### E1 — valid extraction
ACTIVE run, gate unlocked, player alive/in range. Expect one FINALIZING transition, haul promoted/deposited, Survival reset, home teleport, terminal EXTRACTED.

### E2 — locked gate
Extraction request before conditions met. Expect no lifecycle/inventory mutation.

### E3 — duplicate extraction request
Send request twice. Exactly one finalization occurs; no duplicate home stock.

### E4 — replay after extraction
Reconnect/replay old request for terminal run. Expect no item creation and state remains EXTRACTED.

### E5 — crafting during extraction
If craft is merely reserved/channeling when finalization starts, it is reconciled/cancelled according to policy before haul commit. It cannot create output after the same inputs were promoted.

## F. Death

### F1 — normal run death
Player has persistent equipped dagger/armour, satchel raw haul, main-bag run consumables and extracted home stock. Expect run haul/at-risk consumables forfeited, equipped persistent gear retained, home stock untouched, Journal/discovery untouched, return to morgue, terminal DEAD.

### F2 — duplicate death callback
Death finalization called twice. Second call deletes nothing additional.

### F3 — death with reserved craft inputs
Reservations reconciled before deletion classification; no ghost reservation or later craft completion.

### F4 — no corpse recovery
After morgue return, lost haul is not recoverable from a player corpse in Rotwood.

## G. Death/extraction race

### G1 — death first
Lethal event obtains finalization guard before extraction validation. Outcome DEAD; no extracted haul.

### G2 — extraction first and valid
Extraction obtains guard while player alive and begins FINALIZING. Later duplicate death callback cannot re-finalize inventory.

### G3 — already dead extraction
Extraction validation sees dead player. Reject; death path owns terminal transition.

## H. Disconnect/restart

### H1 — disconnect active
Disconnect with 70% satchel, Hunger/Thirst pressure and Ash Hollow used. Reconnect restores same run state; no extraction/reset.

### H2 — disconnect during channel
Transient item-use reservation is cancelled/reconciled; item is not duplicated or consumed twice.

### H3 — restart active
Server restart then login. ACTIVE run resumes according to safe-position policy with same haul/provenance.

### H4 — restart during extraction finalization
Recovery reads durable finalization state and completes/reconciles once. No duplicated extracted stack.

### H5 — restart during death finalization
Recovery completes/reconciles loss once. Persistent kit remains.

## I. Escape exploits

### I1 — hearth blocked
Attempt ordinary hearth while ACTIVE. No home teleport/extraction.

### I2 — unstuck cannot pay out
Any allowed recovery path cannot mark haul extracted.

### I3 — GM teleport
GM moves active-run player home. Lifecycle remains non-extracted until explicit admin reconciliation; geography alone does not pay out.

## J. Spoilage

### J1 — disconnect does not refresh meat
Raw Meat expiry remains based on server state; reconnect does not restart timer.

### J2 — container move does not refresh
Move Raw Meat satchel→legal processing context→back where allowed. Expiry unchanged.

### J3 — preservation creates new item
Successful preservation consumes raw input and creates finished preserved item with its own shelf-life definition; it does not edit raw expiry to fake preservation.

## K. Character isolation

Character A active run/provenance cannot be finalized, resumed or claimed by character B on same account.

## L. Definition of done

All vectors must be reproducible with developer tooling and leave an auditable run/inventory state. Failures must never depend on trusting addon state.