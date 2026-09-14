# Survival + crafting coding plan

This plan sequences implementation so server truth is established before presentation and content expansion.

---

## Milestone 1 — Survival state backbone

Deliver:

- per-character active-run survival state;
- Hunger/Thirst/Infection/Vigor storage;
- fixed 1-second simulation tick;
- centralized tuning constants/config;
- Monastery reset;
- reconnect persistence;
- no UI polish.

Do not add crafting yet.

Acceptance:

- vectors A1–A13 pass;
- zero Vigor never kills directly;
- reconnect does not wipe state.

---

## Milestone 2 — Vigor actions + survival-use channels

Deliver:

- walk recovery ceiling;
- Hustle drain;
- Short Burst hook contract;
- generic combat exertion surcharge;
- interruptible eat/drink/treatment channel;
- safe atomic item consumption.

Acceptance:

- A14/A15 pass;
- Thirst multipliers alter Vigor spend deterministically;
- interrupted use cannot duplicate/consume incorrectly.

---

## Milestone 3 — Infection event API

Deliver a narrow server API for explicit Infection events:

- minor plague wound;
- heavy plague wound;
- special ability amount;
- contaminated consumable event.

Creature scripts call this API rather than editing player survival fields directly.

Acceptance:

- non-plague damage does not raise Infection;
- configured plague events do;
- threshold/effective-max transitions are deterministic.

---

## Milestone 4 — Ash Hollow

Deliver:

- once-per-run recovery flag;
- recovery target;
- safe-pocket state;
- combat cancellation.

Acceptance:

- A11/A12 pass.

---

## Milestone 5 — Craft attempt engine

Deliver:

- server-owned normalized attempt structure;
- station validation;
- ingredient reservation;
- candidate selection;
- exact-match path;
- near-miss path;
- result/failure object;
- no content beyond test recipes.

Acceptance:

- malformed requests cannot mutate state;
- simultaneous duplicate requests are safe;
- no-candidate attempts grant no hint.

---

## Milestone 6 — First born-known recipes

Implement:

- Crude Boil;
- Campfire Roast;
- Crude Bandage;
- Firestart.

Acceptance:

- born-known recipe handling works without accidental discovery gating;
- output capacity failure is safe;
- quality only runs after success.

---

## Milestone 7 — Journal-aware near-miss crafting

Implement:

- Ash Tea candidate;
- dirty-liquid blocker;
- heat/process blocker;
- stable hint grants;
- duplicate hint suppression;
- successful recipe discovery.

Acceptance:

- B2–B5 pass;
- hint persistence survives reload;
- only one new hint per attempt.

---

## Milestone 8 — Charcoal milestone

Implement:

- Charcoal Pit station;
- open-fire failure/byproduct;
- low-air near-miss hint;
- successful Charcoal discovery;
- dependent gate eligibility.

Acceptance:

- B7–B10 pass;
- open fire can never accidentally output Charcoal.

---

## Milestone 9 — first metal chain

Implement:

- Iron Ore + Charcoal → Iron Ingot;
- first Iron Knife/Short Blade upgrade;
- required gear/diagram discovery checks.

Acceptance:

- no pre-gate brute force;
- output item and knowledge state commit atomically.

---

## Milestone 10 — remaining v1 recipes

Implement:

- Stitch Kit;
- Whetstone;
- Cord;
- Snare;
- Rendering/Tallow;
- Simple Stew.

Acceptance:

- all v1 test vectors pass;
- satchel/main-bag sourcing and routing work;
- hidden quality is consistent.

---

## Milestone 11 — temporary UI integration

Only after server behaviour is stable:

- survival meters;
- survival channel progress;
- station attempt feedback;
- Journal update hooks;
- developer diagnostics toggle.

No final art requirement yet.

---

## Milestone 12 — balancing pass

Tune only after end-to-end play:

- Hunger/Thirst rate;
- Vigor field ceiling;
- walk recovery;
- Hustle/Short Burst costs;
- Infection values;
- restore potency;
- channel durations;
- experiment failure costs.

Changes to tuning values should not require architecture changes.

---

# PR strategy

Prefer small PRs:

1. survival state + migration;
2. survival simulation;
3. consumption channels;
4. Infection API;
5. Ash Hollow;
6. crafting evaluator core;
7. born-known recipes;
8. Ash Tea discovery;
9. Charcoal milestone;
10. metal chain;
11. remaining v1 recipes;
12. UI integration.

Avoid one giant Survival+Crafting PR.

---

# Ready-to-code gate

Coding may begin when:

- `SURVIVAL-IMPLEMENTATION-SPEC.md` is accepted;
- `CRAFTING-IMPLEMENTATION-SPEC.md` is accepted;
- `SURVIVAL-CRAFTING-TEST-VECTORS.md` is accepted;
- `SURVIVAL-CRAFTING-V1-SLICE.md` is accepted;
- all referenced Journal key/protocol docs are already canon;
- required first-slice item IDs are either assigned or explicitly scheduled in the first content PR.

No implementation PR should invent new survival semantics or recipe-discovery rules without updating canon first.