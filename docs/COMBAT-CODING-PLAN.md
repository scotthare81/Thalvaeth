# Combat coding plan

Combat should be implemented after the Gear/Equipment foundations it consumes. Keep each stage reviewable and testable.

## PR 1 — action foundation + Vigor

- attack definition registry;
- server action IDs;
- READY/WINDUP/COMMITTED/ACTIVE/RECOVERY state machine;
- server timing;
- Survival Vigor preflight/commit idempotency;
- recovery action locks;
- GM trace.

Fixture: dagger quick attack with no sophisticated secondary effects.

Exit: A/B/R timing-authority tests pass.

## PR 2 — spatial/hit resolution

- frontal reach/arc resolver;
- deterministic candidates;
- target caps;
- LOS hook where reliable;
- server damage resolver consuming Gear profile;
- suppress/audit stock native autoattack/hit side effects.

Exit: C/I/R hit-authority tests pass.

## PR 3 — fast weapon rhythm + bleed

- dagger quick/committed definitions;
- paired logical multi-strike integration;
- bounded BleedService;
- anatomy profile hook;
- fast recovery/cadence;
- logical pair wear once.

Exit: D/E/H/M tests pass.

## PR 4 — heavy rhythm + stagger

- heavy committed attack;
- StaggerService pressure/decay/protection;
- greatsword/greataxe fixture differences;
- long recovery/turning restrictions;
- creature opportunity hook.

Exit: F/G/K tests pass.

## PR 5 — cleave

- authored cleave geometry;
- deterministic multi-target ordering;
- independent target validation;
- secondary-target modifiers if data requires;
- no client target-list authority.

Exit: I tests pass under walls/arcs/caps.

## PR 6 — noise + creature reactions

- semantic CombatNoiseService;
- AI/Director stimulus bridge;
- hearing/investigation fixture;
- light/heavy hit reaction semantics;
- ally death stimulus;
- no universal radius aggro.

Exit: J/N tests pass.

## PR 7 — creature authored attacks + interruption

- creature attack definitions using telegraph/active/recovery;
- Ash Caller/special interrupt fixture;
- player hit reaction/interruption rules;
- pre/post-commit semantics;
- anti-stunlock validation.

Exit: K/N/O tests pass.

## PR 8 — condition + durability integration

- Fine/Worn/Damaged/Broken combat profile application;
- advanced-action disable rules;
- semantic wear from strike events;
- no native double durability loss;
- Broken death persistence.

Exit: L/M plus Gear durability tests pass.

## PR 9 — death/reconnect/run-reset hardening

- pending action cancellation on DEAD;
- committed-cost no-refund rule;
- creature death idempotency;
- reconnect replay protection;
- run teardown clears statuses/actions;
- randomized spawn compatibility tests.

Exit: P/Q pass.

## PR 10 — first playable balance pass

- fixture data moved to reviewed content definitions;
- first Rotwood creature combat profiles;
- first fast/heavy relative tuning;
- debug geometry/trace tools;
- playtest against `COMBAT-V1-SLICE.md` questions;
- document measured tuning changes.

Exit: combat is safe for broader content authoring.

## Implementation hazards to audit

AzerothCore/WotLK can silently fight the custom design through:

- auto-attack timers;
- native hit/miss/dodge/parry tables;
- spell target selection;
- proc/weapon enchant triggers;
- rage/energy resources;
- automatic durability loss;
- client cast/channel assumptions;
- creature stock melee timers;
- movement/facing tolerances.

Each reused path needs an explicit decision: **use as carrier**, **override**, or **suppress**. Never leave two combat systems active simultaneously.

## Before code

Resolve/verify:

1. exact Survival Vigor API and canonical integer units;
2. exact Gear effective-profile API/data keys;
3. logical paired carrier mapping from Equipment;
4. creature combat profile storage location;
5. Director/noise interface boundary;
6. which native WotLK combat paths can be safely reused;
7. active-run disconnect/avatar behaviour;
8. first creature fixtures and their reviewed anatomy/stagger traits.

Do not hard-code provisional content traits into generic Combat just to get the first demo running.