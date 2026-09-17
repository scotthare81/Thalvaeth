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
- semantic combat-wear hook interface;
- dagger butcher/tool wear;
- armour body wear;
- derived condition effect hook;
- Broken persistence.

Exit: deterministic Gear durability tests pass. Full combat-generated wear is owned by the Combat implementation sequence.

## PR 3 — sharpening + structural repair

- Whetstone field action;
- repair definitions;
- Forge/Stitch/Workbench station validation using canonical `station.*` keys;
- Inventory reservations/material consumption;
- duplicate-request protection;
- no max-durability erosion v1.

Exit: sharpening/repair tests pass including Broken recovery.

## PR 4 — upgrade graph transaction

- upgrade-node registry/validation;
- source→target validation;
- process/diagram/station gates;
- material reservation/atomic consumption;
- one-time hidden-quality resolution;
- persistent node mutation;
- audit/reconciliation state.

First nodes: Crude Dagger → one pre-forge blade → Iron Knife/Short Blade → Steel Blade.

Exit: graph/quality/transaction tests pass.

## PR 5 — first weapon branch + equipment identity

- one fast child from Steel Blade;
- one heavy child from Steel Blade;
- **paired weapons as one logical persistent matched set** with two presentation carriers where required;
- explicit butcher/chop capabilities;
- Vigor/noise/reach/stagger/bleed fields exposed as Gear profile data for Combat;
- bulk preflight for carried/spare forms;
- diagram-required branch proof;
- equip/unequip reconstruction tests.

Exit: branch cannot be skipped/leaked; paired identity is singular; capabilities are explicit; weapon Equipment tests pass.

## PR 6 — armour body proof

- Rag → Quilted → Boiled Leather;
- armour derived profile;
- noise/Vigor/protection integration hooks;
- underlayer requirement plumbing;
- wear/repair/quality persistence.

Exit: armour Gear/Equipment tests pass for body slice.

## PR 7 — Journal + ThalvaethUI gear presentation

- Gear Journal nodes/silhouettes from server knowledge;
- condition presentation without hidden numeric quality leakage;
- known upgrade choices and station/material requirements;
- repair/sharpen result feedback;
- reload/full-sync reconstruction.

Exit: leakage tests pass.

## PR 8 — crash/race hardening + Gear/Equipment v1 acceptance

- interrupted upgrade/repair reconciliation;
- upgrade vs move/equip race;
- repair vs wear ordering interface;
- death during gear transaction;
- paired-carrier reconstruction/corruption cases;
- restart fixtures;
- run `GEAR-TEST-VECTORS.md` + `EQUIPMENT-TEST-VECTORS.md` v1 subset;
- diagnostics cleanup.

Exit: v1 Gear/Equipment foundation is safe enough for Combat and broad content authoring.

# Combat follows Gear/Equipment

The deep Combat package is now:

- `COMBAT-IMPLEMENTATION-SPEC.md`
- `COMBAT-RUNTIME-SPEC.md`
- `COMBAT-WEAPON-BEHAVIOUR.md`
- `COMBAT-CREATURE-REACTIONS.md`
- `COMBAT-TEST-VECTORS.md`
- `COMBAT-V1-SLICE.md`
- `COMBAT-CODING-PLAN.md`

Do **not** bolt full custom combat into early Gear PRs. Gear exposes persistent identity, effective profiles, condition and semantic wear hooks. Combat owns attack timing, Vigor commitment, reach/facing/hit resolution, stagger, bleed, cleave, noise, creature reactions and interruption.

After Gear/Equipment is stable, follow `COMBAT-CODING-PLAN.md` as its own small reviewable implementation sequence.

## After v1

Only after the foundations are proven:

- full fast/heavy weapon ladders;
- iron/steel/bronze side-grade tuning;
- all armour classes/slots;
- packs and advanced tools;
- charms;
- advanced repair consequences if desired;
- separate edge/structure condition if playtesting justifies it;
- polished station/quarters presentation;
- broader weapon movesets/creature attacks after Combat v1.

## Required decisions before first code PR

Closed:

- **Paired weapons identity:** one logical persistent matched Gear instance; WotLK hand objects are presentation carriers.

Still explicit blockers:

1. **Canonical graph-node namespace.** Normalize `gear.*`/`upgrade.*` persisted conventions.
2. **Canonical station keys.** Reconcile Crafting docs into one `station.*` registry.
3. **Quality inheritance.** Define previous workmanship vs new material quality across major rebuild.
4. **Spare gear risk.** Explicitly settle persistent-quality spare gear carried in main bag on death.
5. **Butcher terminology.** Confirm Crude Butcher/Butcher Knife/skinning-knife capability naming.
6. **Diagram policy.** Pin early discoverable vs diagram-only transitions.
7. **Repair channel policy.** Decide field sharpening instant vs channelled.
8. **Condition presentation.** Confirm qualitative bands only vs exact durability percentage.
9. **Combat native bridge.** Audit AzerothCore autoattack/hit/proc/resource/durability paths for suppress/reuse.
10. **Creature combat fixtures.** Review first anatomy/stagger/hearing profiles rather than guessing them in generic code.

## Documentation contradictions to reconcile before code

1. Journal PR #12 final state/protocol contract wins over older conceptual Journal values/protocol.
2. Crafting failure-consumption vocabulary must be unified where gear experimentation touches Crafting.
3. Preserve Journal solution-matrix highest-unresolved-blocker behaviour; repeated same failure must not leak a lower clue.
4. Survival exact 0–10000 implementation model wins over older normalized runtime wording.
5. Inventory/run branch semantics must be merged/accepted or this stacked Gear branch retargeted before implementation begins.

Do not encode unresolved documentation contradictions into C++.