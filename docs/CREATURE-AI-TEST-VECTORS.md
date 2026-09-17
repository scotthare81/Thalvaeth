# Creature AI deterministic test vectors

Acceptance cases for shared perception/search/social AI and the locked 90001–90010 behaviours.

## A. Vision and knowledge

### A1 — direct sight
Creature with ordinary visual profile sees player in legal LOS/range.
Expected: knowledge updates from current server observation; can acquire according to archetype.

### A2 — player behind wall
Expected: no direct visual acquisition through obstruction.

### A3 — LOS lost
Engaged creature sees player turn behind cover.
Expected: remembers last seen position, not continuously updated hidden player position.

### A4 — player silently relocates
Creature searches old position while player moves elsewhere unseen/silent.
Expected: creature can fail search; it does not magically redirect.

### A5 — reacquire
Player becomes visible during search.
Expected: current evidence updates and creature can re-engage.

## B. Noise

### B1 — investigate sound
Unaware noise-curious Scavenger receives valid nearby noise without seeing player.
Expected: investigates sound position; does not instantly receive live target lock.

### B2 — player leaves sound position
Expected: Scavenger reaches/investigates stale evidence if no new stimulus.

### B3 — weak/irrelevant noise
Expected: profile may ignore/orient without full investigation.

### B4 — heavy combat noise
Relevant creatures receive stronger stimulus; reactions remain profile-specific.
Expected: no generic `aggro all in radius` behaviour.

## C. Search/disengage

### C1 — bounded search
Lose player, search last evidence, no reacquisition.
Expected: SEARCHING eventually ends; creature resumes authored local behaviour.

### C2 — new noise during search
Expected: evidence updates according to profile and search redirects to valid new sound, not hidden player truth.

### C3 — disengage health
Wounded ordinary creature disengages.
Expected: run health remains wounded unless a specific content rule says otherwise.

## D. Social information

### D1 — nearby ally hears fight
Expected: may investigate/assist according to hearing/social profile; no psychic exact target coordinates.

### D2 — ally behind obstruction/out of stimulus
Expected: remains unaffected unless another valid channel informs it.

### D3 — death event replay
Expected: ally/Director death stimulus emitted once.

## E. Ash Sleeper 90001

### E1 — walking past
Player within 10y, not in combat, genuinely walking.
Expected: remains dormant/passive.

### E2 — running past
Player within 10y and running.
Expected: wake latches once; stands/aggresses; wake action begins.

### E3 — combat nearby
Player within 10y and in combat.
Expected: wakes even if movement state otherwise safe.

### E4 — ordinary distant noise
Expected under current locked rule: does not replace special wake condition with generic chase.

### E5 — reconnect after wake
Expected: cannot reset to exploitable fresh dormant state if run persistence records wake state.

## F. Cellar Scavenger 90002

### F1 — noise curiosity
Expected: investigate before direct target lock when only sound known.

### F2 — pair assistance
One Scavenger enters combat visibly/audibly near partner.
Expected: partner response comes from valid social/noise evidence.

### F3 — search escape
Player breaks LOS and relocates quietly.
Expected: possible successful escape.

## G. Catwalk Prowler 90003

### G1 — outside trigger
Player >6y or without LOS.
Expected: remains perched/passive.

### G2 — trigger
Player enters legal 6y+LOS condition.
Expected: one leap sequence; grounded combat follows.

### G3 — disengage after leap
Expected: does not teleport/re-perch and cannot leap again.

### G4 — reconnect
Expected: spent leap state remains spent in active run.

## H. Ash Caller 90004

### H1 — normal engage
Expected: call wind-up begins around locked timing; successful commit requests +2 Scavenger wave once.

### H2 — killed before call
Expected: engage call does not complete; death wave +3 still occurs once.

### H3 — valid interrupt
Expected: call fails only if Combat says authored interrupt condition succeeded before commit.

### H4 — light non-interrupt hit
Expected: does not automatically cancel call merely because damage landed.

### H5 — death/reconnect replay
Expected: no duplicate +2/+3 wave event.

## I. Patchwork Brute 90005

### I1 — within territory
Expected: engages/attacks inside authored post territory.

### I2 — dragged beyond 12y
Expected: disengages/returns to generated home anchor according to locked rule.

### I3 — keeps wounds
Damage to 60%, force leash return.
Expected: returns still at 60% (subject only to explicitly authored ongoing effects), not full health.

### I4 — generated anchor
Spawn Brute at another legal procedural post node.
Expected: leash is relative to that node, not old hard-coded coordinates.

## J. Broken Snare 90006

### J1 — target >5y + LOS
At hook timing.
Expected: Hook can be selected/committed.

### J2 — target closes inside 5y
Expected: Hook condition fails; AI chooses legal alternative/waits.

### J3 — LOS broken
Expected: cannot hook through wall based on stale exact position.

### J4 — hook replay
Expected: one displacement/action per valid committed hook.

## K. Edge Stalker 90007

### K1 — pacing
Nearest player in authored range/context.
Expected: attempts to maintain ~22–26y using navigable positions.

### K2 — first solid hit
Expected: takes damage, enters break-off/withdrawal, does not generic-evade/full-heal.

### K3 — re-stalk
After withdrawal/~8s intent.
Expected: resumes stalking with same wounded HP.

### K4 — obstacle
Expected: pathing respects world; no teleport through obstruction to maintain perfect distance.

## L. Drifter 90008

### L1 — ambient pass
Player skirts at non-acquisition conditions.
Expected: continues slow/wide wander; does not behave like Ghoul.

### L2 — weak noise
Expected: low reaction/hesitation consistent with profile.

### L3 — confirmed engagement
Expected: simple melee, no invented special cast.

## M. Stumbler 90009

### M1 — doorway role
Expected: slow pursuit/body positioning makes doorway pressure meaningful.

### M2 — wide swing
Expected: authored frontal geometry; player outside arc is not hit.

### M3 — pull out
Player creates valid movement/aggro that moves it from doorway.
Expected: counterplay works; it does not teleport back during active pursuit.

## N. Ghoul 90010

### N1 — confirmed rush
Expected: fast pursuit once target genuinely acquired.

### N2 — LOS escape
Despite speed, hidden player position is not continuously known after LOS loss.

### N3 — below 40%
Expected: Frenzy/Enrage transition occurs once/according to authored state and persists correctly through active-run reconnect.

### N4 — attack geometry
Expected: fast cadence still obeys Combat wind-up/active/recovery and can whiff.

## O. Director boundary

### O1 — high Director heat
Creature has no sensory evidence of player.
Expected: Director may choose future pressure but does not inject exact target coordinates into this creature.

### O2 — spawn selection
New creature spawned at legal procedural node.
Expected: initializes archetype AI from generated context; no fixed DB-spawn dependency.

## P. Path failure

### P1 — invalid investigation point
Expected: abandon/choose bounded alternate; no teleport to player.

### P2 — return-home path failure
Expected: content-specific safe fallback/diagnostic; generic AI never teleports into target pursuit.

## Q. Journal

### Q1 — unseen ability
Creature has ability in server definition but never performs it observably.
Expected: Journal/addon receives no hidden ability revelation.

### Q2 — witnessed distinctive action
Expected: server publishes observation; Journal decides promotion.

### Q3 — internal key leakage
Expected: normal combat/addon payload does not reveal unknown raw ability/archetype keys.

## R. Run lifecycle

### R1 — killed creature reconnect
Expected: remains dead/absent in same ACTIVE run.

### R2 — wounded Brute restart
Expected: authoritative run health reconstructs wounded, not fresh.

### R3 — terminal teardown
Expected: awareness/search/HP/death/generated identities are discarded with run instance; persistent Journal knowledge survives.

### R4 — next run
Expected: new run may place different legal population; AI behaviour remains archetype-correct wherever spawned.

## Definition of done

Creature AI is ready for broad authoring when A–R pass with trace output proving **what the creature knew, why it changed state, what evidence it acted on, which action it selected, and why it disengaged**.