# Survival runtime specification

This document converts the high-level survival design into a coding contract for Hunger, Thirst, Vigor, Infection, recovery, exertion, and run reset behaviour.

Related canon:
- [SURVIVAL.md](SURVIVAL.md)
- [APTITUDES.md](APTITUDES.md)
- [MAPS.md](MAPS.md)
- [ECONOMY.md](ECONOMY.md)
- [JOURNAL-IMPLEMENTATION-SPEC.md](JOURNAL-IMPLEMENTATION-SPEC.md)

---

## 1. Runtime ownership

The server owns all survival values. The addon/UI may display current values but must never be authoritative.

Per-character runtime state:
- hunger
- thirst
- vigor_current
- vigor_effective_max
- infection
- fevered flag
- field-recovery ceiling
- last survival tick time
- last exertion classification
- Ash Hollow used flag for the current run

Survival state must survive reconnects during an active run. Monastery reset behaviour is explicit and not equivalent to logout.

---

## 2. Normalized value ranges

Use normalized internal values for the first implementation:

- Hunger: 0–100 pressure, 0 = fed, 100 = severe hunger
- Thirst: 0–100 pressure, 0 = hydrated, 100 = severe thirst
- Infection: 0–100 pressure, 0 = clean, 100 = Fevered threshold
- Vigor: 0–100 base pool before cap penalties

Exact tuning constants belong in config, not hard-coded magic numbers.

Required config knobs:
- hunger base rate
- thirst base rate
- hustle multiplier
- combat multiplier
- heat multiplier
- hunger Vigor-regen penalty thresholds
- hunger Vigor-cap penalty thresholds
- thirst Vigor-cost multiplier thresholds
- infection Vigor-cap penalty thresholds
- infection passive Vigor drain thresholds
- Fevered threshold
- walk Vigor recovery rate
- field recovery ceiling
- Ash Hollow recovery amount / cap

---

## 3. Tick model

Survival updates on a fixed server cadence, not every frame.

Recommended v1 cadence: 1 second.

Each tick executes in this order:
1. determine context (home, run, safe pocket, dead, loading)
2. determine exertion state
3. accrue Hunger/Thirst
4. apply Infection over-time consequences
5. recalculate effective Vigor max
6. clamp current Vigor to effective max
7. apply permitted Vigor recovery
8. apply Fevered state if threshold crossed
9. emit deltas only if values changed meaningfully

No survival clock advances while the player is dead, loading, or in a non-gameplay transition.

---

## 4. Exertion classification

At any tick the Remnant is in one primary exertion band:

- Resting: seated/channeling survival action
- Walking: normal slow movement
- Hustling: fast travel/default run mode where enabled
- Fighting: active combat
- Burst: Short Burst or equivalent high-output aptitude

Precedence: Burst > Fighting > Hustling > Walking > Resting.

Hunger and Thirst accrual use the band multiplier. Fighting and Burst are intentionally expensive.

---

## 5. Hunger

Hunger is a pressure clock, not direct damage.

Effects by band:
- 0–39: no penalty
- 40–69: reduced field Vigor recovery
- 70–89: further reduced recovery + small Vigor cap loss
- 90–100: severe Vigor cap loss

Hunger must never directly kill the player.

Food reduces Hunger after a sit/channel completes. Interrupted eating consumes according to item/action policy defined in the crafting/action spec; it must not grant the full benefit before completion.

Cooked food outperforms raw food. Quality modifies restore amount but does not change the underlying recipe identity.

---

## 6. Thirst

Thirst accrues faster than Hunger and is more sensitive to exertion and heat.

Effects by band:
- 0–39: no penalty
- 40–69: modest Vigor action-cost increase
- 70–89: stronger Vigor cost increase
- 90–100: strong Vigor cost increase + stagger risk hook

The stagger hook should initially be a deterministic cooldown-based debuff trigger rather than random per-frame rolls.

Thirst never directly kills the player.

Drinking requires a vulnerable channel. Heat-capable future districts multiply accrual but do not require a new top-level meter.

---

## 7. Vigor

Vigor is the capacity-to-act pool.

Spent by:
- Hustle
- Short Burst
- aptitudes
- heavy attacks / future heavy actions

Recovered by:
- Walking: slow trickle up to field ceiling
- Ash Hollow: larger partial recovery once per run
- Monastery: full reset

Rules:
- Vigor cannot recover above effective max.
- Field recovery cannot exceed the field ceiling.
- Field ceiling itself cannot exceed effective max.
- Hunger/Infection cap penalties can push current Vigor down immediately via clamp.
- Empty Vigor disables Vigor-costing actions but does not kill.

Action rejection must be deterministic: if cost > current Vigor, the action does not start.

---

## 8. Infection

Infection comes primarily from plague wounds and contaminated survival actions.

Sources are event-based, not time-based by default:
- qualifying creature hit
- dirty wound closed without cleaning
- contaminated water/food consequence
- selected future environmental hazards

Infection bands:
- 0–39: no systemic penalty
- 40–69: mild effective Vigor max reduction
- 70–89: stronger cap reduction + passive Vigor pressure
- 90–99: severe cap reduction + fever warning state
- 100: Fevered

Fevered is a state, not instant death.

Fevered effects:
- strong slow
- reduced Vigor effective max
- increased Vigor costs
- blocks selected high-output actions if necessary for tuning

Cures reduce Infection only after their action completes. Wound closure and Infection treatment are separate concepts.

---

## 9. Wounds vs Infection

Do not collapse wound treatment and Infection into one number.

A Stitch Kit may:
- close a wound / stop bleeding
- prevent further wound-related consequences

It does not automatically erase all Infection.

Antiseptic / Ash Tea / tinctures reduce Infection according to their own effect definitions.

This separation is important for both crafting and Journal discovery.

---

## 10. Ash Hollow

Ash Hollow is a once-per-run recovery event.

On first valid use in a run:
- pause immediate danger pressure as designed
- grant partial Vigor recovery
- optionally pause Infection accrual while inside
- do not reduce Infection unless a separate treatment is used
- mark `ash_hollow_used = true`

Subsequent entries in the same run do not grant the recovery again.

Run start resets the flag.

---

## 11. Monastery reset

Returning alive to the Monastery and waking after death both end the active run.

On full home reset:
- Hunger -> 0
- Thirst -> 0
- Infection -> 0 for v1 unless later design adds lingering injury
- Fevered -> false
- Vigor -> base max
- effective Vigor max -> base max
- Ash Hollow used -> false
- exertion state -> resting

Inventory/haul consequences are handled by death/extraction systems, not by survival reset code.

---

## 12. Persistence and reconnects

If a player disconnects in Rotwood:
- preserve current survival values
- preserve run flags
- do not grant home reset

On reconnect:
- restore survival snapshot
- resume ticks once the player is fully in world

If the run itself is invalidated/reset by server policy, use explicit run-recovery logic rather than silently treating logout as extraction.

---

## 13. Network/UI update policy

The server should not spam addon messages every tick.

Send survival deltas when:
- an integer-visible value changes
- a band/state changes
- Fevered toggles
- effective max changes
- an action consumes or restores a significant amount

Initial login/run sync sends the full state.

---

## 14. Failure-safe rules

- clamp every value on write
- ignore negative restoration/cost definitions
- reject unknown effect keys
- never trust client-reported meter values
- never let duplicate completion callbacks apply an effect twice
- recovery actions must be idempotent per action instance

---

## 15. First tuning baseline

Numbers remain config-driven. For initial playtest only, use conservative bands and tune by run duration rather than theoretical realism.

Target feel:
- Thirst becomes relevant within a normal Rotwood run
- Hunger matters on longer/slower runs
- Infection is dominated by poor combat trades rather than passive time
- Vigor decisions are felt every segment
- a careful player can recover enough Vigor to continue, but cannot reset to full outside home

---

## 16. Acceptance criteria

The runtime is correct when:
- meters never directly kill
- severe neglect reliably degrades Vigor capability
- walking recovery stops at the field ceiling
- cap reductions clamp current Vigor correctly
- Fevered triggers only at the Infection threshold
- home reset restores clean baseline
- reconnecting mid-run preserves pressure
- duplicate ticks/actions cannot double-apply effects
- client spoofed values are irrelevant
