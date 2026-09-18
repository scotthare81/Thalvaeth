# Extraction gameplay specification

## Purpose

Extraction is a gameplay event, not merely a database commit. It converts run-risk inventory into persistent home ownership and ends the run exactly once.

## Core rules

- extraction is server-authoritative;
- only an ACTIVE run may begin extraction;
- an authored extraction site/gate must be valid and available;
- extraction can require presence, interaction and a bounded vulnerable completion sequence;
- exact duration/audio/visual timing is balance/presentation data;
- success is atomic;
- death/extraction race uses the first valid terminal guard;
- disconnect is not success.

## Extraction sites

Rotwood may support multiple authored extraction sites, but v1 can prove one. A site defines:
- stable key;
- world position/volume;
- availability rules;
- activation interaction;
- noise/Director hooks;
- interruption rules;
- completion destination.

Do not randomly invent extraction coordinates. If availability varies by run, selection is from authored sites and stored in the run manifest/state.

## Player knowledge

The game must clearly teach how extraction is located/recognized. Whether every active site is known at run start remains content/UI policy; no hidden UI leak from inactive/unknown sites.

## Activation

Canonical flow:
1. player requests interaction at legal site;
2. server validates run/site/player state;
3. site enters extraction attempt;
4. player performs authored vulnerable action/channel if configured;
5. movement/damage/combat interruption follows explicit rules;
6. on completion server enters FINALIZING;
7. freeze conflicting mutations;
8. reconcile reservations/inventory;
9. validate haul/provenance;
10. promote extracted items/materials to home/persistent ownership;
11. write terminal ledger;
12. reset run-local Survival/Director state;
13. teleport to authored Monastery arrival;
14. mark EXTRACTED/complete presentation;
15. destroy run instance/state.

## Failure/interruption

An interrupted attempt leaves the run ACTIVE unless death wins. No partial promotion occurs. Items remain in run domains.

Repeated interaction cannot duplicate finalization.

## Sound and Director

Starting/operating an extraction may create authored sound/pressure. This must use unified Noise/Director contracts. It cannot directly aggro every creature or guarantee a spawned enemy.

## UI/presentation

No stock WoW quest-complete framing. Player needs restrained feedback for:
- site usable/not usable;
- extraction started;
- interruption;
- completion.

Exact cinematic/fade/arrival presentation remains art direction.

## Arrival

Backend requires a stable `home_extraction_arrival` destination distinct from death's `home_recovery_spawn`. Exact physical Monastery room/location is resolved by Monastery world design.

## Acceptance

Tests cover legal activation, interruption, death race, disconnect, duplicate completion, inventory promotion, home arrival, run teardown, Director/noise integration and no partial extraction.