# Combat implementation specification

This document defines Thal'vaeth's server-authoritative melee combat contract. It sits beside the Gear/Equipment contracts and deliberately avoids turning combat into stock WotLK rotation play.

## 1. Combat pillars

Combat is dangerous, physical and expensive. The player should prefer a clean kill, an avoided fight or a controlled disengagement over grinding through every creature in sight.

Core rules:

1. Every committed attack has a cost even when it misses.
2. Position, facing, reach and recovery matter.
3. Fast and heavy weapons solve different problems; neither is a universal DPS upgrade.
4. Vigor is the immediate action economy and also a survival resource.
5. Noise makes combat strategically expensive beyond the current target.
6. Stagger creates openings but cannot become permanent crowd-control locking.
7. Bleed rewards sustained precision against appropriate anatomy but is not universal damage.
8. Cleave is deliberate weapon geometry, not free circular AoE.
9. Creature reactions are authored and anatomy/behaviour-specific.
10. Broken weapons remain the player's weapon, but perform badly enough that continuing a run is a real decision.
11. The server owns hit validity, costs, damage, status application, interruption and combat state. Animation/addon presentation never becomes authority.

## 2. Attack model

An attack is an authored action definition referenced by stable key, for example:

- `attack.dagger.quick`
- `attack.dagger.committed`
- `attack.paired.flurry`
- `attack.shortsword.cut`
- `attack.greatsword.cleave`
- `attack.greataxe.overhead`

Each definition may declare:

- weapon capability/family requirements;
- wind-up duration;
- commit point;
- active/hit window;
- recovery duration;
- Vigor cost;
- minimum Vigor requirement;
- reach profile;
- facing arc;
- target cap;
- damage profile;
- stagger pressure;
- bleed profile;
- cleave geometry;
- noise event/profile;
- durability wear class;
- movement restrictions/modifiers;
- interruptibility windows;
- allowed follow-ups;
- creature-reaction tags.

Exact numbers are data/balance, not hard-coded branches.

## 3. Attack state machine

Canonical attack lifecycle:

`READY → WINDUP → COMMITTED → ACTIVE → RECOVERY → READY`

Optional terminal/side states:

- `INTERRUPTED` before commit;
- `CANCELLED` only where an authored action permits cancellation;
- `INVALIDATED` for authoritative failures before commitment.

### READY

Player may request a legal attack.

### WINDUP

Animation/presentation begins. Depending on the attack, movement/turning may be restricted. Before the commit point an authored interruption may cancel without applying the full committed effects.

### COMMITTED

The swing has passed the point of no return. Vigor is charged here. The attack will proceed to its active window unless a post-commit hard interruption rule terminates it. Missing later does not refund Vigor.

### ACTIVE

The server evaluates valid target contact/hit events according to the attack's reach/arc/target rules. One target cannot be hit repeatedly by one active window unless the attack definition explicitly has multiple authored strike events.

### RECOVERY

The player has finished the damaging part of the action but has not returned to neutral. Recovery is a primary balance lever, especially for heavy weapons. Starting another attack, sprinting/Hustle, swapping weapons or performing survival interactions may be restricted.

## 4. Vigor economy

Vigor is paid when the attack commits, not when it hits.

This means:

- whiffed swings cost Vigor;
- hitting scenery does not refund Vigor;
- a creature stepping out of range after commitment does not refund Vigor;
- disconnect/reconnect cannot refund an already committed attack;
- duplicate attack messages cannot double-charge Vigor.

An attack definition has a minimum Vigor requirement. If the player cannot meet it at request/commit validation, the server rejects the attack before commitment.

Combat must not permit Vigor to become negative.

Low Vigor remains dangerous because it reduces the player's ability to attack, Hustle/disengage and recover under Survival rules. Combat should therefore create the tension: **spend Vigor to end the threat, or preserve enough to escape the next one.**

## 5. Vigor cost identity

Fast weapons:

- low cost per individual strike;
- can become expensive through repeated cadence;
- encourage restraint rather than button spam;
- should leave room for movement/disengagement.

Heavy weapons:

- high cost per committed attack;
- stronger consequence for a miss;
- larger recovery exposure;
- should reward choosing the moment rather than sustained spam.

Paired weapons are one logical Gear instance. A multi-hit paired attack may contain multiple strike events but has one authored action cost and one logical durability source.

## 6. Reach and spatial validation

Native WotLK melee range alone is insufficient as final design authority.

Each attack uses an authored reach profile. Server validation considers:

- attacker position;
- target position/bounds;
- facing/attack arc;
- maximum reach;
- optional minimum reach/dead-zone for attacks that need one;
- line-of-sight/world obstruction where technically reliable;
- target alive/attackable state;
- attack active-window timing.

Do not allow the client to say "I hit target X." The client says "I attempted attack A"; the server resolves whether X was legally struck.

Latency tolerance may use a small documented server-side allowance, but it must not expand into exploitable long-range melee.

## 7. Facing and arcs

Attacks define a facing arc rather than all using a full circle.

Fast thrust/cut attacks normally use a narrow frontal arc. Heavy horizontal cleaves may use a wider frontal arc. Overhead attacks may be narrow but longer/high-impact.

Turning during WINDUP/ACTIVE may be limited by attack definition. Heavy attacks should not allow instant 180-degree tracking after commitment.

The target's own facing can matter later for creature defence/weak-point rules, but v1 does not require a player backstab system.

## 8. Hit resolution

For each authored strike event:

1. validate attacker still owns/has legal active weapon state;
2. validate attack state and strike event not already resolved;
3. build candidate targets from server world state;
4. apply reach/arc/LOS/target filters;
5. order candidates deterministically for target-capped attacks;
6. resolve defence/avoidance if that creature supports it;
7. resolve damage;
8. resolve stagger pressure;
9. resolve bleed eligibility/application;
10. emit creature reaction stimulus;
11. emit noise as defined by the action/impact;
12. apply one logical durability wear event where appropriate;
13. persist/audit important state.

Damage and secondary effects must derive from the same authoritative strike resolution so a target cannot take bleed/stagger from a rejected hit.

## 9. Damage model

The combat layer consumes the effective weapon profile from Gear, then applies the attack definition and target defence profile.

Conceptually:

`effective strike = weapon profile × attack profile × condition modifiers × target response`

Avoid using raw WotLK weapon DPS/item level as the final authority.

Damage types may include authored semantic tags such as:

- cut;
- pierce;
- chop;
- crush/impact;

These exist to support creature armour/anatomy/reaction rules. Do not build a huge elemental-resistance spreadsheet for v1.

## 10. Fast weapon identity

Fast weapons are not simply "lower damage, faster swing." Their combat identity is:

- short-to-medium reach;
- low commitment per strike;
- short recovery;
- lower immediate stagger;
- controlled bleed pressure on vulnerable creatures;
- ability to exploit openings and disengage;
- quieter impacts/actions than heavy weapons;
- lower individual Vigor cost;
- poor efficiency when blindly striking heavily protected targets.

Paired weapons may use alternating strike presentation and authored flurry actions. They do **not** receive two independent auto-attacks simply because two WotLK carriers are visible.

## 11. Heavy weapon identity

Heavy weapons are about commitment and space control:

- longer reach where form supports it;
- high impact;
- high stagger pressure;
- selected cleave attacks;
- high Vigor cost;
- longer wind-up/recovery;
- larger noise event;
- greater punishment for whiffing;
- reduced ability to redirect after commitment.

Greatsword and greataxe should still feel different through authored attack profiles. A greataxe may concentrate chop/impact and stagger; a greatsword may offer reach and broader controlled cuts. Exact moveset remains content data.

## 12. Cadence

Thal'vaeth combat cadence is driven by authored actions and recovery, not stock WotLK auto-attack swing timers as the sole mechanic.

Native auto-attack may be suppressed, repurposed or used as an implementation carrier, but the server must enforce the custom action cadence.

Cadence includes:

- wind-up;
- active window;
- recovery;
- permitted follow-up window;
- movement restrictions;
- Vigor availability;
- interruption state.

No client-side animation cancel may shorten authoritative recovery.

## 13. Stagger system

Stagger represents destabilization/poise pressure, not a conventional long stun rotation.

Creatures have an authored stagger profile:

- stagger threshold/poise capacity;
- pressure decay/reset behaviour;
- immunities/resistances;
- reaction when threshold breaks;
- post-stagger protection window;
- special vulnerability states if any.

Successful attacks add stagger pressure according to attack/weapon/target response. When threshold is crossed, the creature enters an authored stagger reaction.

### Anti-stunlock rule

After a meaningful stagger, the creature receives a short authored **stagger protection/recovery state** or equivalent threshold reset so repeated rapid attacks cannot indefinitely lock it.

Boss/special creatures may:

- require much more pressure;
- only stagger during specific states;
- convert threshold break into a flinch rather than full interruption;
- ignore selected stagger sources.

## 14. Stagger decay

Pressure should normally decay when the player stops applying meaningful pressure. This rewards committed pressure without turning stagger into a permanent hidden meter carried forever through an encounter.

Exact decay timings are balance data.

A creature leaving combat/resetting should clear transient stagger pressure unless an authored encounter says otherwise.

## 15. Bleed system

Bleed is an authored status effect representing ongoing physical injury. It is primarily a fast/cutting weapon pressure mechanic.

A bleed definition declares:

- eligibility/anatomy tags;
- application strength;
- duration/tick model;
- stack/refresh policy;
- maximum intensity/stacks;
- damage contribution;
- reaction stimulus;
- whether movement/action can worsen it, if ever authored.

V1 should keep bleed bounded and readable. Recommended semantic model: a small number of intensity tiers rather than unlimited stacking.

## 16. Bleed anatomy

Not every creature bleeds meaningfully.

Creature templates can declare:

- normal bleed susceptibility;
- reduced susceptibility;
- immune/no meaningful bleed;
- special reaction.

Examples conceptually include desiccated/construct-like/unnatural enemies resisting or ignoring bleed. The exact Rotwood creature matrix belongs in creature combat data.

Bleed immunity must not make fast builds useless; those enemies should remain solvable through direct damage, positioning or other fast-style strengths.

## 17. Bleed refresh/stack safety

Repeated strikes cannot create unbounded DOT multiplication. The server owns one status record per defined bleed family/source policy and applies deterministic cap/refresh rules.

Relog/disconnect during an active run cannot clear a bleed from a creature if that creature/run state is still authoritative, nor duplicate it on reconnect.

## 18. Cleave

Cleave is an authored attack property, not a weapon-wide always-on passive.

A cleave definition includes:

- frontal arc;
- reach;
- target cap;
- target ordering;
- per-secondary-target damage multiplier if used;
- stagger multiplier if used;
- bleed eligibility;
- obstruction rules.

Each target is independently validated. Hitting the primary target does not grant damage to invalid targets through walls/out of arc.

A heavy cleave should create space but also create risk through commitment, Vigor and noise.

## 19. Cleave target ordering

Target-capped cleaves need deterministic ordering to prevent client manipulation. Suggested order:

1. valid targets intersecting the authored strike geometry;
2. nearest contact/distance from attacker or strike origin;
3. stable GUID tie-break.

The client never selects the entire cleave target list.

## 20. Noise

Combat emits semantic noise events into the same world-pressure ecosystem used by creature AI/Director systems.

Noise sources may include:

- attack commitment/swing;
- weapon impact on creature;
- weapon impact on environment;
- creature pain/death vocalization;
- heavy armour movement during combat where later supported;
- special loud weapon actions.

Noise definitions should include an intensity/radius/category rather than directly commanding every nearby creature to aggro.

The AI/Director consumes the stimulus according to creature hearing/behaviour and run pressure rules.

## 21. Noise identity by weapon

Fast blade combat should generally be less noisy than heavy combat, but never silent by default.

Heavy weapon actions create more substantial noise, especially impacts/cleaves. This is a strategic cost: the heavy weapon may solve the current enemy quickly while making the surrounding district less safe.

Exact radii/values are balance data.

## 22. Creature reaction pipeline

Combat produces semantic stimuli rather than hard-coding every creature response in weapon code.

Potential stimuli:

- `combat.hit.light`
- `combat.hit.heavy`
- `combat.stagger.break`
- `combat.bleed.applied`
- `combat.noise.*`
- `combat.ally_death`
- `combat.player_whiff_heavy` where an AI archetype intentionally exploits openings.

Creature AI decides whether to:

- flinch;
- stagger;
- recoil;
- enrage;
- flee;
- investigate sound;
- call allies;
- exploit player recovery;
- ignore the stimulus.

This keeps weapon mechanics generic while preserving creature personality.

## 23. Creature attacks against player

The same broad commitment model should inform creature attacks even if implementation is simpler initially:

- telegraph/wind-up;
- authored reach/arc;
- active hit event;
- recovery;
- interruption/stagger rules;
- noise/reaction.

Creature attacks must not rely solely on stock instant melee damage if the visual telegraph implies dodgeable spacing/timing.

V1 can support a limited authored creature-attack set before every creature receives bespoke moves.

## 24. Interruption

Interruption is phase-aware.

### Before commit

A qualifying hard hit/stagger may cancel an attack before Vigor commitment depending on attack definition. No full attack cost is charged if commitment never occurred, though an authored small setup cost could exist later.

### After commit

Vigor remains spent. A sufficiently strong authored interruption may terminate the active strike or force recovery, but cannot refund the committed cost.

### Recovery

Being hit during recovery does not magically reset the player to READY. Creature AI may deliberately exploit long heavy recovery windows.

## 25. Player hit reactions

Not every incoming hit should hard-stun the player. Incoming attacks may apply:

- no interruption;
- light flinch/presentation only;
- action interruption before commit;
- hard interruption/stagger;
- movement slow/knock effect where specifically authored.

Armour/gear may influence these outcomes through derived protection/stability profiles later, but v1 should avoid opaque random stun chains.

## 26. Movement during attacks

Each attack defines movement policy:

- free/near-free movement for very light actions;
- reduced movement during committed cuts;
- root/strong restriction for major overhead/heavy actions;
- optional small authored step/lunge.

Movement is server-bounded. Client movement cannot extend an attack's reach beyond allowed geometry.

Hustle/sprint cannot be used to cancel authoritative recovery unless an attack explicitly permits that escape.

## 27. Broken weapon combat behaviour

Broken is severe but recoverable.

At Broken, a weapon may receive an authored degraded profile including:

- strongly reduced damage/edge performance;
- strongly reduced stagger/cleave effectiveness;
- bleed disabled or heavily reduced for broken blades;
- tool capability disabled where structural integrity is required;
- equal or **higher** Vigor burden — never a free efficiency bonus because the weapon is damaged;
- ugly/noisy impact presentation where appropriate;
- no access to selected advanced attacks.

Broken does not:

- delete the weapon;
- auto-unequip it;
- create a replacement;
- reset quality;
- repair on death/extraction.

## 28. Worn/Damaged combat scaling

Condition penalties should be stepped/legible enough to matter without turning every 1% durability loss into noisy stat churn.

Recommended approach: effective combat modifiers derive primarily from condition band, with exact durability used for persistence/threshold crossing.

Fine: intended profile.

Worn: small efficiency/edge degradation.

Damaged: clearly meaningful degradation; player should consider extraction/repair.

Broken: severe emergency profile.

Exact modifiers remain balance data.

## 29. Durability wear from combat

A committed attack does not necessarily cause full wear merely because the button was pressed. Wear definitions distinguish semantic events such as:

- normal successful strike;
- heavy impact;
- environment impact/miss where blade hits world if reliably detectable;
- blocked/parried impact later;
- special stress action.

One authored strike event applies wear once to the logical Gear instance. Paired carriers never double wear.

## 30. Death and combat

On lethal player damage:

- no new player attacks may commit;
- pending uncommitted attacks cancel;
- committed attack effects after death only resolve if explicitly permitted by deterministic strike timing; v1 recommendation is to cancel future unresolved player strike events once DEAD transition wins;
- death/run finalization authority takes precedence over presentation;
- combat statuses/run-local effects are cleared with run teardown;
- persistent gear retains exact durability/quality/node.

The death presentation/bag-drop/instance-reset contract remains owned by Run Lifecycle docs.

## 31. Creature death

Creature death is authoritative and idempotent. A dead creature cannot be hit again for additional bleed/stagger/loot/Journal engagement credit.

Death may emit sound/ally reaction stimuli and feed Director/encounter state.

Loot/harvest availability is a separate transaction from damage resolution.

## 32. Combat state and disengagement

Combat should not be a permanent binary flag merely copied from WotLK. Systems may need semantic state such as:

- currently threatened/engaged;
- recent hostile action;
- active creature pursuit;
- recovery/channel restrictions;
- Director heat contribution.

Extraction gates and survival interactions consume server-owned combat/threat state according to their contracts.

## 33. Anti-cheat/authority invariants

The client cannot authoritatively provide:

- hit target list;
- damage amount;
- stagger amount;
- bleed stack;
- attack completion time;
- recovery completion;
- Vigor refund;
- weapon condition;
- noise radius;
- creature reaction result.

Requests are intents. Server time/world state decides results.

## 34. Idempotency

Every attack/strike resolution requires a server-owned action ID or equivalent identity sufficient to prevent:

- duplicate Vigor charge;
- duplicate hit;
- duplicate bleed;
- duplicate stagger;
- duplicate durability wear;
- duplicate noise emission where replay would matter.

Reconnect/retry cannot replay a successful strike.

## 35. Diagnostics

Required debug tooling should expose server truth to authorized GMs:

- `.thal combat state`
- `.thal combat attack <key>` test/dry-run where safe
- `.thal combat vigor`
- `.thal combat reach`
- `.thal combat stagger <target>`
- `.thal combat bleed <target>`
- `.thal combat noise`
- `.thal combat trace on|off`

Trace should show action ID, phases/timestamps, Vigor commit, candidate targets, rejection reasons, final hits, secondary effects, wear and noise.

## 36. Acceptance

Combat foundation is implementation-ready when deterministic tests prove:

1. attack phases and commit point cannot be bypassed;
2. misses spend committed Vigor;
3. insufficient Vigor rejects safely;
4. reach/facing/target caps are server-authoritative;
5. fast/heavy cadence remains mechanically distinct;
6. stagger cannot create indefinite locks;
7. bleed is bounded and anatomy-aware;
8. cleave validates every target;
9. noise feeds semantic AI/Director stimuli;
10. creature reactions are data/AI-owned rather than weapon-script hardcoding;
11. interruption before/after commit follows cost rules;
12. Broken weapons persist and use degraded profiles;
13. paired weapon carriers never double damage/wear/cost;
14. death/reconnect cannot replay or refund committed combat actions;
15. native/addon messages cannot forge hit results.