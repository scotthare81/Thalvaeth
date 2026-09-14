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

- [x] `MATERIALS.md` — raw catalog (dropped / foraged / mined / scavenged, incl. reclaimed metals)
- [x] `CRAFTING.md` — recipes (food/drink/refining/smelt/forge), discovery (fragments + experiment/hints), stations, durability/mend
- [x] Discovery tree pinned — Tier 0 given vs milestone gates (Charcoal→Forge spine; tanning, brewing, distilling, preservation, waterproofing)
- [x] `CRAFTING-IMPLEMENTATION-SPEC.md` — deterministic station/attempt/recipe/near-miss/quality contract
- [x] `SURVIVAL-CRAFTING-V1-SLICE.md` — exact first Survival + Crafting implementation boundary
- [x] `SURVIVAL-CRAFTING-TEST-VECTORS.md` — deterministic meter/crafting/cross-system tests
- [x] `SURVIVAL-CRAFTING-CODING-PLAN.md` — staged coding and PR sequence
- [x] `GEAR.md` — upgrade-only (no drops): crude dagger + rags start; weapon styles (dual-wield vs 2H); armour classes cloth→plate; slots/axes
- [x] `ITEMS.md` — made & found catalog (crafting→breaking), invisible quality tiers, + fishing, expanded forage/mushrooms, traps, poisons
- [x] Armour ladder — full entry→endgame tiers per class (Leather: Boiled/Studded/Hardened; Mail: Ring/Riveted/Splinted; Plate: Half/Full); layering + endgame-per-playstyle
- [x] Weapon ladder — entry→endgame (shared early blade → dual-wield vs 2H); iron/steel/bronze tradeoffs; weapon-as-tool utility (butcher/wood)
- [ ] Gear tuning — per-slot wear rates, tier count (3 vs 4), diagram gating, skinning-knife slot for 2H builds
- [ ] Item tuning — per-item IDs (60xxx), poison balance, fish/trap yields, mushroom tells
- [x] Bulk model — items have bulk 1/2/4; bags/satchel are bulk pools (worn = free); v1 = capacity budget
- [ ] Fancy grid inventory (future) — footprint UI in ThalvaethUI + server-side virtual inventory (bulk values become footprints)
- [ ] Assign per-item IDs in the 61xxx material bands
- [x] Near-miss hint library + authoritative solution/failure matrix (`JOURNAL-HINTS.md`, `DISCOVERY-SOLUTIONS.md`)

### First Survival + Crafting implementation sequence

- [ ] Active-run survival persistence + migration
- [ ] Fixed server survival tick + centralized tuning values
- [ ] Vigor field ceiling, walk recovery, Hustle/Short Burst/combat exertion
- [ ] Interruptible eat/drink/treatment channels
- [ ] Explicit Infection event API for plague wounds/contaminated consumables
- [ ] Ash Hollow once-per-run recovery
- [ ] Server-owned craft attempt engine + ingredient reservation
- [ ] Born-known recipes: Crude Boil, Roast, Bandage, Firestart
- [ ] Ash Tea near-miss/discovery chain
- [ ] Charcoal Pit + Charcoal milestone gate
- [ ] Filtered water + first Iron Ingot/knife chain
- [ ] Stitch Kit, Whetstone, Cord, Snare, Rendering/Tallow, Simple Stew
- [ ] Run all `SURVIVAL-CRAFTING-TEST-VECTORS.md` cases before expanding content

## Economy

- [ ] `ECONOMY.md` — tiered barter (no coin), material ladder, Monastery keepers
- [x] Corruption replaced by **Infection** (plague raises, Stitch/tincture cures)
- [x] Survival interlock — `SURVIVAL.md` (Vigor hub; meters erode Vigor; collapse-only fail)
- [x] Survival implementation contract — meter ranges, first-pass tuning, thresholds, channels, persistence (`SURVIVAL-IMPLEMENTATION-SPEC.md`)
- [ ] Gather satchel — allow-list, size + upgrade curve, lost-on-death
- [ ] Raw-meat spoil timer + preservation balance

## Journal / UI

- [x] Creature journal table + addon wire stub
- [x] Journal Record design — **Creatures / Survival / Gear** tabs, silhouettes/obscured knowledge, persistent near-miss hints (`JOURNAL-RECORD.md`)
- [x] Concrete Journal content/reveal map (`JOURNAL-CONTENT.md`)
- [x] Canonical hint library (`JOURNAL-HINTS.md`)
- [x] Authoritative discovery/solution matrix (`DISCOVERY-SOLUTIONS.md`)
- [x] Technical implementation contract — keys, persistence, sync, evaluator, security (`JOURNAL-IMPLEMENTATION-SPEC.md`)
- [x] UI/interaction contract — tabs, silhouettes, Field Notes, gear-tree behaviour (`JOURNAL-UI-SPEC.md`)
- [x] First playable slice + end-to-end acceptance scenarios (`JOURNAL-V1-SLICE.md`)
- [x] Data model, wire protocol, key registry, deterministic tests and coding plan

### First Journal implementation sequence

- [ ] Generic per-character discovery persistence + discovery service
- [ ] Versioned HELLO/READY + full Journal snapshot after login and `/reload`
- [ ] GM/debug Journal tooling (dump/grant/revoke/reset/force-sync)
- [ ] Build tabbed Journal shell in `ThalvaethUI` with loading state
- [ ] Render Creature tab: Unknown silhouette → Sighted → Engaged/observed
- [ ] Wire observed creature abilities into persistent journal keys
- [ ] Implement first Survival slice: born-known basics + Ashbloom/Ash Tea
- [ ] Implement deterministic near-miss evaluator + one-hint-per-attempt persistence
- [ ] Implement Charcoal discovery gate + filtration inference
- [ ] Implement Stitch path, Ashbloom/Gravecap layered material knowledge
- [ ] Implement one fishing catch (Eel), one Snare, Whetstone
- [ ] Render Gear slice with Crude Dagger → Iron Knife → Steel Blade and obscured branch sibling
- [ ] Implement one diagram-gated weapon branch that cannot be brute-forced
- [ ] Implement Stillstone + Deep-Lung Token as active/passive charm examples
- [ ] Run all `JOURNAL-V1-SLICE.md` acceptance scenarios, including random-spam resistance and character isolation
- [ ] Journal art polish only after functional discovery loop is proven

## Aptitudes & charms

- [x] Charm/aptitude items — charms grant aptitudes (2 slots; aptitude vs passive); craft from materials; invisible quality; upgradeable
- [ ] Confirm charm slot count; pick which proposed aptitudes (Night Eyes / Iron Gut / Deadened Step / Steady Hand / Second Wind) ship v1

## Client / branding

- [x] Brand concept set — splash, wordmark logo, circular seal, red-eyes emblem (`docs/branding/`)
- [x] `CLIENT-BRANDING.md` — login-reskin + patch-MPQ plan (concept-stage)
- [ ] Rebuild the wordmark in real weathered-serif type (verify "Thal'vaeth" spelling; esp. the seal's curved ring)
- [ ] Produce shippable assets (BLP export, login dimensions) + original login music
- [ ] Decide: crest-only vs crest + corner wordmark; blood-red vs ash-orange accent
- [ ] Build the login glue patch (hide 3D scene → static splash; strings/version/copyright)

## Core

- [ ] Spell strip + Remnant first login
- [ ] Port Hearthglen ↔ Rotwood entry
- [ ] Stress Director (full)
