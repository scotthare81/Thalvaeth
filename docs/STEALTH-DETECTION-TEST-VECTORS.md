# Stealth + detection deterministic test vectors

Acceptance cases for player stealth, observer-specific detection, unified sound, concealment, distractions and Director boundaries.

## A. Movement noise

### A1 — still
+Player remains still with no active noisy action.
+Expected: no repeated footstep noise; stillness does not equal invisibility.
+
+### A2 — walk
+Expected: server emits/derives low movement evidence according to surface/Gear; quieter than run under same conditions.
+
+### A3 — run
+Expected: stronger movement noise/visual motion than walk; Ash Sleeper special trigger can see authoritative run state.
+
+### A4 — Hustle
+Expected: stronger authored movement pressure than walk; Survival owns Vigor, Stealth owns evidence/noise consequence.
+
+### A5 — spoofed client state
+Client claims quiet/walk while server movement is running.
+Expected: server run state wins.
+
+## B. Armour/equipment
+
+### B1 — cloth vs plate walk
+Same surface/path/speed.
+Expected: plate derived movement/equipment noise > cloth according to data.
+
+### B2 — plate still
+Expected: no continuous loud-radius aura merely because plate is equipped.
+
+### B3 — emitted sound then gear swap
+Expected: old NoiseEvent remains based on state at emission; swap cannot retroactively quiet it.
+
+### B4 — condition
+Damage armour.
+Expected: no accidental stealth advantage unless explicitly authored.
+
+## C. Sound evidence
+
+### C1 — heard footstep without sight
+Expected: listener gets sound position/evidence, not live target coordinates.
+
+### C2 — player relocates quietly
+Expected: listener investigates old sound position and can fail to find player.
+
+### C3 — repeated weak sounds
+Expected: observer-specific suspicion can build according to profile; no universal instant aggro.
+
+### C4 — strong close sound
+Expected: rapid alert/investigation where profile permits, but target identity is not fabricated if source was unseen.
+
+### C5 — duplicate event replay
+Expected: same event/action cannot produce duplicate meaningful sound evidence.
+
+## D. Propagation
+
+### D1 — distance attenuation
+Two identical listeners at different distances.
+Expected: nearer receives stronger stimulus, subject to profile.
+
+### D2 — major obstruction
+Authored sound topology says separated/blocked.
+Expected: reduced/blocked stimulus according to propagation data.
+
+### D3 — unrelated distant creature
+Expected: no full-world aggro or listener scan result outside plausible spatial query.
+
+## E. Vision
+
+### E1 — close clear LOS
+Expected: identification/engagement escalates rapidly according to aggressive profile.
+
+### E2 — distant dim movement
+Expected: marginal visual evidence can produce suspicion rather than instant engagement.
+
+### E3 — continued exposure
+Expected: certainty builds while legal confirming sight continues.
+
+### E4 — break LOS before confirmation
+Expected: visual evidence stops; observer acts on last-known evidence/decay rather than hidden player transform.
+
+### E5 — LOS flicker
+One-tick obstruction/flicker.
+Expected: hysteresis prevents state jitter but does not grant wallhack.
+
+## F. Light
+
+### F1 — same distance, dim vs bright authored zone
+Expected: bright condition yields stronger/faster visual evidence under same observer profile.
+
+### F2 — darkness at close range
+Expected: darkness does not make player invisible in clear arm's-length LOS.
+
+### F3 — client brightness/gamma change
+Expected: no authority effect; server authored visibility state wins.
+
+### F4 — portable light
+If enabled, active player light changes authored visibility cue/profile once; addon cannot suppress it.
+
+## G. Concealment
+
+### G1 — dense authored concealment
+Expected: visual contribution reduced according to profile; hearing unaffected.
+
+### G2 — loud movement while concealed
+Expected: creature can investigate sound despite weak visual evidence.
+
+### G3 — leave concealment
+Expected: visual modifier updates from authoritative world position/state.
+
+### G4 — client claims concealment
+Expected: ignored.
+
+## H. Crouch
+
+### H1 — crouch in suitable cover
+If crouch implemented, expected: authored posture modifier applies.
+
+### H2 — crouch in open close LOS
+Expected: creature can still identify player; crouch is not invisibility.
+
+### H3 — crouch/run contradiction
+Expected: invalid/incompatible state resolved by server movement authority.
+
+## I. Still Breath
+
+### I1 — activate while still
+Expected: avoidable body/equipment noise reduced/suppressed according to action definition; no evidence erase.
+
+### I2 — move during Still Breath
+Expected: action ends; movement evidence resumes.
+
+### I3 — direct LOS
+Expected: Still Breath does not make player invisible.
+
+### I4 — weak suspicion
+Player breaks sight and holds still.
+Expected: weak evidence may decay naturally if no new stimulus; not instant reset.
+
+## J. Distractions
+
+### J1 — throw object unseen
+Impact creates `noise.distraction` at actual impact position.
+Expected: eligible suspicious/unaware creature may investigate impact, not thrower's live position.
+
+### J2 — player relocates after throw
+Expected: investigation remains tied to evidence unless new sight/sound occurs.
+
+### J3 — engaged creature sees player
+Pebble lands behind it.
+Expected: direct strong player evidence is not overwritten by weaker distraction.
+
+### J4 — competing sounds
+Expected: observer selects/updates evidence according to profile/confidence, not hard-coded newest event always wins.
+
+### J5 — repeated abuse
+Expected: attention/confidence rules prevent pathological ping-pong where appropriate without arbitrary global immunity.
+
+## K. Social/investigation propagation
+
+### K1 — investigator reacts to sound
+Second creature notices only investigator movement/state but never original sound.
+Expected: it cannot gain exact player position from nothing.
+
+### K2 — direct witness alerts ally
+Expected: communicated evidence bounded by authored social event; ally response profile-specific.
+
+### K3 — Caller shriek
+Expected: explicit Caller escalation remains stronger authored event and fires idempotently.
+
+## L. Ash Sleeper
+
+### L1 — walk within 10y
+Expected: remains dormant under locked rule.
+
+### L2 — Still Breath within 10y
+Expected: remains dormant absent combat/run trigger.
+
+### L3 — run within 10y
+Expected: wakes once.
+
+### L4 — combat within 10y
+Expected: wakes once.
+
+### L5 — generic nearby distraction
+Expected: generic sound system does not silently replace wake condition.
+
+## M. Edge Stalker
+
+### M1 — legal perception/path
+Expected: pacing uses legitimate world/perception context; no through-wall perfect coordinate tracking.
+
+### M2 — player enters deep concealment
+Expected: behaviour follows reviewed Stalker perception profile; generic code does not invent guaranteed invisibility or omniscience.
+
+### M3 — Director pressure high
+Expected: Director cannot feed exact hidden coordinates to Stalker.
+
+## N. Director
+
+### N1 — loud heavy combat
+Expected: nearby listeners receive sound according to hearing; Director may also receive aggregate pressure event.
+
+### N2 — Director response
+Expected: can alter future pressure/spawn choices but does not set existing creatures ENGAGED.
+
+### N3 — distraction
+Expected: may count as disturbance if designed, but never becomes global exact player location.
+
+## O. Combat/search
+
+### O1 — fight then break LOS
+Expected: creature transitions from ENGAGED toward SEARCHING using last-known evidence; combat state does not wallhack.
+
+### O2 — player runs during escape
+Expected: new footsteps can refresh auditory evidence and compromise escape.
+
+### O3 — player walks/holds still after LOS break
+Expected: possible successful escape if evidence expires and no new confirmation occurs.
+
+## P. Surfaces
+
+### P1 — soft vs metal authored surface
+Same player/Gear/movement.
+Expected: different footstep intensity according to surface data.
+
+### P2 — debris hazard
+Cross authored noisy debris.
+Expected: discrete sound event at crossing; duplicate movement callback cannot spam duplicate event beyond authored cadence.
+
+### P3 — unknown surface
+Expected: safe default profile, not silent/noise explosion.
+
+## Q. Run/reconnect
+
+### Q1 — reconnect during investigation
+Expected: no reroll of population; reconnect cannot be guaranteed awareness wipe exploit.
+
+### Q2 — duplicate distraction packet after reconnect
+Expected: no duplicate event.
+
+### Q3 — terminal run teardown
+Expected: detection records, sound history, investigation and Director run pressure discarded.
+
+### Q4 — next run
+Expected: fresh observer evidence with fresh generated population.
+
+## R. UI/security
+
+### R1 — normal addon payload
+Expected: no exact creature hearing radius, certainty number, hidden profile or Director heat leakage unless explicitly approved for player UX.
+
+### R2 — debug
+GM trace can show exact evidence contributions and transition reasons.
+
+### R3 — hidden creature
+Expected: stealth debug/player UI cannot leak undiscovered creature identity through observer lists in normal play.
+
+## S. Acceptance
+
+The system is ready for implementation when A–R can be traced deterministically and every observer transition can answer: **what evidence did this creature possess, where did that evidence point, how strong was it, what modified it, and why did the creature choose its next state?**
