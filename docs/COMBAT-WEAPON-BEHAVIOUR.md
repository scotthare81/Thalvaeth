# Combat weapon behaviour

This document defines the intended *feel and decision grammar* of Thal'vaeth melee weapons without locking premature numeric balance.

## 1. Universal combat grammar

Every weapon should answer five questions differently:

1. **How close must I get?** — reach and arc.
2. **How long am I committed?** — wind-up and recovery.
3. **What does a miss cost me?** — Vigor, time, noise, exposure.
4. **What opening does a hit create?** — direct damage, bleed, stagger, space.
5. **What does this fight tell the district?** — noise/creature reaction/Director pressure.

Weapons are not tiered solely by DPS. An upgraded weapon becomes more capable/reliable while retaining a distinct risk profile.

## 2. Common action families

V1 should prove a small action vocabulary rather than dozens of buttons:

- **Quick attack** — low commitment, narrow target profile;
- **Committed attack** — more cost/recovery, stronger secondary effect;
- **Special geometry attack** — flurry/cleave/overhead depending on weapon family;
- future defensive/utility action only after core offensive rhythm works.

This is not intended to become an MMO hotbar rotation. A small set of readable actions should carry most combat depth through spacing, timing and resource pressure.

## 3. Crude Dagger / Knife

Role: survival weapon, close-range precision tool, early reliable fallback.

Behaviour:

- shortest reach;
- low per-strike Vigor;
- short wind-up/recovery;
- narrow frontal attacks;
- modest direct damage;
- modest stagger;
- good bleed eligibility against vulnerable anatomy;
- low combat noise;
- explicit butcher capability.

Failure mode: requires dangerous proximity and struggles to create space against multiple/heavy targets.

Broken dagger:

- edge-dependent bleed sharply reduced/disabled;
- direct cut performance poor;
- butcher capability may be disabled until structural repair;
- still usable as desperate close-range weapon if definition allows.

## 4. Iron/Steel Short Blade

Role: generalist single-blade bridge before specialization.

Behaviour:

- slightly more reach/control than dagger;
- moderate direct damage;
- moderate Vigor;
- better committed cut/thrust;
- some stagger but not heavy-control level;
- bleed remains useful;
- moderate noise.

It should feel like a meaningful upgrade without invalidating why a later player might choose paired speed or heavy impact.

## 5. Paired Daggers

Role: close pressure, opening exploitation, bleed-focused fast style.

Logical equipment rule: one persistent matched set.

Suggested action grammar:

### Quick alternating cut
One action, one strike event, presentation alternates hand. Low cost/recovery.

### Committed double sequence
One action with two authored strike events. Vigor paid once at commit for the full sequence. Each strike independently validates target contact; missing the first does not magically redirect/guarantee the second.

### Short flurry
Higher commitment fast-style special. Multiple bounded strike events, target cap/bleed rules authored, meaningful recovery afterward. It must not become a zero-risk blender.

Strengths:

- pressure during creature recovery;
- rapid bleed build within caps;
- quick disengagement after light attacks;
- relatively low noise.

Weaknesses:

- close range;
- low stagger;
- repeated attacks can drain Vigor surprisingly quickly;
- poor against bleed-resistant/heavily protected anatomy if player refuses to adapt.

## 6. Paired Shortswords / Twin Blades

Role: fast-style mature generalist with more reach/direct damage than paired daggers.

Compared with paired daggers:

- slightly longer reach;
- higher direct cut performance;
- somewhat higher Vigor and recovery;
- bleed still useful but less singular identity;
- less/no butcher utility depending on exact form;
- still lower stagger/cleave authority than heavy line.

Twin Blades should not simply be "paired daggers but every number higher." They shift the fast build toward reach and direct combat competence.

## 7. War Cleaver

Role: transitional heavy weapon combining brutal close impact with utility.

Behaviour:

- shorter than greatsword but heavier impact than fast line;
- high chop/cut pressure;
- meaningful stagger;
- selected heavy-butcher capability;
- moderate-to-high noise;
- committed recovery.

It can teach the heavy player to time attacks before the full long-reach greatweapon forms.

## 8. Greatsword

Role: reach, lane control, broad committed cuts.

Suggested grammar:

### Measured cut
Longer reach, moderate-heavy cost, controlled frontal strike.

### Committed thrust/overhead
Narrower geometry, high single-target impact/stagger, long commitment.

### Sweeping cleave
Wide frontal arc, bounded target cap, reduced secondary-target effectiveness if needed, high Vigor/noise/recovery.

Strengths:

- keeps dangerous targets farther away;
- strong direct damage;
- strong stagger pressure;
- controlled multi-target space creation.

Weaknesses:

- expensive miss;
- long recovery;
- loud;
- poor responsiveness after commitment;
- no default butcher/chop capability.

The greatsword should make the player feel powerful **and exposed**.

## 9. Greataxe

Role: concentrated impact/chop, brutal stagger, narrower commitment.

Suggested grammar:

### Heavy chop
Narrower than greatsword sweep, very strong impact/stagger.

### Overhead commitment
High single-target consequence with severe whiff exposure.

### Short brutal sweep
Some multi-target potential but less graceful/wide than greatsword.

Strengths:

- highest/near-highest stagger pressure;
- strong against targets where chop/impact matters;
- explicit chopping utility;
- decisive hit reactions.

Weaknesses:

- very high Vigor;
- loud;
- long recovery;
- misses are dangerous;
- less flexible geometry than greatsword.

## 10. Fast vs heavy matchup philosophy

Against one ordinary vulnerable creature:

- fast can win by entering safely, applying pressure/bleed and avoiding retaliation;
- heavy can win by choosing one or two decisive openings and staggering/ending the threat.

Against multiple creatures:

- fast should rely more on movement, target selection and disengagement;
- heavy may create space with cleave but pays loudly and heavily for doing so.

Against armoured/bleed-resistant creature:

- fast retains direct attacks/mobility but loses some bleed efficiency;
- heavy may gain relative value through impact/stagger.

Against highly mobile creature:

- fast can react/recover more safely;
- heavy must predict/commit and is punished for whiffs.

No matchup should be reduced to a hard "wrong weapon = cannot play" gate in ordinary content.

## 11. Creature reaction readability

Weapon hits should produce reactions proportional to semantic impact, not only floating damage:

- dagger: small flinch, wound/bleed cues where anatomy allows;
- paired pressure: repeated recoil/defensive behaviour without stun-lock;
- greatsword: heavier recoil/stagger when threshold breaks;
- greataxe: violent impact reaction when appropriate;
- Broken weapon: poor/awkward response signalling loss of effectiveness.

Animation availability may limit presentation, but server semantics remain richer than the animation set.

## 12. Whiffs

Whiffs are part of the game.

A miss communicates:

- Vigor was spent;
- recovery still occurs;
- heavy attacks may emit swing/environment noise;
- nearby AI may exploit the opening;
- no damage/stagger/bleed is invented.

Do not add generous auto-homing merely to hide missed attacks.

## 13. Environmental contact

If reliable engine collision/contact can identify weapon impact with world geometry, heavy misses into walls/objects may create extra noise/wear. If this cannot be implemented reliably, omit the mechanic rather than using inconsistent client guesses.

## 14. No free animation cancelling

Switching target, jumping, toggling auto-attack, opening UI, weapon swapping or sending another attack request cannot shorten server recovery.

If later playtesting wants intentional cancel windows, they must be explicit attack-definition features with their own cost/trade-off.

## 15. Progression and movesets

Weapon upgrades may unlock/replace attack definitions, but Gear progression should not explode into a spellbook.

A good progression may improve:

- reach profile;
- recovery efficiency;
- Vigor efficiency;
- damage/stagger/bleed capability;
- access to one signature action;
- reliability/durability.

It should not add five new buttons every tier.

## 16. Broken behaviour by family

Fast blades:
- lose edge/bleed efficiency sharply;
- retain some desperate quick-strike capability;
- repeated use remains costly and ineffective.

Heavy blades/axes:
- lose major damage/stagger/cleave efficiency;
- remain heavy, therefore Vigor cost does not conveniently collapse;
- advanced heavy actions may lock out if structural integrity is required.

This prevents Broken gear from becoming a weird lightweight exploit.

## 17. First playtest questions

Do not tune around DPS meters. Ask:

- Does a dagger make close range feel dangerous but controllable?
- Does paired combat tempt overcommitment through cadence?
- Does a greatsword miss feel costly enough to matter without feeling unusable?
- Does a greataxe hit feel different from a greatsword hit?
- Can heavy stagger create an opening without stun-locking?
- Does bleed matter without becoming passive free damage?
- Does cleave save space without becoming the universal answer?
- Does combat noise meaningfully alter the decision to keep fighting?
- At low Vigor, does the player consider disengagement?
- When a weapon becomes Damaged/Broken, does extraction/repair become an organic choice?

Those answers should drive numeric balance.