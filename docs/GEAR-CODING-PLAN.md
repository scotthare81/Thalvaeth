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

## Required design decisions before first code PR

These are explicit blockers rather than implementation choices:

1. **Canonical graph-node namespace.** Draft docs conceptually use both `gear.*` and `upgrade.*`; choose one persisted convention and normalize all Gear docs/fixtures.
2. **Canonical station keys.** Reconcile Crafting runtime/implementation/station docs and use one `station.*` registry.
3. **Paired weapons identity.** Decide whether paired/twin weapons are one logical persistent gear instance representing the set or two linked persistent instances. This materially affects repair, durability, equip slots and transactions.
4. **Quality inheritance.** Define how previous workmanship quality and new material quality contribute when the same persistent gear is substantially rebuilt. The result must be one-time/deterministic at commit.
5. **Spare gear risk.** Inventory/Run canon must explicitly say what happens to a persistent-quality spare weapon/armour piece carried in the main bag on death before such items are enabled.
6. **Butcher terminology.** Confirm Crude Butcher/Butcher Knife/skinning-knife capability and stable registry naming.
7. **Diagram policy.** Pin which early transitions are discoverable and which branch/tier transitions are diagram-only.
8. **Repair channel policy.** Decide whether field sharpening is instant or channelled in v1; structural repairs remain home/station work.
9. **Condition presentation.** Confirm normal UI uses qualitative bands only in v1 rather than exact durability percentage.
10. **Dual-wield combat bridge.** Verify how the WotLK chassis will represent the authored paired-weapon form before locking the persistent representation.

## Documentation contradictions to reconcile before code

1. Journal PR #12 final state/protocol contract wins over older conceptual Journal values/protocol.
2. Crafting failure-consumption vocabulary must be unified where gear experimentation touches Crafting.
3. Preserve Journal solution-matrix highest-unresolved-blocker behaviour; repeated same failure must not leak a lower clue.
4. Survival exact 0–10000 implementation model wins over older normalized runtime wording.
5. Inventory/run branch semantics must be merged/accepted or this stacked Gear branch retargeted before implementation begins.

Do not encode unresolved documentation contradictions into C++.