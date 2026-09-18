# Thal'vaeth v1 production roadmap

## Product target

A complete vertical slice in which one player can:
**prepare in Thal'vaeth → enter a freshly populated Rotwood run → stealth/explore/fight/gather → manage survival → discover/craft → extract or die → return home → make persistent progress → enter a meaningfully different next run.**

## Phase 0 — contract cleanup

- resolve open Canon Reconciliation blockers that affect code;
- normalize stable keys/protocols;
- audit native AzerothCore systems that must be suppressed/reused;
- ensure docs have one authority chain.

Exit: no programmer must choose between contradictory specs.

## Phase 1 — foundation/runtime identity

- stable player/run IDs;
- run lifecycle;
- persistence/migrations;
- inventory domains/reservations;
- home/run provenance;
- debug/audit framework.

Exit: enter/reconnect/finalize a run without dupes.

## Phase 2 — Survival + Crafting + Journal

- canonical 0–10000 Survival runtime;
- Vigor API;
- born-known field actions;
- discovery persistence/protocol;
- experiment blocker/hint pipeline;
- first stations/recipes.

Exit: non-combat survival/discovery loop persists correctly.

## Phase 3 — Gear + Equipment

- persistent logical gear;
- WotLK carrier bridge;
- durability/condition;
- repair/sharpen;
- first upgrade graph;
- paired/heavy equipment rules;
- pack/Charm slot ownership.

Exit: persistent kit survives death/reconnect and changes capability.

## Phase 4 — Combat

- custom action lifecycle;
- Vigor commitment;
- reach/facing/hit resolution;
- fast/heavy profiles;
- stagger/bleed/cleave/noise;
- Broken behaviour;
- creature attack framework.

Exit: no stock WoW combat path bypasses custom authority.

## Phase 5 — Creature AI + Stealth

- evidence-based perception;
- unified NoiseService;
- awareness/search;
- visibility/concealment;
- movement/Gear noise;
- Still Breath;
- distractions/social evidence;
- 90001–90010 behaviour adapters.

Exit: player can intentionally evade, distract, engage and escape without AI omniscience.

## Phase 6 — Run Generation + Director

- authored node registry;
- deterministic seed/manifest;
- population budgets/special placement;
- mutation persistence;
- Director stress/heat/depth;
- legal pressure opportunities/release windows.

Exit: two runs share geography but produce meaningfully different, reproducible populations/pacing.

## Phase 7 — Extraction + Monastery + Economy

- playable extraction interaction;
- home arrival/death recovery;
- stock;
- preparation/loadout;
- repair/craft spaces;
- barter;
- first functional home progression.

Exit: complete risk→return→progress loop.

## Phase 8 — Progression

- first Aptitudes;
- two Charm slots/effects;
- acquisition/knowledge integration;
- no-level horizontal balance.

Exit: at least two meaningful persistent build identities.

## Phase 9 — Rotwood content pass

- first route/landmarks;
- spawn nodes/perches/posts/chokes;
- visibility/surface/concealment tags;
- resource ecology;
- Ash Hollow;
- extraction site;
- environmental storytelling;
- reviewed creature profiles.

Exit: systems exist in an authored place rather than test rooms.

## Phase 10 — First-hour vertical slice

Run the `FIRST-HOUR-VERTICAL-SLICE.md` experience end to end. Instrument failures, revise rules that are not fun/readable and resist expanding scope until the loop works.

## Phase 11 — hardening

- crash/restart/race tests;
- anti-dupe;
- performance with full population;
- hidden-information audit;
- addon failure resilience;
- GM recovery;
- migration/version compatibility.

## V1 scope guard

Do not block the vertical slice on:
- multiple districts;
- full endgame;
- PvP;
- ranged/firearms/magic combat;
- shields/parry system;
- procedural map generation;
- randomized loot placement;
- elaborate housing;
- dozens of Charms/Aptitudes;
- perfect acoustics;
- advanced weather/cold unless separately activated.

## Definition of foundation complete

The foundation is complete when the first-hour slice works twice in succession with different Rotwood populations, death and extraction both reconcile correctly, no native WoW system bypasses authority, and a player can understand the loop without developer explanation.

After that, design should increasingly happen through **playtest → evidence → targeted revision**, not endless pre-production documentation.