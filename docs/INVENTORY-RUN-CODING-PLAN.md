# Inventory + run lifecycle coding plan

Implement in small reviewable PRs. Do not build the fancy inventory UI first.

## PR 1 — Run identity + persistence foundation

- migration for run lifecycle state;
- unique active-run constraint/guard;
- run service with PREPARING/ACTIVE/FINALIZING/terminal states;
- basic dump/reconcile debug commands;
- no item deletion/payout yet.

Gate: duplicate starts/reconnect state tests pass.

## PR 2 — Bulk + satchel authority

- item bulk definition registry;
- satchel allow-list;
- main/satchel capacity service;
- worn-free rule;
- server acquisition/move validation;
- provisional 12/16 tuning constants.

Gate: capacity/routing vectors A1–A6 pass.

## PR 3 — Run provenance + reservations

- provenance storage tied to run ID;
- inventory transaction IDs;
- craft/use reservations;
- reconnect cleanup of transient reservations;
- concurrency diagnostics.

Gate: B/C vectors pass and no double-spend under synthetic duplicate requests.

## PR 4 — Extraction transaction

- gate interaction calls lifecycle service;
- finalization guard;
- freeze/reconcile mutations;
- promote/deposit run haul exactly once;
- Survival reset + Monastery return;
- extraction ledger.

Gate: E vectors pass including replay/restart simulation.

## PR 5 — Death transaction + morgue

- death hook requests DEAD finalization;
- at-risk classification/deletion;
- persistent equipped gear preservation;
- controlled revive/Monastery morgue path;
- death ledger;
- duplicate callback protection.

Gate: F vectors pass.

## PR 6 — Race/crash hardening

- shared terminal guard;
- death/extraction race tests;
- FINALIZING reconciliation after restart;
- PREPARING orphan recovery;
- teleport/hearth bypass blocking;
- admin recovery commands.

Gate: G/H/I vectors pass.

## PR 7 — Spoilage plumbing

- Raw Meat absolute/durable expiry metadata;
- split/merge rules;
- qualitative UI state hook;
- preservation output creates new shelf-life state.

Gate: J vectors pass.

## PR 8 — Minimal ThalvaethUI inventory presentation

- main bag/satchel bulk display;
- legal-container messaging;
- item-busy/no-room errors;
- spoil-state presentation;
- no client authority.

The footprint grid remains future work.

## Cross-PR rules

Every PR must:

- remain character-isolated;
- use server-authoritative state;
- preserve Journal knowledge on death;
- avoid gold/vendor plumbing;
- avoid recipe-specific inventory hacks;
- add/extend deterministic tests before content expansion;
- never infer extraction from map position alone.

## Ready-for-next-system gate

Inventory/run lifecycle is considered code-ready once the documentation in this stack is merged. It is considered implementation-complete for v1 only after all test vectors pass, including restart and race tests.