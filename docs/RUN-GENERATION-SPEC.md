# Rotwood run generation specification

## Authority

**Map layout authored. Run population procedural. Director pressure dynamic.**

Rotwood's physical geography is authored and stable. A new run does not rebuild the map. It creates a fresh, server-authoritative population manifest over authored spawn nodes and encounter slots.

Generation covers creatures only unless another content system explicitly opts in. It does **not** silently make loot placement procedural.

## Run identity

Every run owns:
- unique run ID;
- generation version;
- server-generated seed;
- immutable initial generation manifest;
- mutable run-world state derived from that manifest.

The seed is persisted for the lifetime of the run. Reconnect never rerolls it. Server restart reconstructs from persisted manifest/state rather than generating a different run. A terminal run is destroyed; the next run receives a new ID/seed.

A seed alone is insufficient persistence after play begins. Kills, Caller additions, Director activations and other mutations must be represented in run state.

## Authored generation data

Rotwood contains stable authored:
- zones/segments;
- spawn nodes;
- encounter slots;
- special/setpiece slots;
- perch nodes;
- guard/post anchors;
- choke/trap nodes;
- traversal/environment tags.

A node may define:
- stable node key;
- position/orientation;
- zone/depth;
- indoor/outdoor;
- allowed creature families/entries;
- empty weight;
- threat capacity;
- group size bounds;
- required geometry tags;
- mutual exclusions;
- special eligibility;
- activation rules.

Creature AI receives the generated home/node/anchor. It must not assume a permanent spawn GUID or hard-coded coordinate.

## Population budgets

Each zone has an authored population/threat budget. Creatures carry authored threat costs. Generation selects legal combinations without exceeding budget except explicit setpieces.

Budgets create controlled variation, not difficulty roulette. They must prevent pathological rolls such as every nearby node becoming a Brute/Caller cluster.

Some nodes deliberately roll empty. Empty space is gameplay: route uncertainty, tension and room for Director pressure.

## Special placement

Special rules override generic pools.

- Ash Sleeper: only legal dormant/setpiece-compatible nodes; generation must preserve its wake rule.
- Catwalk Prowler: requires a legal perch and one-shot descent geometry.
- Patchwork Brute: requires a valid generated guard/post anchor; return territory is relative to that anchor.
- Broken Snare: requires meaningful LOS/choke geometry.
- Edge Stalker: requires zones compatible with its stalking/reposition behaviour.
- Ash Caller: authored limits prevent uncontrolled starting density.
- Runner/distinctive mobile threats: selected from multiple legal nodes/slots so the player cannot rely on one fixed location every run.

No special may be forced into geometry that breaks its behaviour contract.

## Determinism

Given the same generation version, authored data revision and seed, initial manifest generation must be reproducible.

Deterministic ordering must not depend on hash-map iteration, wall-clock time or database row return order.

Manifest records conceptually include:
- run ID;
- generation version;
- node key;
- spawn ordinal;
- creature archetype/entry;
- home anchor;
- special parameters;
- generated group relation;
- initial activation state.

## Director boundary

Initial generation establishes the run's starting population and eligible pressure capacity.

The Stress Director may later activate/spawn only through authored Director-eligible slots/rules. It does not rewrite the initial manifest invisibly or place creatures arbitrarily around the player.

Director additions become explicit run mutations and survive reconnect.

## Mutation ledger

Run state must record meaningful mutations such as:
- creature killed;
- creature current health where required;
- Prowler descended;
- Caller call resolved/additions created;
- Brute displaced from post;
- Director spawn/activation;
- temporary encounter object state;
- Ash Hollow consumed;
- extraction state.

Events are idempotent and keyed so replay cannot duplicate population.

## Teardown

Death and extraction are terminal. Run population, mutations, AI memory, Director state and run-local objects are destroyed. There is no corpse-return instance.

## Debug

Required:
- `.thal run seed`
- `.thal run manifest`
- `.thal run node <key>`
- `.thal run regenerate <seed>` in isolated GM/test context
- `.thal run mutations`
- `.thal run validate`

A bug report should be reproducible from generation version + seed + manifest/mutation trace.

## Acceptance

Generation is ready when tests prove reproducibility, legal special placement, empty nodes, budget enforcement, no fixed Runner location, reconnect/restart stability, mutation persistence, death teardown and a fresh next-run roll.