# Combat deterministic test vectors

These tests lock behaviour, not final balance numbers.

## A. Attack lifecycle

### A1 — normal attack
Request legal quick attack with enough Vigor.
Expected: READY→WINDUP→COMMITTED; Vigor charged once; ACTIVE resolves legal hit once; RECOVERY enforced; returns READY.

### A2 — spam duplicate request
Send duplicate/replayed request/action packet.
Expected: no duplicate action, Vigor cost, hit, wear or noise.

### A3 — new attack during recovery
Request incompatible attack before recovery end.
Expected: reject; recovery timestamp unchanged.

### A4 — client animation cancel
Jump/swap/UI/autoattack toggle/new packet during recovery.
Expected: authoritative recovery remains.

## B. Vigor

### B1 — miss costs Vigor
Commit attack then target leaves valid geometry before ACTIVE.
Expected: Vigor remains spent; no damage/stagger/bleed; recovery occurs.

### B2 — insufficient at request
Vigor below minimum.
Expected: reject before commitment; no negative Vigor.

### B3 — Vigor changes before commit
Attack accepted during windup, another authoritative cost leaves insufficient Vigor before commit.
Expected: commit fails safely; no damaging active window; no double cost.

### B4 — reconnect after commit
Disconnect/reconnect after Vigor commit.
Expected: no refund and no replayed strike.

## C. Reach/facing

### C1 — in range/in arc
Expected: candidate accepted.

### C2 — out of reach
Expected: rejected even if client claims target.

### C3 — behind attacker
Narrow frontal attack, target behind.
Expected: rejected.

### C4 — heavy tracking
Target crosses behind after heavy commit.
Expected: no instant 180-degree retarget beyond authored turning allowance.

### C5 — wall/LOS
Where reliable LOS applies, target through obstruction.
Expected: rejected.

## D. Paired weapons

### D1 — alternating quick strike
Expected: one logical action/strike/cost/wear source despite hand presentation.

### D2 — double sequence
Expected: one action cost; two unique strike IDs; each independently validates contact; target cannot be hit twice by replaying one strike ID.

### D3 — carrier exploit
Forge main/off-hand native attack events separately.
Expected: cannot create independent attacks outside logical Combat action.

## E. Fast identity

### E1 — dagger pressure
Expected: short recovery, low individual Vigor, modest stagger, valid bleed on susceptible target.

### E2 — repeated spam
Repeat fast attacks until Vigor pressure accumulates.
Expected: cadence does not bypass Vigor; player eventually loses safe action economy.

### E3 — bleed-immune target
Expected: direct hit still works; bleed rejected with anatomy reason; no hidden stack.

## F. Heavy identity

### F1 — committed heavy hit
Expected: high authored stagger/noise/recovery relative to fixture fast attack.

### F2 — heavy whiff
Expected: full committed cost/recovery/noise policy; no damage.

### F3 — exploit recovery
Creature attacks during player's heavy recovery.
Expected: recovery does not auto-cancel to protect player.

## G. Stagger

### G1 — pressure below threshold
Expected: pressure accumulates; no hard stagger.

### G2 — threshold break
Expected: one stagger reaction; threshold state resets/reduces; protection begins.

### G3 — attempted stun-lock
Continue rapid pressure during protection.
Expected: no repeated full stagger until protection/rules permit.

### G4 — decay
Stop pressure long enough for authored decay.
Expected: transient pressure decreases/resets deterministically.

### G5 — resistant brute
Same attack sequence against high-poise fixture.
Expected: fewer/no threshold breaks without sufficient heavy pressure.

## H. Bleed

### H1 — normal apply
Expected: bounded bleed state created once and ticks server-side.

### H2 — repeated apply
Expected: follows cap/refresh policy; never unlimited stacking.

### H3 — resistant anatomy
Expected: reduced authored effect.

### H4 — immune anatomy
Expected: no bleed record/ticks.

### H5 — creature death
Expected: future bleed ticks stop; no corpse damage/credit.

## I. Cleave

### I1 — two valid targets
Expected: both independently hit within cap.

### I2 — third over cap
Expected: deterministic target ordering chooses only cap count.

### I3 — target behind
Expected: not hit because primary was hit.

### I4 — target behind wall
Expected: independently rejected where LOS applies.

### I5 — forged client target list
Expected: ignored; server builds candidates.

## J. Noise

### J1 — fast hit noise
Expected: semantic low/moderate combat noise event emitted once.

### J2 — heavy impact
Expected: stronger fixture noise event emitted once.

### J3 — replay
Replay action callback.
Expected: no duplicate meaningful noise.

### J4 — hearing reaction
Nearby hearing-capable AI consumes noise.
Expected: behaviour can investigate/alert according to profile; Combat does not directly force universal aggro.

## K. Interruption

### K1 — pre-commit interrupt
Qualifying stagger before commit.
Expected: attack terminates; no full committed Vigor; no strike.

### K2 — post-commit interrupt
Qualifying hard interruption after commit.
Expected: Vigor stays spent; active strike terminates/changes only per definition; interrupted recovery applies.

### K3 — light hit
Non-interrupting hit during windup.
Expected: feedback may occur but action continues.

## L. Broken condition

### L1 — break fast blade
Expected: same instance remains equipped; degraded direct/bleed profile; no deletion.

### L2 — break heavy
Expected: same instance remains; major stagger/cleave/damage degradation; Vigor does not become beneficially cheap.

### L3 — advanced action lock
If Broken definition disables advanced action, request it.
Expected: reject with stable reason; basic desperate action may remain.

### L4 — death Broken
Expected: wake with exact Broken state.

## M. Durability

### M1 — one hit one wear
Expected: one logical wear event.

### M2 — paired carrier double callback
Expected: one wear event against logical pair.

### M3 — rejected hit
Expected: no successful-strike wear; optional miss/world-impact wear only if separately authored/reliably detected.

## N. Creature reactions

### N1 — light hit
Expected: feedback without mandatory hard interrupt.

### N2 — stagger break
Expected: creature-specific valid stagger reaction.

### N3 — Caller interruption
Fixture special action requires threshold break.
Expected: ordinary light hit does not cancel; valid stagger break does.

### N4 — ally death
Expected: one semantic event; nearby AI reaction depends on relation/hearing/visibility data.

## O. Creature attacks

### O1 — telegraphed attack
Expected: windup visible/presented; damage only during authored active geometry.

### O2 — player leaves range
Expected: creature swing can whiff; no homing instant damage.

### O3 — creature staggered pre-commit
Expected: attack cancels if definition allows.

## P. Death/race

### P1 — player dies before own commit
Expected: action cancels; no future strike.

### P2 — player dies after commit before active
V1 expected: no refund; future unresolved player strike cancelled once DEAD wins.

### P3 — target dies from first strike of sequence
Expected: later strike cannot farm dead target; may resolve another valid target only if action definition explicitly allows retargeting.

## Q. Run/reset/random spawns

### Q1 — instance teardown
Expected: all bleed/stagger/actions/alert combat state destroyed with run.

### Q2 — next run same creature archetype elsewhere
Expected: same combat profile works at randomized legal spawn; no fixed-coordinate assumptions.

## R. Authority

### R1 — forged damage
Client sends desired damage.
Expected: ignored/rejected.

### R2 — forged bleed/stagger
Expected: ignored/rejected.

### R3 — forged recovery complete
Expected: server timestamp wins.

### R4 — forged long-range target
Expected: spatial resolver rejects.

## Definition of done

Combat v1 is not ready for content expansion until A–R pass with trace tooling proving action ID, phase timing, Vigor commit, target geometry, hit result, stagger/bleed, noise, wear, reaction and recovery.