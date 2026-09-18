# Home stock, materials and barter economy specification

## Principles

There is **no money economy**. Materials must remain things with uses, not gold wearing different icons.

A valuable extracted material should often compete between:
- immediate consumption;
- repair;
- crafting;
- gear upgrade;
- station/home restoration;
- barter;
- saving for a later known need.

## Home stock

Successful extraction promotes eligible run haul into persistent home stock. Home stock is safe outside runs and server-authoritative.

Stock records preserve semantic data needed by crafting:
- material key/family;
- quantity;
- quality;
- spoil state/timestamp where applicable;
- provenance where meaningful;
- reservation state.

Death never removes already-extracted home stock.

## Material economy

Each material has authored sinks. Avoid single-purpose upgrade tokens where possible.

Material families should connect ecology to systems: plant/fungi, creature harvests, timber/charcoal, hides/leather, reclaimed metal/ore, cloth/fibre, chemical/apothecary inputs and other authored families.

## Barter

Barter exchanges authored goods/services for authored material bundles or items. No hidden conversion to a universal coin value.

A barter offer defines:
- stable offer key;
- provider;
- prerequisites/knowledge;
- requested material set;
- granted item/service/knowledge;
- repeatability/limits;
- stock/refresh policy if any.

Do not generate rotating FOMO shops in v1.

## Economic tension

Repair costs matter because they compete with advancement. Consumables matter because carrying/using them competes with haul. Home restoration can create longer-term material sinks.

Costs should be legible and deterministic once known.

## Anti-exploit

Transactions reserve inputs, validate atomically and commit once. Cancel/disconnect cannot duplicate goods. Quality-sensitive recipes/barters cannot downgrade/upgrade stacks through merge tricks.

## No vendor trash

Every retained material should have a plausible use or explicit barter purpose. If an item exists only to be sold for abstract value, question why it exists.

## Acceptance

V1 economy proves extraction→stock, repair/craft competing sinks, at least one barter decision, deterministic atomic transactions and no universal currency.