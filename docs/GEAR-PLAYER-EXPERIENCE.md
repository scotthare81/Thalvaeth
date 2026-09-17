# Gear player experience

Gear is a persistent relationship with the Remnant's equipment, not a loot carousel. This document defines what the player should understand, feel and see as equipment improves and wears down.

## 1. Core fantasy

The Crude Dagger you begin with matters because it is **yours**. You improve it, maintain it, learn what it can do and eventually reshape your fighting style around what you build.

The player should remember stories about a weapon because it survived runs with them — not because a random higher item-level sword replaced it.

## 2. What a run gives you

A run may give:

- raw/refined materials;
- unusual components;
- diagrams/schematics;
- Keeper/field knowledge;
- repair supplies;
- barter choices that compete with gear improvement.

A run does **not** normally drop a ready-made better sword or armour set.

Finding a rare diagram should create the feeling: “I now know what I could make,” followed by the material problem of actually making it.

## 3. Upgrade anticipation

Unknown future upgrades remain Journal silhouettes/obscured entries. The player may understand that a branch exists without being told its exact final recipe/stat package.

Knowledge arrives in layers:

1. **possibility** — a silhouette/rumour/diagram fragment suggests something exists;
2. **understanding** — the relevant diagram/process becomes known;
3. **requirements** — known materials/station are intelligible;
4. **construction** — the upgrade is committed at home;
5. **experience** — the player learns its actual feel through use.

The UI should not flatten these into a conventional talent tree with every future node visible on day one.

## 4. Station experience

Gear upgrading happens at a physical Monastery station. The station is part of progression: returning home with materials should feel materially different from opening a menu anywhere in the world.

At a station, the player should be able to inspect the current item and understand:

- its current form;
- its condition in plain language;
- what known work can be performed now;
- what known materials are missing;
- whether a diagram/process is still unknown without leaking the answer;
- which known branch choices are mutually meaningful.

Do not present “Upgrade available!” quest-marker spam. The Journal and station are places the player chooses to consult.

## 5. Durability feedback

Durability should be readable without a constant numeric meter dominating play.

Player-facing vocabulary:

- **Fine** — maintained and dependable;
- **Worn** — used, edge/straps/plates beginning to need attention;
- **Damaged** — clearly compromised; repair before pushing deeper;
- **Broken** — badly compromised but recoverable.

Feedback channels can include:

- inventory/gear condition word;
- restrained icon treatment;
- item description changes;
- station inspection text;
- occasional transition message when entering Damaged/Broken;
- combat/tool feel from the actual condition effect.

Avoid durability percentages in normal v1 UI unless later playtesting proves the bands are too opaque.

## 6. Transition messaging

Do not spam every point of wear.

Useful moments:

- first transition Fine → Worn: subtle;
- Worn → Damaged: clear warning;
- Damaged → Broken: unmistakable;
- successful field sharpen: concise confirmation;
- successful structural repair: concise confirmation;
- upgrade completion: meaningful but not loot-rarity fireworks.

Example tone, not locked copy:

- “The edge is beginning to turn.”
- “The blade is badly nicked.”
- “The binding gives under your grip.”
- “The edge will not hold. It needs proper work.”

These are diegetic observations, not `Durability: 31/100`.

## 7. Broken in the field

Broken should create a survival problem, not an account-loss panic.

A Broken weapon can leave the player deciding whether to:

- continue with severe penalties;
- switch to a carried spare/tool if they accepted the bulk cost;
- avoid combat and extract;
- use a limited field intervention if that specific item/action supports it.

A Broken armour piece similarly makes continued exposure dangerous. The answer is usually to get home and repair, not destroy/drop the persistent item.

## 8. Sharpening experience

A Whetstone is preventative field maintenance. It should feel like extending the life of a working blade during a run.

It is not a universal repair kit. A snapped binding, cracked haft, torn coat or structurally ruined blade needs the appropriate station/material work.

With the v1 single durability pool, presentation still distinguishes “the edge needs attention” from “this needs proper repair” according to condition/action eligibility.

## 9. Weapon branch choice

The late branch is a playstyle decision the player understands through consequences.

**Fast:** less commitment per action, lower individual impact, harvest convenience with dagger forms, more contacts/bleed-oriented pressure.

**Heavy:** high commitment and Vigor demand, loud, strong spacing/impact/stagger/cleave, less convenient harvesting and larger spare bulk.

The UI must not label one “best” or use green/red comparison arrows that imply a universal upgrade score. It can compare concrete known changes such as heavier/louder/longer reach.

## 10. Armour choice

Armour is also a build choice rather than linear rarity.

The player should understand consequences in physical language:

- leather is quiet and easy to move in;
- mail protects better but announces movement;
- plate lets you survive punishment but taxes Vigor and stealth;
- quilted layers remain useful beneath heavy protection;
- warmth/weather layers are separate from armour class.

Hardened Leather remains desirable after Full Plate exists.

## 11. Hidden quality experience

Quality is hidden numerically but not meaningless.

A particularly good piece can be suggested by:

- adjective/description;
- how long the edge holds;
- how well it protects;
- how it feels in repeated use.

Do not create a community-obvious disguised rarity system where three fixed adjectives map perfectly to green/blue/purple tiers. Variation should be experiential and bounded.

## 12. Death continuity

After death, the player wakes in their personal quarters. Their persistent equipment is still theirs, in the condition it was in when the run ended.

That continuity matters. The cost of death is the lost run haul/satchel and opportunity; the game does not add a hidden gear-reset punishment or hand out a fresh replacement dagger.

A player who died because their blade was Damaged should wake with a repair problem waiting for them. That creates a natural home-loop decision without making death delete progression.

## 13. Personal quarters relationship

The quarters are the emotional anchor for persistent kit, even if full display systems come later. Future presentation may support a weapon rack, work notes, storage or visual history, but v1 does not require these.

Do not make those future ideas authoritative mechanics until separately designed.

## 14. Journal relationship

The Gear Journal records learned possibilities and knowledge. The physical item/gear screen records what the player currently owns and its condition. The station records what work can be performed.

Keep these responsibilities distinct:

- Journal = **what I know**;
- Gear = **what I have/wear**;
- Station = **what I can do to it here**.

## 15. UX anti-patterns

Do not introduce:

- item-level scores;
- rarity colours;
- random loot beams;
- DPS-score recommendation arrows;
- durability repair-all button with no material identity;
- exact hidden-quality stars;
- full future upgrade tree revealed by client data;
- death-generated replacement copies;
- repair spam after every fight;
- constant “your weapon lost 1 durability” messages.

The desired experience is deliberate maintenance and earned knowledge, not MMO inventory churn.