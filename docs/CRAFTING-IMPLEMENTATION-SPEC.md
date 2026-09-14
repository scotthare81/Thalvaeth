# Crafting implementation specification

This document converts the Crafting canon into a deterministic implementation contract.

Related canon:

- `CRAFTING.md`
- `DISCOVERY-SOLUTIONS.md`
- `JOURNAL-HINTS.md`
- `JOURNAL-KEY-REGISTRY.md`
- `ITEMS.md`
- `MATERIALS.md`
- `GEAR.md`
- `SURVIVAL-IMPLEMENTATION-SPEC.md`

---

## 1. Core laws

1. No crafting skill levels.
2. Recipe knowledge is character knowledge, not client state.
3. A valid recipe succeeds because the player supplied the right inputs/process, not because of hidden profession XP.
4. Near-misses may teach one persistent hint when the attempt is meaningfully close.
5. Random ingredient spam should usually fail silently.
6. Quality is evaluated only after recipe success.
7. Diagram/fragment/keeper-gated content cannot be brute-forced unless its gate explicitly allows experimentation.

---

## 2. Server-owned attempt object

Every craft attempt is normalized into a server-side record containing:

- character GUID;
- station/process key;
- run/home context;
- ordered or canonicalized ingredient multiset;
- ingredient item IDs and hidden quality bands;
- optional tool key;
- optional fuel/process modifiers;
- known discovery keys relevant to the recipe family;
- attempt timestamp/nonce;
- source location/station GUID.

The client may request an attempt, but never declares the result.

---

## 3. Recipe definition shape

Every implemented recipe must have a stable definition with:

- `recipe_key`;
- result item/result action;
- allowed station/process keys;
- exact required inputs;
- optional substitutions or family inputs;
- required discovery gates;
- forbidden/invalid substitutions where needed;
- success consumption rules;
- output quantity;
- quality model;
- ordered near-miss rules;
- hint keys for those rules;
- failure-consumption class;
- Journal discovery result.

No recipe should exist only as hand-coded `if` logic without a stable key and documented definition.

---

## 4. Station model

Canonical v1 stations/processes:

| Key | Player-facing station | Location | Main roles |
|---|---|---|---|
| `station.butcher` | Butcher Block | Monastery | carcass breakdown |
| `station.cookfire` | Cookfire | Monastery / crude field variant | cooking, boil, render, smoke |
| `station.stitch` | Stitch Table | Monastery | textile/leather work, bandages, repair |
| `station.ash_still` | Ash-still | Monastery | medicine, brewing, distilling |
| `station.char_pit` | Charcoal Pit | Monastery | low-air charring |
| `station.forge` | Forge | Monastery | smelting, blades, plates, metal repair |
| `station.workbench` | Workbench | Monastery | assembly, tools, gear frames |
| `station.field` | Field craft | Run | crude bandage, cordage, traps, sharpen, temporary mend |

The field is not a universal station. Each recipe explicitly lists whether `station.field` is valid.

---

## 5. Station validation order

Before matching recipes:

1. confirm player is alive and eligible to craft;
2. confirm station exists/is reachable;
3. confirm station key is valid for requested operation;
4. confirm inventory ownership of ingredients/tools;
5. reserve ingredients atomically;
6. build normalized attempt;
7. evaluate candidate recipes.

If station validity fails, do not leak recipe information.

---

## 6. Candidate selection

To prevent one attempt from revealing multiple recipes, candidate selection is deterministic.

Recommended order:

1. recipes valid for the station;
2. recipes whose core ingredient family matches;
3. recipes whose gate state allows experimentation;
4. exact matches first;
5. otherwise score meaningful near-miss rules;
6. choose the single highest-priority candidate;
7. if tied, use stable recipe-key ordering rather than random choice.

The player receives at most one learned hint per attempt.

---

## 7. Exact-match evaluation

A successful recipe requires all of:

- valid station/process;
- all mandatory ingredient families and quantities;
- no disallowed ingredient conflict;
- required knowledge gate satisfied;
- required tool/fuel/process state satisfied.

Success then proceeds:

1. consume inputs atomically;
2. determine output hidden quality;
3. create result(s);
4. grant recipe discovery if newly learned;
5. grant any process/material knowledge implied by success;
6. emit Journal/crafting delta;
7. log the result for diagnostics.

---

## 8. Near-miss evaluation

Near-miss rules are recipe-owned and ordered by blocker precedence.

Each rule defines:

- predicate over the attempt;
- stable failure key;
- hint key;
- minimum prior knowledge if needed;
- whether it infers the recipe node;
- failure consumption class;
- optional side effect.

Only the first meaningful unmet blocker teaches knowledge.

Example Ash Tea blocker order:

1. correct herb family but contaminated/foul liquid → clean-water hint;
2. correct ingredients but wrong/no extraction heat → needs-heat hint;
3. correct process but gate not learned → no brute-force if gate is hard;
4. exact success.

Do not reveal later blockers while an earlier blocker remains unresolved.

---

## 9. Failure consumption classes

Canonical classes:

| Class | Behaviour |
|---|---|
| `none` | no ingredients consumed |
| `sample` | consume only a small expendable sample/reagent |
| `partial` | consume configured fraction/subset |
| `all` | attempt destroys all committed inputs |
| `transform_bad` | consume inputs and produce a failed/spoiled result |

V1 defaults:

- UI/input mistakes and invalid station: `none`;
- meaningful herb/medicine experiments: usually `sample` or `partial`;
- cooking attempts: usually `transform_bad`;
- charcoal/open-fire mistakes: `all` or ash byproduct;
- metalworking wrong-heat attempts: `partial` where practical;
- diagram-gated blocked attempts: `none` and no useful hint unless a gate-specific observation is intended.

The goal is consequence without making experimentation economically suicidal.

---

## 10. Hidden quality

Quality is a post-success calculation.

V1 quality bands:

- 0 = low;
- 1 = standard;
- 2 = good;
- 3 = best.

The player does not see numeric bands.

Inputs contribute a bounded quality score. The output roll is weighted by that score.

A simple v1 model:

- poor inputs cap output at standard;
- standard inputs: 70% standard / 25% good / 5% best;
- good inputs: 45% standard / 40% good / 15% best;
- best inputs: 20% standard / 50% good / 30% best.

The exact percentages are tuning knobs but must be centralized.

Quality affects potency/durability/yield according to item family, not recipe validity.

---

## 11. V1 recipe families to implement first

The first Crafting implementation should cover only enough breadth to prove the system.

### Born-known

- crude butcher;
- campfire roast;
- crude boil;
- firestart;
- crude bandage;
- basic forage handling.

### Early discoverable

- Ash Tea;
- Stitch Kit;
- Whetstone;
- Cord;
- Snare;
- Tallow/rendering;
- simple stew;
- Charcoal.

### First milestone chain

- Charcoal;
- filtered Clean Water;
- Iron Ingot;
- Iron Knife/short blade upgrade.

Do not implement the entire catalogue in the first coding pass.

---

## 12. V1 canonical truth table

### Crude boil

- station: cookfire/field campfire;
- input: Foul Water;
- process: sustained boil;
- result: Boiled Water;
- born-known;
- failure: insufficient heat/time may produce a non-persistent process message, not a new discovery.

### Campfire roast

- station: cookfire/field;
- input: Raw Meat;
- result: Roast;
- born-known;
- quality affected by meat quality;
- grossly wrong heat may produce Burnt Meat (`transform_bad`).

### Ash Tea

- station: ash-still or approved cookfire preparation if canon allows;
- inputs: Ashbloom + Clean/Boiled Water;
- process: controlled steep/heat;
- result: Ash Tea;
- discoverable through experimentation;
- near misses map to `DISCOVERY-SOLUTIONS.md` hint keys.

### Stitch Kit

- station: stitch table; crude field variant allowed at reduced quality;
- inputs: Thread + clean Cloth/Rags;
- result: Stitch Kit;
- recipe can be learned by first successful structured attempt or taught knowledge depending final key registry.

### Whetstone

- station: field/workbench;
- inputs: Rough Stone + Grit;
- result: Whetstone;
- early experimentable.

### Cord

- station: field/workbench;
- family input: suitable sinew/fibre;
- process: twist/bind;
- result: Cord;
- early experimentable.

### Snare

- station: field/workbench;
- inputs: Cord + suitable trigger/anchor material;
- requires Cordage knowledge;
- result: Snare;
- early discoverable.

### Rendering

- station: cookfire;
- input: Fat;
- controlled heat;
- result: Tallow/Rendered Oil family according to material definition;
- early discoverable.

### Charcoal

- station: char pit only;
- input: suitable Deadwood;
- process: low-air char;
- result: Charcoal;
- open fire never produces Charcoal;
- open-fire attempt may produce Ash and the low-air hint;
- successful discovery grants the forge milestone key.

### Filtered Clean Water

- station: cookfire/workbench filtration setup as defined;
- requires Charcoal discovery;
- inputs: Boiled/Foul Water + viable filter media including Charcoal;
- result: Clean Water;
- does not exist as a brute-force pre-Charcoal recipe.

### Iron Ingot

- station: forge;
- requires Charcoal + Smelting gate;
- inputs: Iron Ore + Charcoal;
- result: Iron Ingot;
- wrong fuel/insufficient process heat follows documented metal hints.

---

## 13. Craft channels and interruption

Crafting actions longer than trivial assembly use server-authoritative channels.

An active craft:

- reserves ingredients;
- is interrupted by combat, meaningful damage, station range loss, logout or map transfer;
- on interruption applies the recipe's interruption consumption policy;
- on completion commits consumption/result atomically.

Do not create the item client-side before server completion.

---

## 14. Inventory integration

Crafting must respect container rules:

- raw materials may be sourced from the satchel if allowed;
- finished outputs go to the correct main-bag/satchel destination by item classification;
- if output capacity is unavailable, the attempt must not destroy ingredients unless a deliberate ground-drop/output-buffer system exists;
- ingredient reservation must prevent double-use through simultaneous crafting requests.

Bulk enforcement happens before successful commit.

---

## 15. Journal integration

Successful or meaningfully failed crafting can grant:

- recipe discovery;
- material property discovery;
- process discovery;
- hint discovery;
- milestone/branch discovery.

All grants use stable keys from `JOURNAL-KEY-REGISTRY.md` and are idempotent.

The crafting system does not store its own duplicate notion of "known recipe" outside the agreed discovery model unless needed for performance caching.

---

## 16. Diagnostics

Every craft attempt should be loggable in a compact diagnostic form:

- player GUID;
- recipe candidate key;
- station;
- outcome: success / near-miss / invalid / gated;
- failure key;
- hint granted yes/no;
- consumed inputs class;
- output quality band;
- elapsed channel time.

GM tooling should eventually support evaluating an attempt without consuming inputs.

---

## 17. Acceptance criteria

Crafting is ready for broader content when:

1. born-known recipes work without Journal prepopulation bugs;
2. Ash Tea can be discovered through the intended near-miss chain;
3. duplicate failed experiments do not grant duplicate hints;
4. random ingredient spam does not reveal recipe answers;
5. Charcoal cannot be made in an open fire;
6. successful Charcoal discovery unlocks dependent recipes exactly once;
7. diagram/hard-gated recipes cannot be brute-forced;
8. quality is evaluated only after success;
9. interrupted crafts cannot duplicate or lose items incorrectly;
10. satchel/main-bag routing is respected;
11. output-capacity failure is safe;
12. all test vectors are deterministic.

---

## 18. Explicit non-goals for first implementation

- crafting queues;
- offline crafting;
- profession XP;
- mass/batch crafting UI;
- marketplace/vendor recipe auto-learning;
- hundreds of recipes;
- client-side recipe validation authority.