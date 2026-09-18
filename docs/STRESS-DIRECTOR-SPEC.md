# Stress Director specification

## Purpose

The Director shapes **pacing**, not fairness by cheating. It observes the run and chooses when to add, withhold or redirect pressure through authored opportunities.

It must make Rotwood feel reactive without becoming an omniscient dungeon master.

## Three inputs

### Stress
Short-term pressure on the player: recent engagements, pursuit, low breathing room, survival strain and nearby active danger.

### Heat
Accumulated disturbance/attention created by the player's behaviour: loud combat, repeated heavy noise, creature calls, conspicuous actions and authored events.

### Depth
Authored danger context of the player's current Rotwood zone/segment. Depth is geography/progression context, not elapsed-time punishment.

These are server-only tuning values. They are not player-facing meters.

## Director state

Conceptual pacing states:
`CALM → BUILDING → PRESSURE → PEAK → RELEASE → CALM`

Transitions use hysteresis/cooldowns. The Director must deliberately create release windows. Constant pressure destroys horror pacing.

## Inputs

The Director may consume semantic events:
- noise pressure;
- confirmed combat/engagement;
- creature calls/deaths;
- prolonged pursuit;
- zone/depth transition;
- survival state bands where explicitly approved;
- extraction attempt;
- elapsed quiet time;
- run history.

It does not read player intent or hidden future choices.

## Permitted outputs

Through authored eligible content only, the Director may:
- activate a dormant pressure slot;
- choose among eligible future creature/event opportunities;
- delay/withhold activation;
- schedule distant audio/environmental pressure;
- alter patrol/pressure opportunities where authored;
- select a Ghoul/Runner-style future pressure event;
- create deliberate quiet;
- modify later encounter selection within budget.

## Forbidden outputs

The Director may not:
- give individual creatures exact hidden player coordinates;
- set all creatures ENGAGED;
- spawn enemies immediately behind/in sight of the player without an authored legal rule;
- violate spawn geometry or special-creature contracts;
- resurrect killed creatures invisibly;
- refill/reset creature health;
- change persistent player gear/survival inventory;
- fabricate Journal knowledge;
- guarantee punishment because the player is succeeding;
- endlessly escalate without release.

## Pressure budget

Director actions have authored costs and cooldowns. A run/zone has pressure capacity. Existing active threats count against it.

Before activation:
1. determine current pacing state;
2. collect eligible authored opportunities;
3. reject illegal visibility/proximity/geometry states;
4. reject budget/cooldown conflicts;
5. choose deterministically from run RNG stream;
6. commit event once;
7. record mutation;
8. publish only legitimate world stimuli.

## Randomness

Director RNG is derived from run state/seed with its own stream so unrelated systems do not perturb generation determinism. Decisions are traceable.

## Noise relationship

Unified NoiseService can feed both nearby AI and Director:
- AI receives positional evidence according to hearing;
- Director receives aggregate pressure/zone event.

Director pressure never loops back into exact AI target knowledge.

## Extraction

Extraction may be an authored pressure moment, but the Director cannot automatically manufacture an unavoidable ambush. Extraction rules define eligible responses and safety constraints.

## Death/reconnect

Director state is run-local. Reconnect to the same active run cannot reset heat/pressure for free. Death/extraction destroys Director state with the run.

## Debug

`.thal director state`, `.thal director heat`, `.thal director stress`, `.thal director opportunities`, `.thal director trace on|off`.

Trace every input, state transition, candidate rejection, selected action, cost and cooldown.

## Acceptance

A successful Director produces recognizable tension waves, legal pressure, meaningful quiet, deterministic debug replay and zero omniscient AI leakage.