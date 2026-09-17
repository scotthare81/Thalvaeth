# Gear upgrade graph — first implementation contract

This file converts the broad ladders in `GEAR.md` into stable graph semantics suitable for coding. Names and exact coefficients remain tunable; graph identity and gating must be stable once persisted.

## 1. Rules

- A node is a persistent progression state, not a rarity.
- An edge is an allowed upgrade transition.
- Every edge declares station, knowledge gate, diagram policy and material family requirements.
- Nodes may branch. Branching is a player build choice; do not silently auto-select by highest stat.
- A later respec/rebuild system is not implied by this graph.

## 2. Weapon roots

Starting persistent weapon:

`gear.weapon.crude_dagger`

Early transition family:

```text
gear.weapon.crude_dagger
  ├─ gear.weapon.bone_blade
  └─ gear.weapon.flint_blade
        ↓
gear.weapon.iron_short_blade
        ↓
gear.weapon.steel_blade
        ├─ fast branch
        └─ heavy branch
```

Bone and flint are alternative pre-forge improvements and converge on the first iron form. Implementation must support converging source nodes.

## 3. Fast branch

Suggested stable nodes:

```text
gear.weapon.steel_blade
  → gear.weapon.fast.fighting_dagger
      → gear.weapon.fast.paired_daggers
          → gear.weapon.fast.twin_blades

gear.weapon.steel_blade
  → gear.weapon.fast.shortsword
      → gear.weapon.fast.steel_shortswords
          → gear.weapon.fast.twin_blades_sword
```

This deliberately allows dagger and sword identities to remain distinct where butcher capability matters.

Fast-line design contract:

- lower per-hit impact than heavy line;
- lower Vigor/action burden;
- faster cadence/mobile feel;
- bleed hooks where authored;
- dagger nodes retain butcher capability;
- carried spare bulk follows Inventory data, not this graph.

## 4. Heavy branch

Suggested nodes:

```text
gear.weapon.steel_blade
  → gear.weapon.heavy.war_cleaver
      → gear.weapon.heavy.steel_greataxe
          → gear.weapon.heavy.masterwork_greataxe

gear.weapon.steel_blade
  → gear.weapon.heavy.iron_greatblade
      → gear.weapon.heavy.steel_greatsword
          → gear.weapon.heavy.masterwork_greatsword
```

Heavy-line design contract:

- high impact/reach/stagger or cleave;
- high Vigor/action burden;
- higher noise;
- greatsword has no default butcher/chop capability;
- greataxe may provide chopping capability;
- cleaver may provide heavy-butcher capability;
- dedicated skinning knife remains relevant for heavy builds.

## 5. Material profile is separate from node

Where technically practical, node identity describes form/progression and material profile describes metal choice. This avoids duplicating every node into iron/steel/bronze variants.

If a node requires a specific metal for lore/progression (for example the first Steel Blade), the edge definition enforces that material. Later authored side-grade rebuilding can alter a material profile only if explicitly supported.

## 6. Armour graph

Body progression roots:

```text
gear.armour.body.rag
  → gear.armour.body.quilted
      ├─ gear.armour.body.leather.boiled
      │   → gear.armour.body.leather.studded
      │       → gear.armour.body.leather.hardened
      └─ gear.armour.body.mail.ring
          → gear.armour.body.mail.riveted
              → gear.armour.body.mail.splinted
                  → gear.armour.body.plate.half
                      → gear.armour.body.plate.full
```

This graph represents progression availability, not a claim that the player must abandon lighter endgame builds. Hardened Leather and Splinted Mail remain valid terminal build destinations even when Plate knowledge exists.

Mail/plate dependencies on quilted underlayer must be represented as equipment/layer requirements rather than consuming the quilted piece into nothing unless a specific recipe explicitly does so.

## 7. Matching slots

Head/hands/legs/feet/back upgrades should use their own stable graphs and material requirements rather than being generated automatically from body tier. The body establishes dominant armour class; mixed pieces remain legal and contribute their own effects.

V1 coding does **not** need every matching-slot graph. Prove the system with body armour plus one supporting slot before content expansion.

## 8. Diagram policy

A graph edge declares one of:

- `born_known` — available from start if process/station requirements are met;
- `discoverable` — can be learned through the documented experimentation/knowledge system;
- `diagram_required` — cannot be brute-forced; requires persistent diagram knowledge.

Late weapon branch commitment and major armour tier jumps should default to `diagram_required` unless a design document explicitly says otherwise.

## 9. Station policy

Canonical station keys must use the same `station.*` namespace as Crafting implementation canon. Gear code must not introduce bare duplicate station identifiers.

Expected families:

- `station.cookfire` etc. are irrelevant to gear;
- metal weapon/armour upgrades use the canonical Forge station key;
- cloth/leather upgrades use canonical Stitch Table key;
- tools/packs use canonical Workbench where appropriate.

Before code, verify exact key spelling against the final Crafting registry and use that value everywhere.

## 10. V1 graph proof

First implementation only needs enough graph to prove branching, gating, durability and persistence:

1. `Crude Dagger` persistent root;
2. one pre-forge blade transition;
3. `Iron Knife / Short Blade`;
4. `Steel Blade`;
5. one fast branch child;
6. one heavy branch child;
7. `Rag Armour → Quilted Coat → Boiled Leather`;
8. one diagram-required transition;
9. one material-quality-sensitive upgrade;
10. repair/sharpen across at least Fine/Worn/Damaged/Broken.

Do not author the entire endgame item catalogue before these transitions pass the deterministic test suite.