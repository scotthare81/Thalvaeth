# Rotwood creature behaviour matrix — 90001–90010

This matrix translates the locked roster in `CREATURES.md` into implementation-facing AI/combat responsibilities **without inventing replacement identities**. Existing locked rules win. Numeric values already present in `CREATURES.md` remain POC/tuning unless explicitly locked there.

## 1. Matrix summary

| ID | Archetype | Primary encounter job | Normal acquisition | Combat identity | Disengage/reset identity |
|---|---|---|---|---|---|
| 90001 | sleeper | punish careless movement/combat near setpiece | special wake trigger | explosive wake/opening pressure | setpiece-specific |
| 90002 | scavenger | cheap fodder/pair pressure | ordinary perception/noise | basic melee + Rend | local wander/disengage |
| 90003 | rafter | overhead ambush | 6y + LOS locked leap trigger | leap opener then grounded combat | one-shot leap remains spent |
| 90004 | caller | escalation/wave trigger | ordinary engage | scream timing is the fight | death also escalates |
| 90005 | brute | corridor/gate pressure | defensive/post encounter | slow, huge HP, stomp/space denial | 12y post leash, **keeps HP** |
| 90006 | snare | choke displacement | ordinary engage | timed >5y LOS hook | ordinary bounded disengage unless content says otherwise |
| 90007 | stalker | persistent unease/pace pressure | special pacing relation | not normal offensive caster v1; flees on first solid hit | re-stalk after break-off, **keeps HP** |
| 90008 | drifter | avoidable ambient threat | slow/low-reactivity perception | melee only | resumes wide wander |
| 90009 | stumbler | doorway soft wall | ordinary perception | slow/tanky + wide swing | bounded local disengage, keeps run damage |
| 90010 | ghoul | fast noise/peak punish | ordinary perception | rush + Rend + low-HP frenzy | bounded local disengage, keeps run damage |

## 2. 90001 Ash Sleeper — `sleeper`

### Locked rule

Passive/kneeling. Wakes when a player within 10y is in combat or running. Opens with Claw Rampage. Walking/Still Breath is the counter. Setpiece; never random trash.

### AI contract

The Sleeper does **not** use ordinary roaming acquisition while dormant. Its dormant substate is a special sensor:

`DORMANT → WAKING → ENGAGED`

While DORMANT:

- remains passive/kneeling;
- evaluates the locked proximity + `IsInCombat() || IsRunning()` condition;
- does not chase ordinary distant noise;
- does not inherit generic social aggro that would invalidate the "don't wake it" rule unless later explicitly designed;
- may receive presentation/orientation cues only if they do not expose hidden behaviour incorrectly.

On valid trigger:

1. latch wake once;
2. stand/aggressive;
3. create authoritative Combat wake action/Rampage;
4. publish noise/reaction as appropriate;
5. Journal can observe *Wakes swinging* only from valid witnessed behaviour.

### Combat relationship

Its opening should feel sudden and dangerous, but the custom Combat layer must eventually give the opening honest active geometry rather than invisible stock damage.

### Procedural placement

Although population is procedural generally, Sleeper remains **setpiece/designer-node constrained**. Run generation may select among eligible Sleeper setpiece nodes if later desired; it must not place one in arbitrary open terrain and destroy the readable counterplay.

## 3. 90002 Cellar Scavenger — `scavenger`

### Locked rule

Low-threat fodder, often pairs, random OOC wander, noise-curious, Rend in combat.

### AI identity

Scavenger is the baseline proof of ordinary perception:

- modest visual acquisition;
- meaningfully reacts to nearby noise;
- INVESTIGATING is important — it should be possible to draw one toward a sound without instantly giving it player lock;
- paired Scavengers may alert/assist one another through local witnessed/heard combat, not psychic aggro sharing.

### Combat

Simple attack set:

- basic melee action;
- Rend as an authored committed attack/status action once custom Combat replaces stock SmartAI casting.

Scavenger should be easy alone but dangerous when the player lets Caller/noise stack bodies into the same space.

### Search/disengage

After LOS loss, inspect last evidence briefly, then return to local wander. Do not chase through half the district using live player coordinates.

## 4. 90003 Catwalk Prowler — `rafter`

### Locked rule

Passive above. Player within 6y and LOS triggers one-shot leap. Attacks shortly after landing. Never re-leaps.

### AI contract

Substates:

`PERCHED → LEAP_COMMITTED → GROUNDED_ENGAGED → GROUNDED_DISENGAGED`

- PERCHED suppresses generic chase/ordinary ground combat.
- Trigger requires locked range + LOS.
- Leap is latched one-shot in run state.
- After leap, creature permanently uses grounded behaviour for that run.
- Disengagement never teleports it back to the catwalk and never resets the leap flag.

### Combat

The leap becomes an authored movement/attack sequence with a readable landing threat. The player can bait it from a safer position, after which it is a conventional grounded threat.

### Procedural placement

Only spawn at nodes explicitly tagged as legal overhead/perch geometry. Procedural selection cannot assign a Prowler to a ground-only node.

## 5. 90004 Ash Caller — `caller`

### Locked rule

About 2s after engage, shrieks once and spawns +2 Scavengers; death spawns +3. Caller is the peak budget spend.

### AI contract

Substates:

`ENGAGED_UNCALLED → CALL_WINDUP → CALLED → NORMAL_ENGAGED`

The call is **one-shot persistent run state**.

The custom Combat transition should turn the approximate 2s behaviour into an honest interruptible special action:

- telegraph/wind-up;
- commit;
- successful call event;
- wave request to encounter/run system.

The existing counter "burst it before the shriek" must remain valid. Whether direct damage alone interrupts is not assumed; Combat stagger/interrupt rules decide. Killing it before call obviously prevents that engage-call, but death still invokes the separately locked +3 death wave.

### Wave ownership

Caller AI requests an authored wave event. It should not contain generic creature-spawn logic duplicated throughout AI. The run/encounter service validates eligible spawn locations and run state.

### Death

Death wave emits once, idempotently. Reconnect/restart cannot replay it.

## 6. 90005 Patchwork Brute — `brute`

### Locked rule

Slow, loud, ×12 HP POC, defensive, opens with Stomp, 12y leash from post, returns/teleports home if dragged out, **keeps wounds**.

### AI identity

Brute is territory denial, not a marathon pursuer.

Substates:

`GUARDING → ENGAGED_IN_TERRITORY → RETURNING → GUARDING`

- home anchor comes from generated/authored spawn node;
- 12y leash is measured from that run's anchor, not hard-coded coordinates;
- leaving territory causes disengage/return according to locked behaviour;
- HP/durability-like combat state is preserved;
- return must not call a generic full-heal evade reset.

### Combat

- slow deliberate attack cadence;
- high stagger resistance/poise is consistent with the locked corridor-block role but exact profile requires reviewed combat data before becoming canon;
- Stomp is the signature opening and should use honest telegraph/geometry;
- its long recovery should create player openings without making it trivial.

### Noise

Brute is explicitly loud. Its movement/attacks can publish stronger stimuli once Noise tuning is implemented.

## 7. 90006 Broken Snare — `snare`

### Locked rule

~3s after engage, then ~8s: if target is in LOS and >5y, hooks target into choke. Counter is LOS break or closing inside 5y.

### AI contract

The Snare's decision-making must preserve the geometry puzzle:

- only choose Hook when target evidence is current enough to validate LOS and distance;
- no hook through walls after LOS loss;
- >5y requirement evaluated authoritatively at commit/active resolution;
- hook timing/cooldown is server-owned;
- close-range player intentionally suppresses hook selection but remains exposed to ordinary combat.

### Combat

Hook should become an authored special action with telegraph, commit and displacement resolution. Breaking LOS during the legal pre-hit window must matter according to Combat timing.

### Placement

Run generation should prefer nodes where a choke/pull has meaningful geometry. Do not place Snare in a featureless field just because the creature pool permits it.

## 8. 90007 Edge Stalker — `stalker`

### Locked rule

Paces ~22–26y from nearest player within 40y. When hit once, breaks off, wanders ~15y, re-stalks after ~8s. Keeps wounds. No offensive cast v1.

### AI identity

This is **not ordinary ENGAGED chase AI**. It is a pressure state machine:

`STALKING → HIT_BREAKOFF → WITHDRAWING → COOLDOWN → RESTALKING`

While stalking:

- choose movement positions that maintain the authored distance band where pathing permits;
- avoid reading this as "always attack the nearest player";
- do not enter standard melee merely because it is within generic aggro range;
- preserve health across all substates.

On first solid hit in a stalking cycle:

1. record damage normally;
2. break off rather than generic evade;
3. choose a navigable withdrawal/reposition target roughly consistent with locked ~15y intent;
4. wait/recover approximately locked ~8s;
5. resume stalking if still alive/run-valid.

### Search/perception

The Stalker can use stronger target-follow awareness than fodder while performing its explicit pacing rule, but it still must not path through walls or teleport to hidden coordinates. Exact stealth interaction belongs to the later Stealth contract.

### Director relationship

Director can use Stalker as pace pressure, but once spawned the Stalker's own state machine owns its behaviour.

## 9. 90008 Drifter — `drifter`

### Locked rule

Slow wide wander, slow to aggro, solo, melee only, intended to be routed around.

### AI identity

Drifter proves that not every visible creature immediately attacks.

- lower/slow acquisition response relative to aggressive fodder;
- weak noise interest compared with Scavenger unless stimulus is strong;
- longer suspicious/investigation hesitation is appropriate to its locked "barely reacts" identity;
- once truly engaged, uses simple melee.

The player should often be able to leave it alone.

## 10. 90009 Stumbler — `stumbler`

### Locked rule

Slow (0.6), tanky, blocks doorways, Wide Swing/Cleave, soft wall.

### AI identity

- ordinary perception;
- slow pursuit;
- geometry/position matters more than chase speed;
- Wide Swing becomes an authored frontal cleave rather than stock free AoE;
- player counter remains stepping out/pulling it from the doorway.

### Combat

Its cleave should have clear wind-up/arc and bounded target geometry. High health/slow movement creates space puzzle rather than tactical intelligence.

### Placement

Procedural nodes should favour doorway/narrow-space eligibility when using Stumbler as intended soft wall, while still permitting appropriate general pressure nodes if authored.

## 11. 90010 Ghoul — `ghoul`

### Locked rule

Fast rush (run 1.4), Rend, Frenzy/Enrage below 40%, peak/noise punish.

### AI identity

- aggressive fast acquisition once genuinely roused;
- closes distance rapidly;
- shorter search persistence may still cover last-known position, but speed makes a confirmed Ghoul dangerous;
- low-HP frenzy triggers once/according to authored state and cannot replay on reconnect.

### Combat

- fast rush/pursuit;
- Rend as committed attack/status;
- Frenzy below threshold changes its authored combat profile/cadence, not merely a hidden stock buff if custom Combat can express it;
- must still obey attack recovery and legal geometry.

### Director

Noise/failed actions can make Ghoul more likely as a Director pressure choice, but an already spawned Ghoul does not know where the player is simply because Director heat is high.

## 12. Cross-archetype social rules

V1 recommended information flow:

- Scavenger/Drifter/Stumbler/Ghoul can react to nearby combat/noise according to hearing profiles;
- specials keep their locked behaviour first;
- Caller shriek is explicit escalation, not generic hearing;
- Sleeper dormant rule is not overridden by ordinary social aggro;
- Stalker pressure behaviour is not converted into ordinary chase by a random ally shout unless explicitly authored later;
- Brute remains territory-bound.

Exact social affinities/group tables should be data, not switch statements.

## 13. First combat-profile review points

Before C++ data is locked, review per archetype:

- vision/hearing sensitivity;
- search duration;
- bleed susceptibility/anatomy;
- stagger threshold/protection;
- attack definitions;
- interruptible special windows;
- noise emitted by attacks/death;
- social response classes.

Only properties already explicit in `CREATURES.md` are canon by default. The rest are tuning/design review points.

## 14. Acceptance

The roster matrix is working when each 90001–90010 creature remains recognizable by its **rule**, not its HP bar:

- Sleeper can be deliberately slipped past;
- Scavenger is cheap but noise/socially dangerous;
- Prowler's one leap can be baited and stays spent;
- Caller creates a readable escalation race;
- Brute owns a corridor and keeps wounds;
- Snare makes distance/LOS/choke geometry matter;
- Stalker pressures pace without becoming generic melee AI;
- Drifter is often worth avoiding rather than killing;
- Stumbler is a slow spatial blocker with honest cleave;
- Ghoul is a fast confirmed-threat/noise punishment.