# Survival + crafting v1 playable slice

This is the first implementation boundary for Survival and Crafting. It is intentionally smaller than the full design canon.

The goal is to prove the loop:

**enter run → accumulate pressure → spend/recover Vigor → forage/craft → consume survival items → discover recipes/hints → extract/reset → repeat**

Do not expand beyond this slice until the acceptance scenarios pass.

---

## 1. Survival systems in scope

Ship these first:

- Hunger;
- Thirst;
- Infection;
- Vigor;
- walk recovery ceiling;
- Hustle Vigor drain + Hunger/Thirst surcharge;
- generic combat exertion surcharge;
- Fevered threshold/state;
- one-time Ash Hollow recovery;
- Monastery full reset;
- server-authoritative survival-use channels;
- reconnect persistence for active runs.

Out of scope:

- temperature/cold meter;
- morale/sanity;
- sleep/fatigue;
- per-limb wounds;
- long-term survival scars.

---

## 2. Crafting stations in scope

First slice stations:

- Field craft;
- Cookfire;
- Stitch Table;
- Ash-still;
- Charcoal Pit;
- Forge;
- Workbench only as needed for Whetstone/basic assembly.

Butcher Block can remain minimal if carcass breakdown content is not ready at the same time.

---

## 3. Exact first recipes

### Born-known

1. Crude Boil
2. Campfire Roast
3. Crude Bandage
4. Firestart
5. Basic Forage handling

### Discoverable

6. Ash Tea
7. Stitch Kit
8. Whetstone
9. Cord
10. Snare
11. Rendering → Tallow
12. Simple Stew
13. Charcoal
14. Filtered Clean Water
15. Iron Ingot
16. Iron Knife / Short Blade upgrade

No additional recipes are required for first acceptance.

---

## 4. Exact first materials

The slice requires functional semantic items for:

- Foul Water;
- Boiled Water;
- Clean Water;
- Raw Meat;
- Cooked Roast;
- Ashbloom;
- Ash Tea;
- Cloth/Rags;
- Thread;
- Stitch Kit;
- Rough Stone;
- Grit;
- Whetstone;
- Sinew or equivalent fibre;
- Cord;
- simple anchor/trigger material;
- Snare;
- Fat;
- Tallow;
- Deadwood;
- Ash;
- Charcoal;
- Iron Ore;
- Iron Ingot;
- Crude Dagger;
- Iron Knife/Short Blade.

If final item IDs are not yet assigned, the implementation PR that introduces them must update the item registry/canon at the same time.

---

## 5. Discovery/hint loops required

### Ash Tea loop

Must prove:

- wrong/dirty liquid near-miss;
- heat/process near-miss;
- duplicate hint suppression;
- successful recipe discovery;
- Journal persistence after reload.

### Charcoal loop

Must prove:

- open flame produces no Charcoal;
- low-air hint can be learned;
- Charcoal Pit success;
- milestone unlock enables dependent candidate families without auto-learning them.

### Gear gate loop

Must prove:

- Iron upgrade is unavailable before required metalworking gate;
- valid Iron Ingot + upgrade knowledge permits Iron Knife/Short Blade result;
- hidden/diagram-only sibling path remains obscured.

---

## 6. Survival-use items required

### Food

- Roast: basic Hunger restore.
- Simple Stew: better Hunger restore.

### Water

- Foul Water: Thirst restore with Infection risk.
- Boiled Water: safe born-known restore.
- Clean Water: stronger safe restore.

### Medicine

- Crude Bandage: wound-management proof item; does not magically heal.
- Stitch Kit: small Infection reduction/field treatment.
- Ash Tea: primary Infection treatment for first slice.

---

## 7. Required UI behaviour

The first implementation does not need final art, but it must expose enough state to test:

- four survival meters;
- effective/current Vigor distinction if needed;
- channel progress/interruption;
- Journal updates for recipe/hint discovery;
- station attempt result feedback;
- no hidden recipe leakage.

Temporary developer visuals are acceptable if clearly isolated from final styling.

---

## 8. Required server state

A run-state record must be sufficient to reconnect safely with:

- current run identifier/state;
- Hunger;
- Thirst;
- Infection;
- current Vigor;
- Ash Hollow used flag.

Craft/discovery state remains character-persistent through the Journal discovery backend.

---

## 9. First-play session target

A useful internal test session should support this sequence:

1. Start at Monastery fully reset.
2. Enter Rotwood.
3. Walk and observe field Vigor ceiling behaviour.
4. Hustle and observe faster Hunger/Thirst + Vigor drain.
5. Take a configured plague wound and gain Infection.
6. Drink Foul Water and see risk tradeoff.
7. Boil water safely at a fire.
8. Attempt Ash Tea incorrectly and gain one hint.
9. Correct one mistake, fail differently, gain one new hint.
10. Make Ash Tea successfully.
11. Use Ash Tea and reduce Infection.
12. Attempt Charcoal in an open fire and fail meaningfully.
13. Discover/make Charcoal at the correct process.
14. Unlock first metalworking path.
15. Use Ash Hollow once.
16. Attempt second Ash Hollow recovery and get no second jump.
17. Extract.
18. Confirm survival meters fully reset.
19. Confirm Journal/crafting discoveries remain.

If this sequence is not fun/stable, do not author another 50 recipes.

---

## 10. Hard acceptance scenarios

### Scenario S1 — pressure but no arbitrary death

Reach severe Hunger/Thirst/Infection and zero Vigor without incoming lethal damage.

Pass if:

- player is heavily disadvantaged;
- no survival threshold directly kills them.

### Scenario S2 — field ceiling

Spend Vigor below 6500, then walk safely.

Pass if recovery stops at configured ceiling.

### Scenario S3 — active-run reconnect

Disconnect mid-run and reconnect.

Pass if the survival state is restored rather than reset.

### Scenario C1 — learn from failure

Fail Ash Tea in two distinct meaningful ways.

Pass if two different hints can be learned sequentially and duplicates are suppressed.

### Scenario C2 — no spam learning

Submit unrelated ingredient piles.

Pass if they do not reveal useful recipes/hints.

### Scenario C3 — milestone gate

Make Charcoal correctly.

Pass if dependent families become eligible without mass-unlocking all metal knowledge.

### Scenario C4 — transactional safety

Interrupt, double-submit and capacity-fail crafting attempts.

Pass if no duplication or unintended loss occurs.

### Scenario X1 — extract split

Extract after making discoveries.

Pass if survival run-state resets but character knowledge persists.

---

## 11. Expansion order after slice passes

1. Preservation: Jerky / Smoked Meat.
2. Tanning and first Leather armour.
3. Antiseptic Wash / Poultice / Styptic.
4. Fishing: first catch + Grilled Fish + Smoked Fish.
5. Distillation.
6. Steelworking.
7. Waterproofing/waders.
8. Advanced traps.
9. Full charm/gear crafting catalogue.

Each expansion should add test vectors before code.

---

## 12. Definition of done

This slice is complete only when:

- all required recipes have stable keys and definitions;
- all required materials/items have stable IDs or registry entries;
- survival state survives reconnect;
- discovery survives reload/login;
- the listed deterministic tests pass;
- no known client spoof can set survival/crafting truth;
- no hidden recipe name leaks through errors/tooltips;
- duplicate crafting requests cannot duplicate items;
- extraction cleanly separates transient survival pressure from persistent learned knowledge.