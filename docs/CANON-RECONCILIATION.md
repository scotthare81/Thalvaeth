# Canon reconciliation register

This document is the single cleanup register for known cross-document contradictions. New code must follow the **resolved** value below.

## Resolved

### Survival numeric representation
Canonical runtime representation: integer **0–10000** for Hunger, Thirst, Infection and Vigor where applicable. Any 0–100 wording is conceptual/display shorthand only.

### Journal discovery states
Canonical generic states:
- 0 absent/unknown
- 1 inferred
- 2 known
- 3 observed/deep-known

Old 0/10/20/30/40/50 values are superseded.

### Journal protocol
Canonical v1 prefix `THALVAETH`, separator `~`, handshake/snapshot `HELLO~1`, `READY~1`, `SNAP_BEGIN`, records, `SNAP_END`; record families use final `DISC`/creature contracts. Old `JOURNAL_BEGIN/JOURNAL_END` and old `DISCOVERY/HINT` wire forms are superseded.

### Station keys
Canonical persisted/runtime station identifiers use `station.*` stable keys. Bare display names are presentation only.

### Failure precedence
Crafting experimentation reveals at most one new persistent hint per attempt: the **highest unresolved blocker** under the canonical evaluation order. Repeating the same mistake does not reveal lower blockers.

### Death recovery
No morgue. Stable backend destination `home_recovery_spawn`; physical location is the Remnant's personal quarters/bed.

### Run death
Death terminates and destroys the entire run instance/state. No corpse-return run.

### Paired weapons
Paired Daggers, paired Shortswords and Twin Blades are one logical persistent gear instance with one v1 durability pool and potentially two WotLK presentation carriers.

### Sound/detection
Unified semantic NoiseEvent system is authoritative. Sound supplies evidence position, never hidden live target coordinates.

## Canonical mappings requiring implementation normalization

### Crafting consumption vocabulary
Journal-facing experiment outcomes and Crafting runtime consumption terms must map through one explicit enum/adapter. Runtime code must not persist free-form strings. Final enum should preserve distinctions for none/partial/full/damaging/hazard/transform where actually required.

### Crude Butcher
Born-known Tier 0 includes the **Crude Butcher/basic butchery action** as established by Crafting design. Registry key/name must be verified before code.

## Still open — must be decided before dependent code

1. **Spare persistent gear on death:** equipped persistent gear survives; exact risk policy for unequipped persistent spare weapons/armour carried in main bag remains unresolved against the older “haul/satchel loss” statement.
2. **Major gear rebuild quality inheritance:** preserve/derive/reroll policy must be locked; no implementation may reroll opportunistically.
3. **Still Breath Vigor/channel policy:** behaviour locked, exact cost/timing not.
4. **Crouch v1:** only if technically honest on WotLK client.
5. **Extraction arrival physical location:** backend `home_extraction_arrival` locked; exact Monastery placement not.
6. **Repair/sharpen channel timing:** transactional semantics locked; timing balance open.
7. **Exact Combat numbers:** Vigor/timing/reach/damage/stagger/bleed/noise remain balance data.
8. **First reviewed creature perception/anatomy profiles:** generic contracts locked; final per-entry tuning still content review.
9. **Short-lived AI suspicion persistence over full server restart:** reconnect cannot clear; exact restart approximation remains runtime policy.

## Rule

When an older document conflicts with a resolved item here, this register plus the newest implementation contract wins until the older document is edited. Before implementation milestones, remove the stale wording rather than relying indefinitely on precedence notes.