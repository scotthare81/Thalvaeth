# Equipment + paired weapon test vectors

Deterministic acceptance cases for `EQUIPMENT-CONTRACT.md`.

## A. Paired weapon identity

### A1 — pair creation
Upgrade a legal single/branch weapon into Paired Daggers.

Expected: one logical gear instance represents the pair; expected main/off-hand carriers both resolve to it; no second independently progressing Gear record exists.

### A2 — relog pair
Relog with paired weapon equipped.

Expected: same logical ID/node/quality/durability; two expected carriers reconstructed; no reroll.

### A3 — missing off-hand carrier
Delete/corrupt the presentation off-hand carrier in a test fixture, leaving logical instance intact.

Expected: login/reconciliation rebuilds carrier. Gear is not lost and no new logical instance is created.

### A4 — duplicate carrier
Inject duplicate off-hand presentation carrier.

Expected: reconciliation removes/quarantines duplicate safely; exactly one logical pair remains.

### A5 — pair upgrade
Upgrade Paired Daggers to next paired node.

Expected: one transaction; one material set; one quality/result transition; both visible carriers change together.

### A6 — pair repair
Repair worn paired weapon.

Expected: one repair target and one resulting durability pool. Cannot separately repair each native hand carrier.

## B. Weapon roles

### B1 — heavy occupies weapon role
Equip Greatsword/Greataxe.

Expected: one logical heavy instance active; no fake off-hand persistent weapon required.

### B2 — heavy + skinning knife
Heavy weapon equipped; persistent skinning knife available through legal tool state.

Expected: combat uses heavy weapon; butcher action can resolve knife capability without unequipping/deleting heavy weapon.

### B3 — greatsword no butcher
No eligible knife/tool; Greatsword equipped.

Expected: butcher action rejects even though player has a bladed weapon.

### B4 — greataxe chop
Greataxe definition has chop capability.

Expected: chopping accepts it and applies wear to greataxe logical instance.

## C. Switching and bulk

### C1 — stow dagger
Switch from dagger to another legal weapon with enough bag capacity.

Expected: dagger becomes carried, its authored bulk applies, new weapon becomes worn/free.

### C2 — stow heavy into full bag
Heavy weapon equipped; switch would require storing bulk 4 but only 3 free.

Expected: switch fails atomically; heavy remains equipped; target remains carried; no overflow.

### C3 — paired set bulk
Unequip paired weapon set.

Expected: complete set consumes its single authored carried-bulk value, not double-counted per presentation carrier.

### C4 — interrupted swap
Begin future channelled swap then interrupt before commit.

Expected: original loadout remains authoritative; no weapon duplicated/missing.

## D. Broken state

### D1 — break equipped weapon
Drive active weapon to zero durability.

Expected: remains equipped; Broken profile applies; no replacement created.

### D2 — break paired weapon
Drive paired weapon to zero.

Expected: complete logical pair is Broken; carriers remain; no one-hand healthy/one-hand broken divergence in v1.

### D3 — death while Broken
Die with Broken weapon equipped.

Expected: wake with same logical weapon still Broken.

## E. Armour/layers

### E1 — quilted under mail
Equip logical quilted underlayer and mail body combination using bridge representation.

Expected: both logical gear instances persist even if presentation requires one combined native body visual.

### E2 — remove required underlayer
Attempt to remove underlayer while outer layer definition requires it.

Expected: reject or perform an explicitly legal coordinated removal with destination preflight; never leave invalid hidden state.

### E3 — mixed armour
Wear heavy body, light head, light boots.

Expected: derived profile sums actual authored pieces; no automatic conversion to all-heavy.

## F. Pack

### F1 — pack upgrade
Upgrade pack to larger-capacity node.

Expected: capacity changes only at authoritative commit.

### F2 — pack downgrade overflow
Current inventory exceeds capacity of requested lower pack.

Expected: downgrade/unequip fails before mutation/material consumption.

### F3 — pack carrier corruption
Visible backpack model/item missing but logical pack remains equipped.

Expected: capacity remains authoritative and presentation is reconstructed; player does not gain/lose capacity from visual corruption.

## G. Charms

### G1 — two slots
Equip two legal charms.

Expected: two logical slots occupied; effects delegated to Charm/Aptitude authority.

### G2 — third charm
Attempt third simultaneous charm without future capacity upgrade.

Expected: reject; no hidden third effect.

## H. Death/run

### H1 — normal death
Die with paired weapon, armour, pack and charms worn.

Expected: worn persistent loadout survives with exact identities/quality/durability. Run haul loss presentation does not drop/delete persistent worn gear.

### H2 — extraction
Extract with worn gear in damaged condition.

Expected: exact condition persists; extraction is not free repair.

### H3 — reconnect active run
Disconnect/reconnect with paired weapon equipped.

Expected: active run continues; same logical equipment reconstructed; no carrier duplication.

## I. Native/addon bypass

### I1 — native drag invalid
Attempt native-client drag that would violate logical role or bag capacity.

Expected: server rejects/reconciles; native slot state cannot become authority.

### I2 — forged addon request
Send request naming a carrier GUID as though it were a separate paired weapon.

Expected: server resolves/rejects through logical gear identity; cannot mutate half a pair.

### I3 — hidden carrier deletion
Client/addon fails to display one carrier.

Expected: no authoritative gear mutation.

## Definition of done

Equipment foundation is ready for Combat when A–I pass and diagnostics can show logical gear instance → logical role → carrier mapping, reservations, bulk preflight and reconstruction status.