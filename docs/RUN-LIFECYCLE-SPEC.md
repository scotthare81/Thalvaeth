# Run lifecycle — entry, death, extraction and recovery

This document is the authoritative transaction contract for a Thal'vaeth run.

Related canon: `RUN-GATES.md`, `SURVIVAL-IMPLEMENTATION-SPEC.md`, `INVENTORY-IMPLEMENTATION-SPEC.md`, `ECONOMY.md`, `DIRECTOR.md`.

## 1. Core invariant

A run has one durable identity and exactly one terminal outcome.

`PREPARING → ACTIVE → FINALIZING → EXTRACTED | DEAD | ABORTED_ADMIN`

A run may never become both EXTRACTED and DEAD. Disconnect is not a terminal outcome.

## 2. Run identity

On valid entry create a server-owned `run_id` tied to character GUID, district key, start timestamp and lifecycle state. All run-local survival state, haul provenance, Ash Hollow usage, Director state that must survive reconnect, and finalization records refer to this run ID.

Only one ACTIVE/FINALIZING run may exist per character.

## 3. Entry transaction

Before teleport/phase commit:

1. reject an incompatible active run unless reconnect/recovery path applies;
2. validate carried loadout and bulk;
3. snapshot/identify persistent equipped kit;
4. create run row in PREPARING;
5. initialize Survival run state;
6. bind run-present at-risk items to run provenance where required;
7. initialize Director/segment state;
8. teleport/phase into Rotwood;
9. mark ACTIVE only after entry succeeds.

If entry fails before ACTIVE, rollback PREPARING state and do not strand or delete items.

## 4. Active run

While ACTIVE:

- Survival accumulates and persists;
- acquired haul is tagged to run ID;
- crafting/consumption transactions operate normally;
- Ash Hollow is once per run;
- segment/extract gates follow Director rules;
- disconnect persists state and permits resume;
- home-stock mutation is prohibited except explicit remote-safe systems added later.

## 5. Extraction eligibility

The extract gate is not merely a teleport. Interaction requests extraction.

Server validates:

- run is ACTIVE;
- correct character/run/district;
- extract gate is currently unlocked;
- player is in valid extraction volume/range;
- player is alive;
- no incompatible transaction/channel is committing;
- any Director conditions are satisfied.

Client cannot declare extraction success.

## 6. Extraction transaction

Extraction must be atomic/idempotent at the gameplay level.

Recommended order:

1. acquire per-run finalization guard;
2. transition ACTIVE → FINALIZING with requested outcome EXTRACTED;
3. freeze new loot/craft/move transactions for the run;
4. reconcile/cancel reservations;
5. validate authoritative carried inventory;
6. convert eligible run haul into extracted/home-owned stock semantics;
7. persist extraction ledger/audit result;
8. reset Survival/home-return state;
9. clear run-only flags (Ash Hollow, temporary Director state, etc.);
10. teleport/phase to Thal'vaeth Monastery;
11. mark run EXTRACTED;
12. release finalization guard.

If a crash occurs during FINALIZING, recovery must inspect durable steps and continue/rollback safely; it must never duplicate haul.

## 7. Death semantics

Character death during ACTIVE requests terminal outcome DEAD.

Death means:

- run haul is lost;
- satchel contents associated with the run are lost;
- persistent equipped/upgraded gear survives;
- persistent character discoveries survive;
- Journal knowledge survives;
- Monastery stock already extracted before this run survives;
- run-local Survival state is cleared on home recovery;
- the Remnant wakes in bed in their **personal quarters / place of residence within Thal'vaeth Monastery** rather than being permanently deleted.

The quarters are the canonical death-recovery location. They are a lived-in room belonging to the Remnant, not a morgue. The exact room layout, decoration and later progression features are presentation/content concerns and may evolve independently of the run lifecycle.

The game does **not** need to explain how the Remnant physically returned from the district. The run system records the death and recovery outcome without establishing a transport/rescue explanation in canon.

No corpse-recovery loop is required in v1. Lost haul is gone.

## 8. Death transaction

Recommended order:

1. acquire same per-run finalization guard used by extraction;
2. ACTIVE → FINALIZING with requested outcome DEAD;
3. freeze new inventory/craft/loot actions;
4. cancel/reconcile reservations;
5. classify all run-present items from authoritative provenance/risk rules;
6. delete/consume only at-risk haul;
7. preserve persistent equipped kit;
8. write death ledger/audit result;
9. reset Survival/run-local state;
10. revive/transfer the character to the **bed recovery point in the Remnant's personal quarters** through a controlled server path;
11. mark run DEAD;
12. release guard.

The server-facing spawn should use a neutral stable recovery identifier (for example `home_recovery_spawn` or equivalent), not lore-specific terminology such as `morgue`. This keeps lifecycle code independent from room dressing while preserving the quarters/bed presentation contract.

Repeated death callbacks after finalization are no-ops for inventory.

## 9. Race: extraction versus death

This is a required correctness case.

If lethal damage and extraction interaction occur near-simultaneously, whichever terminal transition obtains the finalization guard first is validated against current state. Extraction validation requires the player to be alive. Once FINALIZING begins, the other outcome cannot commit.

No client timing can produce both rewards and death cleanup.

## 10. Disconnect

Disconnect while ACTIVE:

- leaves run ACTIVE;
- persists inventory, Survival and run flags;
- cancels unsafe transient channels/reservations;
- does not extract;
- does not kill;
- does not reset meters;
- reconnect resumes the same run where technically safe.

If exact world-position resume proves unsafe in engine testing, reconnect may return the character to a deterministic safe point within the same run while retaining run state. It must not grant a free extraction.

## 11. Server restart/crash

Durable run records are required precisely so restart cannot become an exploit.

On login after restart:

- EXTRACTED/DEAD runs stay terminal;
- ACTIVE runs resume according to reconnect policy;
- FINALIZING runs are reconciled from ledger/step state before normal play;
- orphaned PREPARING runs are rolled back or safely resumed according to whether entry actually committed.

Never infer extraction merely because the player is geographically in the Monastery after a crash; use lifecycle state.

## 12. Hearth/teleport/admin escape

Ordinary Hearthstone/teleport/unstuck paths must not bypass extraction.

During ACTIVE runs:

- prohibited normal teleport/hearth routes are blocked;
- GM/admin teleport may be permitted but must not silently mark EXTRACTED;
- admin recovery should explicitly choose resume, abort-with-loss, or controlled rollback.

`ABORTED_ADMIN` is audit-only and should not be reachable by ordinary players.

## 13. Voluntary abandonment

No free "leave run" is required in v1. If a future abandon option is added, its default semantic should be equivalent to death/forfeit of haul, not extraction.

## 14. Persistence boundaries

Persist during ACTIVE:

- run ID/state/district;
- Survival values;
- Ash Hollow used flag;
- run item provenance;
- enough position/segment data to resume safely;
- finalization intent/ledger when FINALIZING.

Do not persist purely cosmetic addon state as run authority.

## 15. Finalization ledger

Maintain an auditable record per run containing at least:

- run ID;
- character GUID;
- terminal outcome;
- started/finalized timestamps;
- district;
- extracted/lost item summary or transaction reference;
- schema/protocol version where useful.

The ledger is for correctness/debugging, not a player-facing score screen.

## 16. GM/debug tooling

Required before broad content expansion:

- `.thal run status`
- `.thal run dump`
- `.thal run force-extract` (explicit dev path)
- `.thal run force-death`
- `.thal run reconcile`
- `.thal run clear` guarded/admin only

Commands are illustrative and permission-gated.

## 17. Acceptance criteria

The lifecycle is implementation-ready when tests prove:

1. exactly one active run per character;
2. extraction is server-validated and idempotent;
3. death deletes haul but preserves persistent gear/knowledge;
4. simultaneous death/extraction cannot pay and wipe simultaneously;
5. disconnect never grants extraction or resets pressure;
6. crash/restart cannot duplicate or rescue haul;
7. normal teleports cannot bypass extraction;
8. legitimate death recovery returns the Remnant to the bed in their Monastery quarters;
9. Monastery reset occurs only through a legitimate terminal/home transition;
10. a finalized run cannot be replayed.