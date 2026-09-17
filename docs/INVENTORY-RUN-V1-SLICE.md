# Inventory + death/extraction v1 playable slice

The first slice proves the core extraction promise:

**enter with persistent kit → gather under hard capacity → choose what to carry → extract and keep it, or die and lose it → reconnect cannot cheat the result**.

## 1. In scope

- one active Rotwood run per character;
- main bag bulk pool;
- gather satchel bulk pool + allow-list;
- worn gear free;
- provisional capacities 12 main / 16 satchel;
- run provenance;
- ingredient/use reservations;
- Raw Meat spoil metadata plumbing;
- extraction finalization;
- death finalization;
- death recovery by waking in bed in the Remnant's personal quarters / place of residence within Thal'vaeth Monastery;
- reconnect/restart reconciliation;
- debug tooling.

## 2. Out of scope

- footprint-grid inventory;
- corpse recovery;
- explaining how the Remnant returns to the Monastery after dying in a district;
- detailed quarters decoration/progression systems;
- insurance;
- multiplayer loot ownership;
- account-wide stash sharing;
- mail/auction/trade systems;
- sophisticated stash UI;
- weight-based movement penalties;
- item dropping as a capacity workaround unless explicitly implemented later.

## 3. First item classes

Satchel examples: Ashbloom, Grave Salt, Resin/Pitch, Raw Meat, Raw Hide, Sinew, Bone, Scrap, Ore.

Main-bag examples: Clean/Boiled Water, Roast/Stew, Ash Tea, Stitch Kit, Whetstone, carried spare weapon/tool.

Persistent worn examples: Crude Dagger, Rag Armour, Rag Hood, equipped charms when implemented.

## 4. First run flow

1. Character prepares at Monastery.
2. Server validates 12/16 bulk budgets and creates run ID.
3. Enter Rotwood phase/run.
4. Gather raw materials into satchel.
5. Finished survival items use main bag.
6. Crafting may consume from both legal containers.
7. Capacity forces leaving some loot behind/consuming/crafting it.
8. Reach unlocked extract gate.
9. Successful extraction finalizes haul and returns home.
10. Repeat run; deliberately die.
11. Second run's haul disappears while persistent equipped gear and prior extracted stock remain.
12. Death recovery places the Remnant in bed in their personal Monastery quarters.
13. Relog/restart tests prove neither outcome can be replayed.

## 5. Required integration with existing systems

Journal discoveries are persistent and never deleted by run death. Survival active-run values resume on reconnect and reset after legitimate death/extraction home return. Crafting reservations use the shared inventory reservation layer. Director/gates request lifecycle transitions but do not mutate inventory directly.

The quarters are a presentation/home-space contract. Backend run code should target a stable neutral home recovery spawn identifier rather than embedding room lore into lifecycle logic.

## 6. Acceptance scenarios

A. Fill satchel exactly to capacity; next raw gather fails safely.

B. Attempt finished consumable in satchel; server rejects.

C. Equip/unequip demonstrates worn-free bulk and safe failure if no room to unequip.

D. Extract a mixed haul; verify home ownership exactly once.

E. Die with equivalent haul; verify loss exactly once, persistent kit retained, and Remnant wakes at the bed recovery point in their quarters.

F. Disconnect mid-run; resume same haul/meters.

G. Restart during a synthetic FINALIZING extraction/death test; reconcile without duplication.

H. Race lethal damage against extraction; only one terminal outcome.

I. Attempt hearth/teleport bypass; no free payout.

J. Craft with last free capacity/reserved ingredients; no double-spend.

## 7. Definition of done

Do not expand inventory content until all A–J pass, logs show deterministic finalization, and there is no known route to duplicate, protect or rescue run haul through client timing, reconnect or container movement.