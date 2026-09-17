# TODO — implementation status

## Maps

- [x] Home — **Thal'vaeth Monastery** at **Hearthglen** (map 0)
- [x] Run — **Rotwood** outdoor Duskwood worgen cluster (map 0, gated)
- [x] Gate SQL shell + segment + extract (91001+)
- [ ] Phase 1/2 split (home vs run)
- [ ] MPQ — Hearthglen Brill-grey
- [ ] Strip vanilla spawns (incl. Nightbane worgen) inside Rotwood AT
- [x] Director/run pacing sketched — `DIRECTOR.md` (rhythm, heat/noise, segments, hooks)
- [ ] Director segment gate open hooks
- [ ] Tune fence ring coords in GM mode
- [ ] Future runs (proposed): snow/cold (warmth gate) + fog (visibility) — `MAPS.md`; decide if cold is tracked

## Creatures

- [x] Catalog + C++ specials + fodder SmartAI
- [x] POC spawns in Rotwood (phase 2)
- [ ] Sleeper damage tune
- [ ] MPQ Sleeper eyes
- [ ] Persistent run health — `RegenHealth=0` (90001–90010); base AI keeps HP on evade/reset (Brute leash, Stalker flee); Director last-HP clamp; clear on `OnRunStart`
- [ ] Animal layer (90101+) — Rotwood Boar / Hound (worg model) / Deer / Hare, rare Rotwood Tusker; drops feed Hunger + barter
- [x] Animal naming convention `<District> <Kind>` locked in `NAMES.md`

## Items & crafting

- [x] `MATERIALS.md` — raw catalog
- [x] `CRAFTING.md` — recipes, discovery, stations, durability/mend
- [x] Discovery tree pinned — Charcoal→Forge spine plus tanning/brewing/distilling/preservation/waterproofing
- [x] `CRAFTING-IMPLEMENTATION-SPEC.md`
- [x] `SURVIVAL-CRAFTING-V1-SLICE.md`
- [x] `SURVIVAL-CRAFTING-TEST-VECTORS.md`
- [x] `SURVIVAL-CRAFTING-CODING-PLAN.md`
- [x] `GEAR.md` — upgrade-only gear canon
- [x] `ITEMS.md` — made/found catalog
- [x] Armour ladder
- [x] Weapon ladder
- [ ] Gear tuning — per-slot wear rates, tier count, diagram gating, skinning-knife slot for 2H builds
- [ ] Item tuning — per-item IDs, poison balance, fish/trap yields, mushroom tells
- [x] Bulk model — bulk 1/2/4; worn free; v1 capacity budget
- [x] `INVENTORY-IMPLEMENTATION-SPEC.md` — authoritative main bag/satchel, bulk, provenance, spoilage and anti-duplication contract
- [x] `INVENTORY-RUNTIME-SPEC.md` — movement, stacking, reservations and finalization runtime contract
- [x] `INVENTORY-RUN-V1-SLICE.md` — exact first inventory/extraction playable slice
- [x] `INVENTORY-RUN-TEST-VECTORS.md` — deterministic capacity/death/extraction/race/restart tests
- [x] `INVENTORY-RUN-CODING-PLAN.md` — staged implementation PR sequence
- [ ] Fancy grid inventory (future)
- [ ] Assign per-item IDs in the 61xxx material bands
- [x] Near-miss hint library + authoritative solution/failure matrix

### First Survival + Crafting implementation sequence

- [ ] Active-run survival persistence + migration
- [ ] Fixed server survival tick + centralized tuning values
- [ ] Vigor field ceiling, walk recovery, Hustle/Short Burst/combat exertion
- [ ] Interruptible eat/drink/treatment channels
- [ ] Explicit Infection event API
- [ ] Ash Hollow once-per-run recovery
- [ ] Server-owned craft attempt engine + ingredient reservation
- [ ] Born-known recipes: Crude Boil, Roast, Bandage, Firestart
- [ ] Ash Tea near-miss/discovery chain
- [ ] Charcoal Pit + Charcoal milestone gate
- [ ] Filtered water + first Iron Ingot/knife chain
- [ ] Stitch Kit, Whetstone, Cord, Snare, Rendering/Tallow, Simple Stew
- [ ] Run all Survival/Crafting test vectors

## Run lifecycle + inventory

- [x] `RUN-LIFECYCLE-SPEC.md` — PREPARING→ACTIVE→FINALIZING→EXTRACTED/DEAD state machine; reconnect/restart rules
- [x] Death contract — haul/satchel lost, persistent equipped gear + discoveries retained, morgue return
- [x] Extraction contract — server validation, idempotent haul promotion, Survival reset, home return
- [x] Death/extraction race and crash-reconciliation behaviour pinned
- [x] Satchel allow-list and provisional v1 capacity: main bag 12 bulk / satchel 16 bulk
- [x] Run provenance + reservation semantics pinned

### First Inventory + Run implementation sequence

- [ ] Run lifecycle persistence + unique active-run guard
- [ ] PREPARING/ACTIVE/FINALIZING/terminal service + debug dump/reconcile
- [ ] Bulk registry + satchel allow-list + 12/16 capacity service
- [ ] Server acquisition/move validation; worn gear free
- [ ] Run item provenance tied to run ID
- [ ] Shared crafting/survival item reservation layer
- [ ] Extraction finalization guard + haul promotion/deposit + ledger
- [ ] Death finalization + at-risk deletion + persistent-kit preservation + morgue
- [ ] Death/extraction race hardening
- [ ] Restart reconciliation for PREPARING/ACTIVE/FINALIZING runs
- [ ] Block Hearth/ordinary teleport extraction bypass
- [ ] Raw Meat spoil metadata/split/merge plumbing
- [ ] Minimal ThalvaethUI bulk/satchel presentation
- [ ] Run all `INVENTORY-RUN-TEST-VECTORS.md` cases before expanding inventory

## Economy

- [ ] `ECONOMY.md` — tiered barter implementation (no coin), material ladder, Monastery keepers
- [x] Corruption replaced by **Infection**
- [x] Survival interlock — `SURVIVAL.md`
- [x] Survival implementation contract
- [x] Gather satchel design/implementation contract — allow-list, bounded bulk, lost-on-death
- [ ] Satchel upgrade curve tuning beyond v1 16-bulk start
- [ ] Raw-meat spoil timer tuning + preservation balance

## Journal / UI

- [x] Creature journal table + addon wire stub
- [x] Journal Record design
- [x] Concrete Journal content/reveal map
- [x] Canonical hint library
- [x] Authoritative discovery/solution matrix
- [x] Technical implementation contract
- [x] UI/interaction contract
- [x] First playable slice + acceptance scenarios
- [x] Data model, wire protocol, key registry, deterministic tests and coding plan

### First Journal implementation sequence

- [ ] Generic per-character discovery persistence + discovery service
- [ ] Versioned HELLO/READY + full Journal snapshot
- [ ] GM/debug Journal tooling
- [ ] Tabbed Journal shell/loading
- [ ] Creature Unknown→Sighted→Engaged/observed
- [ ] Persistent observed creature abilities
- [ ] Born-known basics + Ashbloom/Ash Tea
- [ ] Deterministic near-miss evaluator + one-hint persistence
- [ ] Charcoal gate + filtration inference
- [ ] Stitch path, Ashbloom/Gravecap layered knowledge
- [ ] Eel, Snare, Whetstone
- [ ] Gear slice + obscured sibling
- [ ] Diagram-gated weapon branch
- [ ] Stillstone + Deep-Lung Token
- [ ] Run Journal v1 acceptance scenarios
- [ ] Art polish last

## Aptitudes & charms

- [x] Charm/aptitude items — 2-slot design currently assumed; aptitude vs passive; craft/upgrade
- [ ] Confirm charm slot count; choose proposed v1 aptitudes

## Client / branding

- [x] Brand concept set
- [x] `CLIENT-BRANDING.md`
- [ ] Rebuild wordmark in real weathered-serif type
- [ ] Produce shippable BLP/login assets + original login music
- [ ] Decide crest/wordmark/accent treatment
- [ ] Build login glue patch

## Core

- [ ] Spell strip + Remnant first login
- [ ] Port Hearthglen ↔ Rotwood entry
- [ ] Stress Director (full)
