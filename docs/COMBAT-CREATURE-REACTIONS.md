# Combat creature reactions

This document defines the boundary between generic Combat mechanics and creature personality. Combat publishes semantic events; creature definitions decide what those events mean.

## 1. Principle

A weapon should not contain Rotwood-specific AI logic. A creature should not need to reimplement hit resolution.

Generic Combat determines:

- legal hit;
- damage tags/severity;
- stagger pressure/break;
- bleed status;
- noise stimulus;
- action/recovery state.

Creature combat data/AI determines:

- flinch/recoil animation/behaviour;
- stagger threshold/profile;
- bleed susceptibility;
- aggression/fear/call behaviour;
- whether it exploits player recovery;
- whether it reacts to ally death/noise;
- special anatomy/resistance.

## 2. Reaction profile

Each creature archetype should eventually declare a compact profile:

```text
CreatureCombatProfile
  defense/anatomy tags
  bleedProfile
  staggerProfile
  hearingProfile
  hitReactionProfile
  aggression/reaction tags
  attackSet
```

Do not overload creature entry IDs with hidden code branches when stable data keys can express the rule.

## 3. Hit reactions

A successful hit can be classified semantically, for example:

- light cut/pierce;
- meaningful wound;
- heavy impact;
- stagger break;
- lethal.

Creature reaction does not have to interrupt its action for every hit. Most light hits should produce visual/audio feedback without becoming hard control.

## 4. Stagger personality

Different creatures can respond differently to the same pressure:

- frail humanoid/scavenger: lower threshold, meaningful recoil;
- fast predator: moderate threshold but quick protection/recovery;
- brute: high threshold, dramatic threshold-break opening;
- unnatural/desiccated creature: high resistance or altered reaction;
- caller/support creature: may interrupt a call if threshold breaks during its commit window.

Exact mappings for creature IDs 90001–90010 should be authored when their combat roles are reviewed rather than guessed globally here.

## 5. Bleed personality

Creature anatomy tags should answer whether cut/pierce wounds produce useful ongoing bleed pressure.

Profiles:

- `normal` — standard bounded bleed;
- `resistant` — reduced intensity/duration/effect;
- `immune` — no bleed state;
- `special` — custom authored consequence.

UI/presentation should communicate immunity/resistance through behaviour/impact cues where possible rather than MMO "IMMUNE" spam, though debug tools need explicit reason codes.

## 6. Noise/hearing

Hearing is not binary aggro radius.

A creature receiving a noise stimulus may:

- ignore it;
- orient/listen;
- investigate last-known sound position;
- alert nearby allies;
- enter heightened state;
- directly pursue if source is already identified.

Noise category/intensity, distance, current AI state and creature hearing profile influence the response.

## 7. Ally reactions

Creature death/pain/combat can create nearby stimuli. Not every enemy is a psychic hive mind.

An ally may need:

- audible range;
- line-of-sight;
- same pack/group relation;
- authored caller link;
- current alert state.

This allows one messy fight to spread naturally without every creature in the instance snapping into combat.

## 8. Exploiting player recovery

Selected intelligent/aggressive creatures may recognize broad player vulnerability states such as:

- heavy attack committed/recovering;
- player staggered;
- player low-mobility/blocked;
- player turned away/disengaging.

AI receives semantic opportunity tags; it does not read hidden client animation frames.

V1 should use this sparingly. The goal is believable pressure, not perfect omniscient punishment.

## 9. Creature attack grammar

Creature attacks should use readable phases:

`telegraph/windup → commit → active hit geometry → recovery`

Attack definitions specify:

- reach/arc;
- wind-up;
- active timing;
- damage/infection/stagger effects;
- interruptibility;
- noise;
- recovery.

This lets the player learn creatures through observation and supports the Creature Journal's Observed knowledge state.

## 10. Interruptible creature actions

Some actions should be meaningful interruption targets:

- Ash Caller call/alert;
- heavy brute wind-up;
- trap/snare interaction if applicable;
- special charge/lunge.

A light dagger poke should not automatically cancel every special action. The action defines required interrupt/stagger condition.

## 11. Reaction protection

After hard stagger/major recoil, apply an authored protection/recovery period or raised threshold. This is mandatory for ordinary repeatable control.

Light visual flinches can occur without resetting attack state; do not confuse feedback animation with gameplay stun.

## 12. Death

On authoritative creature death:

- stop pending future attack events;
- clear transient stagger/bleed state;
- emit death noise/reaction once;
- notify Director/encounter state once;
- enable loot/harvest through its own authority;
- update Journal engagement/observed knowledge only through valid hooks;
- never allow corpse hits to farm effects/credit.

## 13. Run reset

All transient creature combat state belongs to the run instance. Death/extraction instance teardown destroys:

- current HP;
- stagger pressure;
- bleed;
- alert/investigation state;
- temporary combat relationships;
- pending attack actions.

A new run's randomized population starts clean.

## 14. Random spawn compatibility

Creature combat behaviour is keyed by archetype/profile, not fixed spawn coordinate. The Runner/Drifter/Caller/etc. must behave correctly wherever the run generator legally places them.

No combat script may assume "this creature is always behind this shed" or depend on a permanent spawn GUID from a fixed run layout.

## 15. Journal observation hooks

The Creature Journal can learn from server-confirmed observations such as:

- player has sighted creature;
- player engaged it;
- creature performed a distinctive attack/ability in valid visibility/context;
- player witnessed a special reaction/resistance.

Combat publishes semantic observations; Journal decides whether they promote knowledge. Addon cannot infer hidden abilities from data files.

## 16. Acceptance

Creature reaction integration is ready when:

- generic weapons contain no coordinate/creature-entry-specific behaviour;
- light hits give feedback without hard-locking AI;
- stagger break interrupts only where valid;
- bleed respects anatomy;
- noise produces investigate/alert behaviour rather than unconditional radius aggro;
- creature attacks use honest telegraph/active/recovery timing;
- death emits once;
- run teardown clears all transient combat state;
- randomized spawn placement does not alter archetype combat semantics.