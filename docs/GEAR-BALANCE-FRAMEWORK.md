# Gear balance framework

This document defines the **shape** of Gear balance without prematurely locking final combat numbers. It exists so weapon, armour, durability and material tuning can be implemented as data rather than scattered special cases.

## 1. Design objective

Gear should change **how the Remnant survives**, not simply increase a power score.

Every meaningful equipment choice should trade across several pressures:

- combat effectiveness;
- Vigor economy;
- noise/exposure;
- harvesting/tool convenience;
- protection from wounds/Infection;
- durability/upkeep;
- carried bulk when the item is not worn;
- material and repair cost.

There is no item level and no rarity ladder.

## 2. Weapon profile axes

Every weapon form defines these independent axes:

| Axis | Meaning |
|---|---|
| Impact | Damage/force delivered by a successful strike |
| Cadence | How quickly another attack can be committed |
| Reach | Engagement distance / spacing advantage |
| Vigor burden | Vigor spent to attack/use the weapon |
| Noise | Director/noise contribution from use |
| Stagger | Ability to interrupt or destabilise suitable targets |
| Cleave | Ability to threaten more than one target where authored |
| Bleed | Wound/bleed pressure where authored |
| Utility | Butcher/chop/etc. capability |
| Edge retention | Resistance to ordinary blade wear |
| Structure | Maximum durability / resistance to heavy wear |
| Carried bulk | Inventory cost when not worn/wielded |

A weapon definition must not collapse these into a single `power` number.

## 3. Weapon identity targets

### Crude Dagger

The baseline survival tool. Weak reach and impact, but low Vigor, quiet, cheap to carry and capable of butchering. It remains conceptually useful even after combat upgrades because knife utility matters.

### Flint/Bone Blade

A pre-forge improvement with a genuinely sharp working edge but poor structural life. It should feel like a clever survival improvement, not a disposable grey-quality item.

### Iron Knife / Short Blade

First dependable forged weapon. Moderate durability and reliable repair materials. It establishes the normal forged baseline.

### Steel Blade

The shared high-quality early/mid-game blade: better edge retention and dependable combat performance. This is the branch point, not automatically the final answer.

### Fast line

Identity: initiative, mobility, lower commitment, bleed and harvest convenience. It should tolerate repeated small engagements well but require more successful contacts to solve a dangerous enemy.

### Heavy line

Identity: commitment, spacing, impact, stagger/cleave and risk. Misses or poorly timed attacks should hurt the Vigor economy. Heavy weapons are intentionally noisy and inconvenient as carried spares.

## 4. Provisional relative weapon bands

These are qualitative fixture bands for implementation/testing, not shipping values.

| Form | Impact | Cadence | Reach | Vigor | Noise | Upkeep |
|---|---|---|---|---|---|---|
| Crude Dagger | Low | Fast | Short | Very low | Very low | Medium |
| Flint/Bone | Low–Med | Fast | Short | Low | Low | High |
| Iron short blade | Medium | Medium-fast | Short–Med | Low–Med | Low | Medium |
| Steel Blade | Medium+ | Medium | Medium | Medium | Medium-low | Low |
| Fighting dagger | Medium | Fast | Short | Low | Low | Medium |
| Paired/twin fast | Medium | Very fast | Short–Med | Medium cumulative | Medium | Medium |
| War cleaver | High | Slow | Medium | High | High | Medium |
| Greatsword | Very high | Slow | Long | Very high | High | Medium-low |
| Greataxe | Very high | Slow | Long | Very high | Very high | Medium |

The important part is the **shape**: a Greatsword must not become a Steel Blade with bigger damage and no consequences.

## 5. Armour profile axes

Every armour piece can contribute:

- physical mitigation;
- wound severity reduction where appropriate;
- Infection exposure/resistance modifier;
- movement/Vigor burden;
- noise;
- warmth/weather handling where that slot supports it;
- footing/wet handling for boots/waders;
- carried bulk when spare;
- durability/upkeep.

The body establishes dominant class for presentation and broad build identity, but effects derive from the actual worn pieces.

## 6. Armour identities

### Rags / Cloth

Nearly no direct protection. Quiet, cheap, no meaningful Vigor tax. Quilted cloth later becomes useful protection and the underlayer for heavier armour.

### Leather

The mobile survival line. Meaningful protection without destroying stealth/Vigor economy. Hardened Leather is a valid endgame destination, not a temporary stepping stone.

### Mail

The balanced line. More reliable protection and Infection resistance with audible movement and increased exertion. Splinted Mail is a valid endgame compromise.

### Plate

The stand-and-fight line. Strongest mitigation and wound/Infection protection, but loud, Vigor-hungry and costly to maintain. It should make some encounters safer while making avoidance and long exertion harder.

## 7. Provisional relative armour bands

| Class | Protection | Infection/wound protection | Vigor burden | Noise | Upkeep |
|---|---|---|---|---|---|
| Rags | Minimal | Minimal | None | Minimal | Low |
| Quilted | Low | Low | Very low | Very low | Low |
| Leather | Low–Med | Low–Med | Low | Low | Low–Med |
| Hardened Leather | Medium | Medium | Low–Med | Low | Medium |
| Ring/Riveted Mail | Medium–High | Medium | Medium | Medium–High | Medium |
| Splinted Mail | High | Med–High | Medium–High | High | Medium–High |
| Half Plate | Very high | High | High | High | High |
| Full Plate | Highest | Highest | Very high | Very high | High |

These bands express intended relationships only.

## 8. Condition effects

Condition should matter before zero without making ordinary wear feel like constant punishment.

Recommended behaviour shape:

- **Fine:** full authored profile.
- **Worn:** small upkeep warning; mild edge/protection degradation at most.
- **Damaged:** clearly felt degradation; the player should want to repair before a serious run.
- **Broken:** severe impairment, but item persists and can be recovered at home.

Do not make Worn equivalent to “bad item.” The purpose is to create maintenance decisions, not constant stat anxiety.

For a weapon, condition may affect impact/edge retention/utility effectiveness before it affects swing animation or reach. For armour, condition may reduce protection/Infection resistance before changing Vigor/noise; a damaged breastplate does not become lighter or quieter because it is damaged.

## 9. Wear magnitude classes

Use semantic wear classes so content can tune without direct durability subtraction:

- `wear.light` — ordinary cutting/harvesting/low-stress contact;
- `wear.standard` — normal successful combat/tool event;
- `wear.heavy` — high-impact attack, hard material/tool action;
- `wear.severe` — exceptional authored event/environmental damage.

The gear/material profile converts wear class into integer durability loss. This allows steel and bronze to react differently to the same event.

## 10. Material behaviour

### Iron

Baseline forged material. Common enough to repair without trivialising cost. Reliable structure, ordinary edge retention.

### Steel

Best conventional blade edge retention and strong structure. More expensive knowledge/material chain. Its advantage should be fewer maintenance interruptions as much as raw combat improvement.

### Bronze

A deliberate side-grade: lighter action burden but softer/faster wearing. It should appeal to Vigor-sensitive builds, not exist as an inferior curiosity.

Material modifiers must remain bounded so weapon form matters more than metal gimmicks.

## 11. Repair economy

Repairs should consume enough extracted materials to matter but not enough to make persistent gear feel disposable.

Principles:

- Worn maintenance is cheap;
- Damaged structural repair is meaningful;
- Broken recovery is expensive enough to sting but always realistically recoverable;
- higher tiers require more refined materials, not arbitrary money;
- field sharpening buys time, not infinite self-sufficiency;
- repair never rerolls quality;
- v1 repair never permanently lowers max durability.

## 12. Upgrade economy

An upgrade competes with survival consumables, barter and future crafting. The player should sometimes extract with enough material for an upgrade but choose to spend it elsewhere.

Major branch transitions should therefore require a **small number of meaningful material families**, not dozens of filler components. The friction should come from obtaining/choosing materials and knowledge, not inventory busywork.

## 13. Anti-dominance checks

During balance review, reject a design if:

- one weapon wins impact, Vigor, noise, utility and durability simultaneously;
- heavy weapons have no practical Vigor/noise downside;
- dual daggers equal heavy stagger/impact while retaining harvest convenience;
- plate can sneak/run as efficiently as leather;
- Hardened Leather becomes obsolete the instant Mail/Plate is known;
- bronze is simply worse steel;
- Broken is so punitive that the player would rather replace the item;
- repairs are so cheap that durability can be ignored;
- repairs are so expensive that using good gear feels irrational.

## 14. Telemetry/debug data worth collecting

For playtest builds record aggregate/debug values for:

- attacks per weapon per run;
- Vigor spent by weapon family;
- wear incurred per run;
- repairs/sharpens per extraction cycle;
- deaths by armour class;
- damage mitigated by armour class;
- material cost spent on repair vs upgrade;
- percentage of run time in Fine/Worn/Damaged/Broken;
- frequency of dedicated knife carried by heavy builds;
- branch selection rates.

This is tuning evidence, not player-facing analytics.

## 15. Balance lock order

Tune in this order:

1. weapon/armour **identity**;
2. Vigor/noise tradeoffs;
3. protection/impact relationships;
4. durability lifetime;
5. repair costs;
6. material side-grades;
7. exact damage/mitigation numbers.

Do not begin by spreadsheeting DPS and accidentally erase the survival-game decisions.