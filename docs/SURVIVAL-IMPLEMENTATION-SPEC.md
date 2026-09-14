# Survival implementation specification

This document turns the Survival canon into an implementation contract for Hunger, Thirst, Vigor and Infection.

Related canon:

- `SURVIVAL.md`
- `APTITUDES.md`
- `ECONOMY.md`
- `MAPS.md`
- `JOURNAL-IMPLEMENTATION-SPEC.md`
- `CRAFTING-IMPLEMENTATION-SPEC.md`

---

## 1. Core invariant

The survival system is not four independent death bars.

**Hunger, Thirst and Infection modify Vigor. Vigor collapse is the only systemic hard fail.**

No meter directly kills the player at a threshold. Instead, pressure removes the player's ability to move, evade, use aptitudes and survive the Wild.

The server owns all meter truth.

---

## 2. Canonical meter ranges

All four survival meters use normalized integer ranges internally for deterministic arithmetic.

| Meter | Internal range | Player meaning |
|---|---:|---|
| Hunger | 0–10000 | 0 = fed, 10000 = fully depleted/weak |
| Thirst | 0–10000 | 0 = hydrated, 10000 = fully dehydrated |
| Infection | 0–10000 | 0 = clean, 10000 = Fevered ceiling |
| Vigor | 0–10000 effective pool | 10000 = full Monastery-rested capacity |

The UI may render percentages or bars, but game logic should remain integer-based to avoid drift.

---

## 3. Update cadence

Survival simulation runs on a server-owned fixed cadence rather than frame time.

Recommended v1 cadence:

- evaluate accumulation every **1 second**;
- evaluate derived modifiers every **1 second**;
- send client updates only when values materially change or at a low heartbeat rate.

The cadence is an implementation constant, not a gameplay number exposed to the player.

---

## 4. Run state versus home state

Survival pressure only meaningfully runs while the Remnant is in a run district.

### Thal'vaeth Monastery

On valid return to the Monastery:

- Hunger → 0;
- Thirst → 0;
- Infection → 0;
- Vigor → 10000;
- Fevered clears;
- temporary field-only survival modifiers clear;
- Ash Hollow per-run recovery usage resets for the next run.

This is a **between-run full reset**, not a slow tavern regeneration simulation.

### Rotwood / run districts

While a run is active:

- Hunger and Thirst accumulate;
- Infection can accumulate from wounds/exposure;
- Vigor can be spent and only partially recovered;
- the field recovery ceiling applies;
- Ash Hollow can be used once per run.

---

## 5. Vigor model

Vigor has three relevant values:

1. **Base maximum** — 10000 at zero survival pressure.
2. **Effective maximum** — base maximum reduced by Hunger/Infection pressure and gear/environment modifiers.
3. **Current Vigor** — clamped to effective maximum.

### v1 default tuning

These values are explicitly tunable, but the first implementation needs fixed starting numbers.

| Setting | v1 value |
|---|---:|
| Base max Vigor | 10000 |
| Field walk-recovery ceiling | 6500 |
| Ash Hollow recovery target | 8000 |
| Walk recovery | +18 / sec |
| Hustle drain | -45 / sec |
| Short Burst drain | -180 / sec |
| Generic combat exertion surcharge | -25 / sec while actively fighting |

Walk recovery stops once current Vigor reaches the current field ceiling or effective maximum, whichever is lower.

Ash Hollow can raise current Vigor above the normal walk ceiling, but not above its own target or the effective maximum.

### Collapse

At current Vigor = 0:

- Hustle cannot start;
- Short Burst cannot start;
- Vigor-spending aptitudes cannot start;
- the player remains alive;
- movement/combat remains possible at baseline engine levels unless specifically modified by Fevered or another debuff.

The game should never implement an automatic `KillPlayer()` equivalent at zero Vigor.

---

## 6. Hunger accumulation and effects

### v1 accumulation

Base Hunger accumulation: **+6 / sec** during a run.

Multipliers/adders:

- Hustle: +5 / sec additional;
- active combat: +4 / sec additional;
- Short Burst: +8 / sec additional while active.

This puts a completely unmanaged Hunger clock in the rough order of tens of minutes rather than minutes.

### v1 thresholds

| Hunger | State | Effect |
|---:|---|---|
| 0–3999 | Fed/normal | no penalty |
| 4000–6999 | Hungry | walk Vigor recovery ×0.80 |
| 7000–8999 | Weak | walk Vigor recovery ×0.50; effective Vigor max ×0.90 |
| 9000–10000 | Starving | walk Vigor recovery disabled; effective Vigor max ×0.75 |

### Food restore classes

| Food class | Hunger reduction | Channel |
|---|---:|---:|
| Raw emergency food | 900 | 4 sec |
| Basic cooked food | 2200 | 5 sec |
| Preserved ration | 1800 | 4 sec |
| Hearty meal | 3200 | 7 sec |

Quality can scale these values later; the recipe succeeds independently of quality.

---

## 7. Thirst accumulation and effects

### v1 accumulation

Base Thirst accumulation: **+9 / sec** during a run.

Additional pressure:

- Hustle: +7 / sec;
- active combat: +6 / sec;
- Short Burst: +10 / sec;
- hot environment flag: base accumulation ×1.5.

Thirst intentionally bites faster than Hunger.

### v1 thresholds

| Thirst | State | Effect |
|---:|---|---|
| 0–3499 | Hydrated | no penalty |
| 3500–6499 | Thirsty | all Vigor spend ×1.10 |
| 6500–8499 | Dry | all Vigor spend ×1.25 |
| 8500–10000 | Dehydrated | all Vigor spend ×1.45; periodic brief stagger allowed |

The stagger is a soft pressure effect only. It must never directly deal lethal damage.

### Water restore classes

| Drink | Thirst reduction | Notes |
|---|---:|---|
| Foul Water | 1800 | carries Infection risk |
| Boiled Water | 2600 | born-known field-safe treatment |
| Filtered Clean Water | 3200 | no Infection risk |
| Distilled Water | 3400 | no Infection risk; advanced |
| Broth | 1600 | also minor Hunger reduction |

---

## 8. Infection accumulation and effects

Infection is primarily combat/exposure driven, not a passive clock.

### Sources

| Source | Infection gain |
|---|---:|
| Minor plague wound | +180 |
| Heavy plague wound | +350 |
| Special infected ability | +450–700, per ability definition |
| Drinking Foul Water | +250 risk event |
| Failed contaminated medical treatment | +150–400, recipe-specific |

The creature ability or event explicitly decides whether a hit qualifies. Do not infer Infection from all damage universally.

### v1 thresholds

| Infection | State | Effect |
|---:|---|---|
| 0–2999 | Clean/low | no penalty |
| 3000–5999 | Tainted | effective Vigor max ×0.95 |
| 6000–8499 | Sick | effective Vigor max ×0.80; field recovery ×0.75 |
| 8500–9999 | Severe | effective Vigor max ×0.60; field recovery ×0.40 |
| 10000 | Fevered | Fevered debuff; effective Vigor max ×0.45; movement pressure/weakness |

Fevered is the near-death survival state, not instant death.

### Infection treatment

| Treatment | Infection reduction | Channel |
|---|---:|---:|
| Stitch Kit alone | 300 | 6 sec |
| Ash Tea | 900 | 5 sec |
| Antiseptic Wash | 700 | 5 sec |
| Fever Tincture | 1400 | 6 sec |
| Full Monastery reset | all | between runs |

A Stitch Kit is not a magical heal.

---

## 9. Cross-meter calculation order

Each survival tick must use a stable order:

1. apply raw Hunger/Thirst accumulation;
2. process Infection events queued since last tick;
3. clamp meters to 0–10000;
4. calculate Hunger modifiers;
5. calculate Thirst modifiers;
6. calculate Infection modifiers;
7. calculate gear/environment/charm modifiers;
8. derive effective Vigor maximum;
9. clamp current Vigor to effective maximum;
10. apply current action's Vigor spend/recovery;
11. clamp current Vigor;
12. evaluate Fevered/stagger state transitions;
13. emit delta if needed.

---

## 10. Survival consumption channels

Eating, drinking and treatment use a common interruptible action model.

A survival-use channel:

- starts only if the player is alive and owns the item;
- reserves the item but does not consume it immediately;
- is interrupted by movement, entering combat, meaningful incoming damage, logout or map transfer;
- consumes the item only on successful completion;
- applies the survival effect atomically with consumption.

Interrupted use returns the reserved item unchanged for v1 unless a specific item later defines partial consumption.

---

## 11. Ash Hollow

Ash Hollow is a once-per-run recovery pocket.

On first valid use in a run:

- set current Vigor to `max(current, min(8000, effectiveMax))`;
- pause Infection accumulation from passive/environment sources while inside;
- do not cure existing Infection by default;
- mark `ash_hollow_used = true` for the current run.

Subsequent visits in the same run provide shelter but no second Vigor jump.

Combat breaks the safe-pocket recovery state.

---

## 12. Death interaction

On death:

- haul/satchel loss is handled by the death/extraction system;
- survival values do not need to remain meaningful through ghost state;
- after morgue return, the Monastery reset contract applies.

No corpse-run survival simulation is required.

---

## 13. Persistence policy

For v1, survival meters are **run-state**, not long-term character progression.

Persist only what is needed for crash/reconnect correctness during an active run:

- run identifier/state;
- Hunger;
- Thirst;
- Infection;
- current Vigor;
- Ash Hollow used flag;
- active Fevered state if derivation cannot safely reconstruct it.

Normal extraction/death/home reset clears the run-state persistence.

---

## 14. Configuration knobs

Centralize all first-pass tuning numbers:

- base Hunger/Thirst rates;
- exertion additions;
- thresholds;
- field ceiling;
- walk recovery;
- Vigor action costs;
- Infection event amounts;
- item restore amounts;
- Ash Hollow target.

Do not scatter tuning numbers through AI, item and player scripts.

---

## 15. Acceptance criteria

The first Survival implementation is acceptable when:

1. Hunger and Thirst accumulate only during a run.
2. Hustle/combat materially accelerate both clocks.
3. Hunger changes Vigor recovery/cap but never directly kills.
4. Thirst increases Vigor spend but never directly kills.
5. Infection is event-driven and only specific events raise it.
6. Infection reduces effective Vigor maximum and can produce Fevered.
7. Walking cannot recover above the field ceiling.
8. Ash Hollow works once per run.
9. Monastery return resets all four meters.
10. item-use channels are server-authoritative and interruptible.
11. logout/reconnect does not reset an active run's survival state.
12. zero Vigor does not itself kill the player.
13. all calculations are deterministic under test vectors.

---

## 16. Explicit non-goals for first implementation

Do not add yet:

- temperature as a fifth meter;
- morale/sanity;
- vitamin/nutrient simulation;
- per-limb wounds;
- sleep/fatigue;
- permanent survival debuffs after extraction;
- client-authoritative meter prediction.