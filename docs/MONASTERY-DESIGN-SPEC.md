# Thal'vaeth Monastery design specification

## Role

The Monastery is the persistent counterweight to Rotwood: safety, preparation, recovery, knowledge and visible long-term progress.

It must feel like a place the Remnant inhabits, not an MMO vendor lobby.

## Core zones

### Personal quarters
The Remnant's residence and authoritative death recovery location. Death wakes the player in their own bed. No morgue, corpse recovery or explanation of physical return is required in v1.

Stable backend anchor: `home_recovery_spawn`.

Quarters may later support trophies/storage/personalization, but those are not v1 canon merely because space exists.

### Extraction arrival
A distinct authored arrival point for successful runs: `home_extraction_arrival`. It should communicate successful return without using death recovery presentation.

### Preparation/loadout area
A clear place to inspect persistent kit, choose carried items, manage pack/satchel and prepare to enter a district.

### Craft/work areas
Home stations map to canonical station contracts:
- Workbench;
- Cookfire where appropriate;
- Stitch Table;
- Ash-still;
- Forge;
- other explicitly authored stations.

Stations are spatial gameplay anchors, not menu categories floating anywhere.

### Stock/storage
Persistent extracted materials and supplies live in authoritative home stock. Storage UI must preserve provenance/quality/spoil semantics required by Inventory/Crafting.

### Journal/knowledge
The Journal remains the authoritative UI record. A physical study/archive space may reinforce it, but must not create a second conflicting knowledge system.

### Departure
An authored district departure point handles loadout/run-entry validation before transfer into Rotwood.

## Keepers/NPCs

NPCs should provide world context, knowledge, barter or services only where they have a clear role. Avoid recreating WoW's dense vendor/trainer hub.

Keeper knowledge can unlock or contextualize discoveries but cannot replace discovery-first crafting wholesale.

Exact cast/names/dialogue remain narrative work.

## Progression

Monastery progression should be primarily **functional and visual**, not a generic town-upgrade currency tree.

Potential progression must be authored as meaningful capabilities: restoring a forge, enabling a still, opening a workspace, improving preparation access, etc.

No upgrade is canon until its cost, function, prerequisite and visual/world consequence are specified.

## Safety

Monastery is not an active extraction run:
- no run Hunger/Thirst/Director pressure;
- no run creature population;
- home stock is not at risk;
- death recovery cannot be exploited as repair;
- persistent gear condition remains exact.

## UX

The player should be able to understand the home loop spatially:
**wake/arrive → assess → store → repair/craft/learn → choose kit → depart.**

Avoid excessive menu teleportation.

## Acceptance

A v1 Monastery supports death wake, extraction arrival, stock, gear repair, first crafting stations, Journal access, loadout preparation and Rotwood departure without requiring placeholder WoW city services.