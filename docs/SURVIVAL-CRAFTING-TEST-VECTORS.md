# Survival + crafting deterministic test vectors

These vectors define expected behaviour for the first Survival and Crafting implementation.

They are intentionally implementation-facing. Where exact item IDs are not yet assigned, stable semantic keys are used.

---

# A. Survival meter vectors

## A1 — run entry initializes correctly

Given:

- player is in Monastery with all meters reset;
- player starts a valid Rotwood run.

Expect:

- Hunger = 0;
- Thirst = 0;
- Infection = 0;
- Vigor = 10000;
- effective Vigor max = 10000;
- field ceiling becomes active;
- Ash Hollow used = false.

---

## A2 — idle accumulation

Given a player in Rotwood for 60 seconds, not Hustling and not in combat:

Expect with v1 defaults:

- Hunger increases by 360;
- Thirst increases by 540;
- Infection unchanged;
- Vigor unchanged except any legal walk recovery if below ceiling.

No survival meter may exceed 10000.

---

## A3 — Hustle accumulation

Given 60 seconds of continuous Hustle from full survival meters:

Expect:

- Hunger += `(6 + 5) * 60 = 660`;
- Thirst += `(9 + 7) * 60 = 960`;
- Vigor drains by base Hustle cost modified by Thirst threshold if crossed;
- no direct health damage from Hunger/Thirst.

---

## A4 — walk recovery ceiling

Given:

- current Vigor = 5000;
- effective max = 10000;
- player walks for long enough to exceed theoretical 6500.

Expect:

- current Vigor stops at 6500;
- further walking does not increase it.

---

## A5 — Hunger modifies recovery

Given current Vigor = 4000 and Hunger = 7500:

Expect:

- Hunger state = Weak;
- walk recovery is 50% of base rate;
- effective max applies 0.90 Hunger multiplier before clamping current Vigor.

---

## A6 — Thirst modifies spend

Given Thirst = 7000 and an action with nominal Vigor cost 1000:

Expect:

- Dry state multiplier = 1.25;
- actual spend = 1250 before any later modifier layers;
- Hunger does not directly alter spend in this vector.

---

## A7 — Infection event only

Given Infection = 0 and player takes ordinary non-plague damage:

Expect Infection remains 0.

Then apply a configured minor plague wound:

Expect Infection = 180.

This proves Infection is not inferred from all damage.

---

## A8 — Infection threshold and Vigor clamp

Given:

- Infection rises from 5900 to 6100;
- current Vigor = 9500.

Expect:

- Sick threshold applies;
- effective max becomes 8000 before other modifiers;
- current Vigor clamps to 8000;
- no direct death occurs.

---

## A9 — Fevered does not kill

Given Infection = 10000:

Expect:

- Fevered active;
- effective Vigor max ×0.45;
- weakness/movement pressure active as configured;
- player remains alive.

---

## A10 — zero Vigor does not kill

Given current Vigor reaches 0:

Expect:

- player remains alive;
- Hustle start rejected;
- Short Burst start rejected;
- Vigor-spending aptitude start rejected;
- ordinary movement/combat remains engine-valid unless another debuff modifies it.

---

## A11 — Ash Hollow first use

Given current Vigor = 4100, effective max = 9300, Ash Hollow unused:

Expect first valid recovery sets current Vigor to 8000 and marks used.

---

## A12 — Ash Hollow second use

Same run, current Vigor later falls to 3000, Ash Hollow used = true.

Expect no second jump to 8000.

---

## A13 — Monastery reset

Given arbitrary run-state values:

- Hunger 8700;
- Thirst 9200;
- Infection 6400;
- Vigor 1500;
- Ash Hollow used true.

On valid extraction/home return expect:

- Hunger 0;
- Thirst 0;
- Infection 0;
- Vigor 10000;
- Fevered false;
- run-local flags cleared.

---

## A14 — interrupted drinking

Given player begins a 5-second Clean Water channel and moves at 3 seconds:

Expect:

- channel cancels;
- item remains;
- no Thirst reduction;
- no duplicate/reservation leak.

---

## A15 — completed drinking

Given same setup without interruption:

Expect:

- exactly one item consumed;
- Thirst reduction applied atomically;
- meter clamped at minimum 0.

---

## A16 — reconnect persistence

Given player disconnects during active run with nonzero meters:

Expect reconnect restores the persisted run-state values rather than resetting them.

Returning home normally afterwards clears persisted run-state.

---

# B. Crafting vectors

## B1 — born-known crude boil success

Attempt:

- valid campfire;
- Foul Water;
- sustained boil process.

Expect:

- Boiled Water produced;
- input consumed once;
- no discovery gate rejection;
- recipe knowledge remains/gets idempotently marked known.

---

## B2 — Ash Tea wrong liquid near-miss

Attempt:

- Ashbloom;
- Foul Water;
- otherwise plausible preparation.

Expect:

- candidate resolves to Ash Tea family;
- highest-priority blocker = contaminated/unclean liquid;
- exactly one clean-water hint granted if new;
- recipe not discovered;
- consumption follows configured near-miss class.

---

## B3 — repeat same Ash Tea mistake

Repeat B2 after the hint is already known.

Expect:

- no second discovery grant;
- no additional stronger hint unless the attempt is also closer in a distinct documented way;
- attempt still follows failure-consumption rules.

---

## B4 — Ash Tea wrong process

Given clean/boiled water and Ashbloom but insufficient/incorrect heat/extraction:

Expect:

- heat/process hint granted if new;
- clean-water hint is not re-granted;
- success does not occur.

---

## B5 — Ash Tea success

Given valid station/process, Ashbloom and suitable clean/boiled water:

Expect:

- inputs consumed atomically;
- Ash Tea created;
- recipe discovery granted once;
- output hidden quality evaluated only after success;
- Journal delta emitted.

---

## B6 — random ingredient spam

Attempt unrelated materials with no meaningful candidate recipe.

Expect:

- invalid/no-result outcome;
- no recipe inferred;
- no persistent hint granted;
- no ingredient consumption unless the explicit UI/process already committed to destructive experimentation, which v1 should avoid for no-candidate attempts.

---

## B7 — Charcoal open-fire failure

Attempt good Deadwood in normal Cookfire/open flame.

Expect:

- no Charcoal;
- Ash/byproduct allowed;
- low-air/process hint may be granted;
- inputs consumed according to Charcoal failure definition.

---

## B8 — Charcoal correct process

Attempt suitable Deadwood at Charcoal Pit with low-air process.

Expect:

- Charcoal output;
- Charcoal recipe/process discovery;
- forge milestone key granted exactly once;
- dependent discovery tree becomes eligible, not automatically fully learned.

---

## B9 — filtered water before Charcoal gate

Attempt a would-be filtered-water combination before Charcoal discovery.

Expect:

- hard gate prevents brute-force completion;
- no exact solution leak;
- no dependent recipe discovery.

---

## B10 — filtered water after Charcoal gate

With Charcoal known and valid media/process:

Expect Clean Water produced and recipe discovered/confirmed.

---

## B11 — Iron Ingot without Charcoal

At Forge with Iron Ore but no valid Charcoal fuel/gate:

Expect no Ingot and no brute-force unlock.

---

## B12 — Iron Ingot success

With Iron Ore + Charcoal + valid Smelting knowledge/process:

Expect Iron Ingot and correct material consumption.

---

## B13 — quality after success only

Provide poor-quality meat to valid roast recipe.

Expect recipe succeeds; quality band capped appropriately.

Do not return a recipe failure simply because quality is low.

---

## B14 — output capacity failure

Given ingredients for a bulky result but insufficient valid output capacity:

Expect:

- craft does not commit;
- no ingredients lost;
- no output created;
- clear non-discovery error path.

---

## B15 — simultaneous duplicate attempt

Send two craft requests using the same single input stack before the first resolves.

Expect reservation/transaction logic permits at most one successful consumption/output.

---

## B16 — interrupted long craft

Start a long craft and enter combat before completion.

Expect:

- channel cancels;
- recipe-specific interruption policy applies;
- no duplicate output;
- reservation released or converted to documented consumed subset.

---

## B17 — diagram gate

Attempt a diagram-only gear recipe with correct-looking ingredients but without the diagram discovery.

Expect:

- no craft;
- no recipe identity leak beyond already inferred topology;
- no brute-force unlock.

After diagram discovery, same valid attempt may succeed.

---

## B18 — satchel sourcing

Given raw material in satchel and recipe permits satchel sourcing:

Expect server can reserve/consume it.

Finished output routes to the appropriate destination container and still obeys bulk limits.

---

# C. Cross-system vectors

## C1 — Ash Tea affects Infection only on use

Crafting Ash Tea does not alter Infection.

Using Ash Tea through a completed survival channel reduces Infection by configured amount.

---

## C2 — craft failure hint persists across reload

Learn one Ash Tea near-miss hint, `/reload` or relog, then open Journal.

Expect hint remains known from server persistence.

---

## C3 — Charcoal unlock feeds Journal and crafting

Successful Charcoal discovery should:

- persist process/recipe key;
- update Journal;
- make dependent filter/forge candidate families eligible;
- not auto-create Iron Ingot or reveal all metal recipes.

---

## C4 — survival item quality

Two successful food crafts of different hidden quality should both be known as the same recipe while their restore potency/durability/flavour differs according to item definition.

---

# D. Abuse/leakage vectors

## D1 — malformed craft payload

Invalid station key, impossible quantity or unknown item reference:

Expect request rejected with no state mutation and no hidden recipe information.

## D2 — spoofed survival value

Client attempts to report its own Hunger/Vigor value:

Expect ignored/rejected. Server remains authoritative.

## D3 — hidden recipe name leakage

Unknown recipe/diagram must not leak through error strings, sort labels, debug opcode, tooltip payload or client-facing key names.

## D4 — repeated hint farming

A known near-miss repeated 100 times must not increase discovery state or produce 100 Journal updates.

---

# E. Definition of deterministic

Given the same:

- initial server state;
- recipe definitions;
- normalized input attempt;
- known discovery keys;
- hidden ingredient quality inputs;
- seeded quality RNG where relevant;

the evaluator must choose the same recipe candidate, blocker/hint, consumption class and result.

Only post-success quality randomness may vary, and test builds must be seedable.