# Gear progression content matrix

This is the authoring bridge between the upgrade graph and future concrete recipes/item IDs. It identifies what each progression step is supposed to demand and teach without locking final quantities before materials/station balance is tested.

## 1. Shared weapon progression

| From | To | Gate | Station | Material intent | Knowledge intent |
|---|---|---|---|---|---|
| Crude Dagger | Flint Blade | basic fieldcraft | Workbench/field if approved | flint + binding + suitable haft/grip | discoverable early |
| Crude Dagger | Bone Blade | butchery/material knowledge | Workbench | good bone + binding + grip | discoverable early |
| Flint/Bone Blade | Iron Knife / Short Blade | Charcoal + Smelting/Iron | Forge | iron stock + haft/grip + binding/rivets | first real forge milestone |
| Iron Knife / Short Blade | Steel Blade | Steelworking | Forge | steel stock + improved haft/grip + refined fittings | major process gate |
| Steel Blade | first fast form | branch diagram | Forge/Workbench as authored | refined blade stock + grip/fittings | diagram-required commitment |
| Steel Blade | first heavy form | branch diagram | Forge | more metal + heavy haft/fittings | diagram-required commitment |

Exact quantities must come from the Materials/Crafting economy pass. Do not invent filler ingredients just to increase recipe length.

## 2. Fast branch content

### Fighting Dagger

Purpose: first explicit combat-specialised dagger while retaining butcher utility.

Expected requirements:

- forged blade stock;
- compact reinforced grip;
- refined binding/rivets;
- fast-branch diagram;
- appropriate metalworking knowledge.

Expected learned consequence: low commitment and utility remain strengths; this is not simply “Steel Blade + damage.”

### Paired Daggers

Purpose: introduce dual-weapon identity.

Important implementation question before content lock: whether one persistent gear instance represents the **pair/set** or whether two persistent instances occupy main/off-hand. V1 should prefer a single logical paired-weapon gear instance if the combat implementation treats the pair as one authored form; otherwise persistence, repair and upgrade transactions must explicitly support two linked instances. Do not decide this accidentally in item templates.

### Shortsword path

Keeps fast style without butcher capability. Gives fast-build players a combat-focused alternative to daggers and preserves the knife-as-extra-tool tension.

## 3. Heavy branch content

### War Cleaver

Purpose: transitional heavy weapon with butcher-heavy utility. It should feel improvised/practical rather than a polished fantasy battleaxe.

### Iron Greatblade

Purpose: combat-focused heavy transition. Better reach/impact, no free harvesting capability.

### Steel Greatsword

Purpose: maximum clean heavy-blade combat identity. High Vigor/noise; long reach; no butcher/chop shortcut.

### Greataxe

Purpose: heavy combat plus woodcutting convenience. Its utility is paid for through weight/noise/upkeep rather than making it universally superior to Greatsword.

## 4. Armour progression

| From | To | Gate | Station | Material intent | Role |
|---|---|---|---|---|---|
| Rag Armour | Quilted Coat | stitching/padding knowledge | Stitch Table | cloth/rags + padding + thread | quiet underlayer/light protection |
| Quilted Coat | Boiled Leather | Tanning | Stitch Table/Workbench per station canon | cured leather + cord + fittings | mobile protection |
| Boiled Leather | Studded Leather | metal fittings knowledge | Stitch Table/Workbench | leather + rivets/studs | stronger leather |
| Studded Leather | Hardened Leather | advanced tanning/lamellar diagram | Workbench/Stitch | heavy leather + bone/tusk plates + bindings | light-line endgame |
| Quilted base + iron work | Ring Mail | Smelting/Iron | Forge + assembly station policy | rings + leather straps; quilted underlayer retained | balanced armour entry |
| Ring Mail | Riveted Chain | wire/riveting knowledge | Forge | drawn wire/rings + rivets | improved mail |
| Riveted Chain | Splinted Mail | diagram | Forge | mail + iron splints + straps | mail endgame |
| Mail + quilted base | Half Plate | Steelworking + diagram | Forge | steel plates + straps/fittings | heavy entry |
| Half Plate | Full Plate | late diagram/masterwork process | Forge | fitted steel plate components | heavy endgame |

The exact station split for leather/assembly must be normalized with `STATION-CONTRACTS.md` before code.

## 5. Supporting-slot progression

Supporting pieces should create survival choices rather than duplicate body mitigation.

### Head

Cowl/Hood axes: noise, modest protection, weather/light interaction. Heavy headgear can improve protection but should worsen hearing/noise/comfort only where those systems exist; do not invent unsupported penalties.

### Hands

Wraps/Gloves axes: grip, butcher/tool efficiency, protection and mend handling. Any yield bonus must be bounded so gloves do not become mandatory farming gear.

### Feet

Boots axes: noise, footing, wet handling and protection. This is a strong place for district-specific equipment decisions.

### Legs/Waders

Waders are environmental fit first, armour second. They can protect against wet terrain/exposure without becoming a universal best leg item.

### Back/Pack

Pack progression affects main-bag/satchel capacity and therefore couples directly to Inventory. Capacity upgrades must be server-owned and cannot be faked by equipping a client item template.

### Warmth layer

Shirt/tabard/neck remain independent of armour class. They should not be pulled into the main mitigation ladder merely because WotLK exposes equipment slots.

## 6. Repair material families

Repairs should resemble the thing being repaired:

| Gear | Repair family |
|---|---|
| Cloth/quilted | cloth/rags + thread/padding |
| Leather | leather scraps + cord/thread/fittings |
| Mail | rings/wire/rivets + straps |
| Plate | plate/scrap metal + rivets/straps |
| Blade edge | whetstone for field edge maintenance |
| Metal weapon structure | matching/relevant metal stock + haft/fittings as needed |
| Bone/flint | bindings/grip + replacement edge material; limited field repair only if authored |
| Pack | leather/cloth + straps/buckles |

The server recipe defines exact inputs. The table is semantic guidance.

## 7. Diagram distribution principles

Diagrams replace conventional gear drops. Distribution therefore needs restraint.

- Early survival improvements can be learned experimentally.
- Major forged tiers should require process knowledge.
- Build-defining branch transitions should use diagrams.
- Late/endgame forms should use district/depth-specific diagrams or Keeper knowledge.
- A diagram is persistent knowledge, not a consumable reroll token.
- Duplicate diagrams need a defined outcome before they enter loot tables: either do not drop once known, become barter/salvage, or have another explicit use. Never silently clog the haul with dead knowledge items.

## 8. Discovery vs construction

Knowing an upgrade and being able to build it are different.

Example:

- player finds heavy-branch diagram;
- Journal reveals/infer appropriate heavy form according to discovery state;
- player still lacks Steelworking or enough refined metal;
- station can show the known dependency without granting it;
- acquiring steel alone never teaches the diagram.

## 9. Material substitution

Substitutions are authored, not fuzzy crafting guesses.

If an Iron Knife edge accepts more than one grade/source of iron stock, the definition says so. If bronze is a side-grade material for a later form, that is a distinct legal material profile. The system must not automatically accept any item tagged `metal`.

## 10. Quality propagation

An upgrade transforms the same persistent gear identity but may recompute the new form's hidden quality according to the common Crafting quality policy.

Pre-code decision required: how much previous workmanship quality carries into a major rebuild versus how much new material quality determines the result. Whatever rule is chosen must be deterministic at commit and cannot permit reroll farming.

A sensible model to evaluate in playtesting:

`result quality = weighted prior workmanship + weighted new material quality + bounded craft roll`

This is a candidate model, **not locked arithmetic**.

## 11. Salvage/breakdown boundary

Persistent equipped gear is not automatically salvaged at Broken. If the player later chooses to dismantle an obsolete persistent form at home, that must be an explicit destructive action with confirmation and an authored salvage return.

V1 should omit voluntary destruction of core persistent gear until replacement/identity consequences are designed.

## 12. Content authoring checklist

Before adding a new gear node, answer:

1. What survival/combat decision does it create?
2. What source node(s) reach it?
3. Is it experimental, process-gated or diagram-required?
4. Which canonical station performs it?
5. Which meaningful material families does it consume?
6. What material variants are legal?
7. What capabilities does it explicitly add/remove?
8. What is its carried bulk?
9. What wear/repair family applies?
10. What remains attractive about its alternatives?
11. What safe information can Journal/UI reveal before/after discovery?
12. How does it migrate if the definition changes later?

If those answers are missing, the node is not implementation-ready.