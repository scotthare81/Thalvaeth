# Gear + weapons + durability — coding plan

Implement in small reviewable PRs after the documentation contract is accepted. Each stage should keep the server authoritative and leave the repository runnable.

## PR 1 — persistent gear identity foundation

- SQL migration for gear-instance metadata;
- stable definition registry skeleton;
- bridge persistent instance to equipped WotLK item GUID/template;
- load/save/restart reconstruction;
- GM `gear dump` and guarded durability setters;
- fixture for starting Crude Dagger + Rag Armour.

Exit: same gear identity/quality/durability survives relog/restart with no duplication.

## PR 2 — centralized durability

- integer current/max durability;
- Fine/Worn/Damaged/Broken derivation;
- centralized wear service + event idempotency;
- weapon combat wear;
- dagger butcher/tool wear;
- armour body wear;
- derived condition effect hook;
- Broken persistence.

Exit: deterministic D-series tests pass.

## PR 3 — sharpening + structural repair

- Whetstone field action;
- repair definitions;
- Forge/Stitch/Workbench station validation using canonical `station.*` keys;
- Inventory reservations/material consumption;
- duplicate-request protection;
- no max-durability erosion v1.

Exit: E/F tests pass including Broken recovery.

## PR 4 — upgrade graph transaction

- upgrade-node registry/validation;
- source→target validation;
- process/diagram/station gates;
- material reservation/atomic consumption;
- one-time hidden-quality resolution;
- persistent node mutation;
- audit/reconciliation state.

First nodes: Crude Dagger → one pre-forge blade → Iron Knife/Short Blade → Steel Blade.

Exit: B/C/J transaction tests pass.

## PR 5 — first weapon branch

- one fast child from Steel Blade;
- one heavy child from Steel Blade;
- explicit butcher/chop capabilities;
- Vigor/noise/reach/stagger/bleed plumbing as data hooks;
- bulk preflight for carried/spare forms;
- diagram-required branch proof.

Exit: branch cannot be skipped/leaked and capabilities are explicit.

## PR 6 — armour body proof

- Rag → Quilted → Boiled Leather;
- armour derived profile;
- noise/Vigor/protection integration hooks;
- underlayer requirement plumbing;
- wear/repair/quality persistence.

Exit: H-series tests pass for body slice.

## PR 7 — Journal + ThalvaethUI gear presentation

- Gear Journal nodes/silhouettes from server knowledge;
- condition presentation without hidden numeric quality leakage;
- known upgrade choices and station/material requirements;
- repair/sharpen result feedback;
- reload/full-sync reconstruction.

Exit: K-series leakage tests pass.

## PR 8 — crash/race hardening + v1 acceptance

- interrupted upgrade/repair reconciliation;
- upgrade vs move/equip race;
- repair vs wear ordering;
- death during gear transaction;
- restart fixtures;
- run complete `GEAR-TEST-VECTORS.md` subset;
- diagnostics cleanup.

Exit: v1 gear slice is safe enough for broad content authoring.

## After v1

Only after the foundation is proven:

- full fast/heavy weapon ladders;
- iron/steel/bronze side-grade tuning;
- all armour classes/slots;
- packs and advanced tools;
- charms;
- advanced repair consequences if desired;
- separate edge/structure condition if playtesting justifies it;
- polished station/quarters presentation.

## Before first code PR

Reconcile existing documentation contradictions that affect Gear:

1. exact canonical `station.*` keys between Crafting runtime/implementation/station docs;
2. failure-consumption vocabulary where gear experimentation touches Crafting;
3. Journal final state/protocol contract (PR #12 wins over older conceptual values);
4. Inventory risk semantics for spare gear carried into runs;
5. confirm Crude Butcher/butcher capability naming and registry keys.

Do not encode unresolved documentation contradictions into C++.