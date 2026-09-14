# Crafting station contracts

This file pins what each station is allowed to do, what it owns, and what must remain outside the station layer.

## Shared station rules

Every station has:
- stable station key
- interaction radius
- allowed recipe families
- optional required tool state
- optional fuel/resource state
- optional queued-job capability

Stations do not own Journal truth, survival meters, item identity, or discovery persistence. They provide context to the server crafting evaluator.

## Field

Purpose: emergency survival only.

Allowed:
- crude butcher
- campfire roast/boil when a valid fire exists
- crude bandage
- firestart
- crude whetstone sharpen
- temporary patch mend

Disallowed:
- deep brewing
- tanning
- smelting
- full repair
- gear upgrades
- distilling

Field output may have quality caps.

## Butcher block

Purpose: controlled carcass breakdown.

Inputs:
- carcass/raw animal body resource
- valid cutting tool

Outputs:
- meat
- hide/pelt
- sinew
- bone
- fat
- special anatomical material where defined

Butcher yield depends on carcass quality + tool condition + relevant charm/passive, not skill level.

## Cookfire

Purpose:
- cooking
- boiling
- rendering
- preservation by heat/smoke
- crude liquid processing

Owns fuel/heat context, not recipe truth.

Future heat levels may expose `low`, `steady`, `high`; v1 may use coarse categories.

## Stitch table

Purpose:
- bandages / Stitch Kits
- textile assembly
- leather assembly after tanning
- cloth/leather repair
- padding and waterproofed garments

Does not tan raw hide itself unless recipe explicitly uses a tanning workflow attached to the table.

## Ash-still

Purpose:
- teas
- tinctures
- salves
- washes
- distillation
- lye/potash and apothecary refinement

Must enforce Brewing/Distilling/Advanced Apothecary gates where relevant.

## Charcoal pit

Purpose:
- low-air conversion of suitable Deadwood into Charcoal

Open flame never substitutes.

Outputs:
- Charcoal on valid process
- Ash or poor byproduct on open/incorrect burn path where defined

## Forge

Purpose:
- smelting
- steel/alloy work
- blades
- plates
- metal repair
- metal tool heads/components

Requires Charcoal gate before metalworking.

Forge recipes may require explicit heat category and hammer/anvil context.

## Workbench

Purpose:
- assembly
- hafting
- buckles/rivets/tool frames
- traps
- gear frames
- non-forge structural repairs

Workbench is the general assembly layer, not a substitute for Forge/Stitch/Ash-still transformation steps.

## Station locking

For v1, one player using a station does not need to globally lock it unless a recipe creates a mutable station job.

Queued jobs, once introduced, need:
- owner GUID
- recipe key
- start/end time
- reserved inputs
- station instance key
- completion state

## Failure behaviour

Wrong station should usually fail before consumption and may generate a hint only when the attempt is meaningfully close to a known/inferred process.

## Acceptance

A station implementation is valid if:
- it exposes only its approved families;
- it cannot bypass discovery gates;
- it cannot directly grant Journal knowledge without the evaluator;
- it cannot duplicate reserved inputs;
- future queued jobs can be added without changing recipe identity semantics.
