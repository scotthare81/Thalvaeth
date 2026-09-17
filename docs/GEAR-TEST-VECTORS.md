# Gear, durability and upgrading — deterministic test vectors

These tests are acceptance contracts. Exact balance coefficients can change without invalidating the behavioural expectations.

## A. Persistent identity

### A1 — starting dagger survives relog
Given the character owns persistent `Crude Dagger` instance G1 with fixed quality/durability, relog and restart.

Expected: same gear instance identity, node, quality and durability. No reroll and no duplicate visible item.

### A2 — death preserves condition
Given G1 is Damaged before a run death.

Expected: after waking in Monastery quarters G1 is still Damaged at the same durability value. Death neither repairs nor deletes it.

## B. Upgrade graph

### B1 — valid edge
Given current node can transition to target, knowledge/station/materials are valid.

Expected: exact materials consumed once; same persistent gear instance advances to target; result persisted.

### B2 — skip edge
Attempt `Crude Dagger → Steel Blade` directly when graph requires intermediate nodes.

Expected: reject invalid transition; consume nothing; gear unchanged.

### B3 — wrong station
Attempt valid metal upgrade away from Forge.

Expected: reject; no consumption/no mutation.

### B4 — missing process gate
Attempt Iron transition without required Smelting/Forge knowledge.

Expected: reject without revealing undiscovered downstream content.

### B5 — missing diagram
Attempt a `diagram_required` branch transition without diagram knowledge.

Expected: reject; no brute-force unlock; no hidden node-name leakage if still unknown.

### B6 — duplicate click
Submit same valid upgrade request twice rapidly.

Expected: one commit only. Second request sees reservation/changed node/transaction result and cannot consume a second material set.

## C. Hidden quality

### C1 — quality rolls once
Perform valid quality-sensitive upgrade.

Expected: server commits one hidden result. UI close/reopen, relog and restart preserve it.

### C2 — cancel before commit
Reserve materials then interrupt before commit.

Expected: no quality result is farmable/revealed; reservation released according to transaction rules; inputs remain unless an explicit destructive rule applies.

### C3 — no client leak
Inspect normal addon payload/tooltip path.

Expected: no raw hidden quality value, RNG seed or balance coefficient is transmitted.

## D. Durability

### D1 — deterministic wear
Apply authored wear events with known magnitudes to Fine gear.

Expected: integer durability decreases exactly once per event and condition band derives from configured threshold.

### D2 — replayed wear event
Submit same event ID twice.

Expected: no double wear.

### D3 — band transitions
Drive gear across Fine → Worn → Damaged → Broken thresholds.

Expected: each band derives correctly; zero clamps at Broken; item remains owned.

### D4 — Broken persistence
Relog/restart with Broken gear.

Expected: remains Broken, not deleted/replaced/repaired.

### D5 — Broken effective profile
Query effective combat/tool/armour profile at Broken.

Expected: severe authored penalty/disabled advanced capability applies, while item remains repairable.

## E. Sharpening

### E1 — valid field sharpen
Given eligible worn blade + valid whetstone in permitted field context.

Expected: whetstone/resource consumed once; restoration bounded by sharpening rule.

### E2 — sharpen non-blade
Attempt whetstone on armour/non-sharpenable tool.

Expected: reject, consume nothing.

### E3 — Broken structural item
Attempt ordinary field sharpening on structurally Broken blade where v1 rule requires station repair.

Expected: reject or restore only the explicitly permitted edge portion; must not silently produce full structural repair.

## F. Repair

### F1 — Forge repair
Repair Damaged metal weapon at correct Forge with exact materials.

Expected: materials consumed once; durability restored to authored cap; quality/node unchanged.

### F2 — wrong repair family
Attempt metal structural repair at Stitch Table.

Expected: reject, consume nothing.

### F3 — repair Fine item
Attempt repair at/near full condition.

Expected: reject/no-op without consuming materials.

### F4 — repeated repairs
Repair same item repeatedly across play sessions.

Expected v1: maximum durability does not permanently erode.

## G. Combat/tool integration

### G1 — one strike one wear event
One successful combat strike traverses multiple internal damage hooks.

Expected: weapon wear applied once for the authored strike event.

### G2 — dagger butcher capability
Use dagger node with butcher capability.

Expected: butchering system accepts capability and applies appropriate tool wear.

### G3 — greatsword butcher rejection
Use pure greatsword without butcher capability.

Expected: butchering cannot infer capability from weapon category/name.

### G4 — greataxe chop capability
Use authored greataxe with chop capability.

Expected: woodcut action allowed and applies authored wear.

## H. Armour

### H1 — armour wear from hit
Receive one qualifying physical hit.

Expected: appropriate worn armour receives deterministic wear once according to policy.

### H2 — mixed armour
Wear heavy body + lighter head/feet.

Expected: total effective noise/Vigor/protection derives from actual pieces; system does not silently coerce all pieces to body class.

### H3 — layering
Upgrade/use mail/plate path requiring quilted underlayer.

Expected: underlayer requirement validated without accidental deletion/duplication.

## I. Inventory interaction

### I1 — worn free
Equip persistent weapon/armour.

Expected: worn item contributes zero main-bag bulk.

### I2 — unequip with full bag
Main bag has insufficient capacity for the item's authored carried bulk.

Expected: unequip fails atomically; item remains equipped.

### I3 — form change bulk preflight
Upgrade carried/spare gear into a form with greater bulk while destination lacks capacity.

Expected: reject before material consumption unless result can legally remain equipped/in place under the transaction.

## J. Concurrency/crash

### J1 — upgrade vs move
Gear reserved for upgrade; simultaneous move request arrives.

Expected: one wins according to reservation ordering; never duplicate/lose gear.

### J2 — repair vs wear
Repair transaction overlaps a combat wear event.

Expected: deterministic lock/order; final durability corresponds to exactly one repair and one wear in a documented order.

### J3 — crash during upgrade
Crash after durable transaction intent but before all visible mutation steps complete.

Expected: reconciliation produces either pre-upgrade state with materials intact or one committed upgraded state with materials consumed — never both/neither inconsistently.

### J4 — crash during repair
Same principle as J3.

## K. Journal/presentation

### K1 — unknown branch
Player has not learned heavy branch diagram.

Expected: Journal/UI retains silhouette/obscured state and does not transmit hidden branch details.

### K2 — learned diagram
Grant diagram through Journal authority.

Expected: legal branch becomes known after sync/reload; upgrade UI can present it if other requirements are understood.

### K3 — material possession is not knowledge
Acquire late material before diagram/process knowledge.

Expected: material can be owned without automatically revealing the upgrade.

## Definition of done

The first gear implementation slice is not complete until A–K pass and debug output can prove gear instance identity, node, hidden quality, durability, reservations and last transaction without exposing those hidden values to normal clients.