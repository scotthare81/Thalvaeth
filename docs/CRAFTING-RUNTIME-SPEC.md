# Crafting runtime specification

This document defines how crafting attempts execute at runtime. Discovery meaning and hint text remain governed by the Journal canon; this file defines input validation, station context, attempt evaluation, consumption, success, quality, and output delivery.

Related:
- [CRAFTING.md](CRAFTING.md)
- [DISCOVERY-SOLUTIONS.md](DISCOVERY-SOLUTIONS.md)
- [JOURNAL-HINTS.md](JOURNAL-HINTS.md)
- [JOURNAL-IMPLEMENTATION-SPEC.md](JOURNAL-IMPLEMENTATION-SPEC.md)
- [SURVIVAL-RUNTIME-SPEC.md](SURVIVAL-RUNTIME-SPEC.md)

---

## 1. Server authority

Crafting is server-authoritative. The client may submit an attempt description, never a result.

A runtime attempt contains:
- player GUID
- station key
- station instance/context
- ordered or unordered ingredient multiset as required by recipe family
- selected process/action
- optional tool key
- optional target item for repair/upgrade
- attempt nonce/action id
- timestamp/context snapshot

The server resolves:
- eligibility
- candidate recipes
- failure precedence
- hint award
- input consumption
- success result
- quality
- discovery grants

---

## 2. Station keys

Canonical station keys:
- `field`
- `butcher_block`
- `cookfire`
- `stitch_table`
- `ash_still`
- `charcoal_pit`
- `forge`
- `workbench`

A recipe may allow more than one station with different output/quality caps, but station compatibility is explicit.

Field crafting is a restricted station, not a bypass around station rules.

---

## 3. Attempt lifecycle

Each attempt proceeds in this order:
1. validate player/context
2. validate station and range
3. validate ownership/availability of all inputs
4. validate target item if any
5. build normalized ingredient multiset
6. find eligible recipe/process candidates
7. apply discovery/diagram/branch gates
8. evaluate exact success
9. if not exact, evaluate meaningful near-miss
10. choose highest-priority blocker
11. determine consumption class
12. atomically consume inputs
13. create output/effect or grant hint
14. grant discovery state changes
15. notify Journal/UI
16. commit transaction / action result

No client message may skip directly to step 12+.

---

## 4. Ingredient matching

Recipes define roles, not only raw item IDs.

Example roles:
- meat
- clean_water
- dirty_water
- herb:ashbloom
- cloth
- thread
- fuel
- binder
- salt
- fat
- metal:iron

Specific recipes can require exact keys where identity matters.

Matching rules must distinguish:
- required
- optional
- alternative
- forbidden/incompatible
- prepared/refined state

Extra unrelated ingredients are not silently ignored unless the recipe explicitly permits garnish/additive slots.

---

## 5. Discovery gates

Recipes have one of these acquisition modes:
- born-known
- experimentable
- fragment-unlocked
- diagram-unlocked
- keeper-taught
- branch-unlocked + experimentable

A recipe marked diagram-only cannot be brute-forced by exact ingredient guessing.

When exact ingredients are supplied for a gated recipe the result is a gated failure outcome, not accidental success.

---

## 6. Near-miss evaluation

Near-miss logic uses the authoritative solution records from `DISCOVERY-SOLUTIONS.md`.

A near miss is eligible only if:
- the attempt maps strongly to a real recipe/process family;
- the failure condition is defined;
- the player has enough prerequisite knowledge for the hint tier;
- the same stable hint is not already known at the same or stronger tier.

Random mixtures return a generic failure and no persistent knowledge.

One attempt normally grants at most one new persistent hint.

---

## 7. Failure precedence

When multiple blockers exist, select one deterministic blocker.

Default precedence:
1. hard knowledge/diagram gate
2. invalid station/process family
3. dangerous/incompatible input
4. core ingredient/property mismatch
5. preparation/refinement mismatch
6. missing functional role
7. quantity/balance issue
8. heat/time/process execution issue
9. quality-only shortfall

Individual solution records may override precedence where gameplay reasoning demands it.

---

## 8. Consumption classes

Every failure definition specifies a consumption class.

Recommended classes:
- `none`: validation failure before work begins
- `trace`: no item loss; action/time cost only
- `partial`: consume selected expendable inputs
- `full`: consume normal recipe inputs
- `transform`: inputs become a failed/byproduct item rather than vanish

Examples:
- wrong station: none/trace
- dirty-water herbal near miss: partial or full depending on action
- burned food: transform to ruined food
- open-fire wood when seeking charcoal: transform to Ash

Consumption occurs only after the final outcome is selected.

---

## 9. Success

Exact recipe success requires:
- all hard gates satisfied
- station/process compatible
- required inputs satisfied
- no forbidden conditions
- sufficient quantities
- required tool/target valid

On success:
- consume inputs atomically
- produce result/effect
- roll hidden quality if applicable
- grant recipe/process discovery if first success
- grant linked material-use knowledge if specified
- clear/resolve inferred Journal silhouette where appropriate

---

## 10. Hidden quality

Recipe identity and output quality are separate.

Quality inputs may include:
- material quality band
- freshness
- tool condition
- station quality/future upgrades
- process control modifiers

No crafting skill level exists.

Initial weighted output model remains approximately:
- standard 70%
- good 20%
- best 10%

Input quality constrains reachable bands. Poor inputs cannot roll top quality.

A low-quality successful craft is still a recipe success and permanently learns the recipe.

---

## 11. Craft channels

Crafting actions have explicit channel/duration behaviour.

Each definition specifies:
- duration
- movement allowed?
- combat allowed?
- damage interruption?
- station lock?
- inputs reserved at start or consumed at completion?

Default:
- validate and reserve at start
- consume/transform at completion
- release reservation on legitimate interruption

Long processes such as tanning, smoking, fermentation and charcoal may later become queued station jobs. v1 may model them as immediate/short actions if needed, but their data contract must not preclude asynchronous station jobs later.

---

## 12. Item reservation and duplication safety

Once an attempt begins, inputs are reserved by server-side item GUID/count references.

Before completion:
- verify reservation still valid
- verify player/station context still valid

Completion is idempotent by attempt id. Repeated completion callbacks must return the original result rather than consume/produce twice.

---

## 13. Repairs and upgrades

Repair/upgrade recipes target an existing item instance.

Validation includes:
- item is owned by player
- item instance matches required family/tier
- item not already reserved
- diagram/discovery gate satisfied
- materials sufficient

Repair modifies condition; upgrade transforms/replaces the item while preserving allowed instance metadata.

Gear never disappears because a repair attempt failed unless a future explicit risky process says so.

---

## 14. Survival consumables

Food, drink and medicine crafting creates normal items. Their use effects are defined separately from crafting.

Crafting success must not itself reduce Hunger/Thirst/Infection unless the recipe is explicitly an immediate-use field action.

This prevents station code from owning survival state semantics.

---

## 15. Logging and diagnostics

Log at debug level:
- attempt id
- player
- station
- candidate family
- selected outcome key
- hint key if any
- consumption class
- success result key
- quality band

Do not log hidden solution data to player-visible channels.

GM diagnostic commands should eventually support:
- evaluate attempt without consuming
- grant/revoke discovery in test environments
- inspect candidate ranking
- inspect station eligibility

---

## 16. Acceptance criteria

Crafting runtime is correct when:
- exact valid recipes succeed deterministically before quality roll
- gated recipes cannot be guessed around
- random spam yields no useful hint
- one failure yields one deterministic blocker
- duplicate callback cannot duplicate outputs
- consumption matches failure class
- discovery grants are idempotent
- quality never changes recipe identity
- field crafting obeys its restricted recipe set
- client cannot claim success/output/quality
