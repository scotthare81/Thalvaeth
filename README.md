# Thal'vaeth

**Solo survival-horror extraction on the AzerothCore WotLK 3.3.5a engine. Original IP.**

You are a **Remnant** — one of the last who still walks the Wild. Leave the **Thal'vaeth Monastery**, run a walled district (first: **Rotwood**), scavenge and survive, and extract before Hunger, Thirst, Infection, or a wound you can't stitch drops you. Death costs your **haul**, not your character. Then you go again.

> **Thal'vaeth** — engine only. Every player-facing name, item, and system is original IP (no WoW names in what players see).

## Design docs (canon) → [`docs/`](docs/)

**Survival & the loop**
- [SURVIVAL.md](docs/SURVIVAL.md) — Hunger · Thirst · Vigor · Infection
- [SURVIVAL-IMPLEMENTATION-SPEC.md](docs/SURVIVAL-IMPLEMENTATION-SPEC.md) — exact server-owned meter rules, tuning defaults, thresholds, channels and persistence
- [SURVIVAL-CRAFTING-V1-SLICE.md](docs/SURVIVAL-CRAFTING-V1-SLICE.md) — pinned first playable Survival + Crafting slice
- [SURVIVAL-CRAFTING-TEST-VECTORS.md](docs/SURVIVAL-CRAFTING-TEST-VECTORS.md) — deterministic cross-system acceptance cases
- [SURVIVAL-CRAFTING-CODING-PLAN.md](docs/SURVIVAL-CRAFTING-CODING-PLAN.md) — staged implementation/PR sequence
- [DIRECTOR.md](docs/DIRECTOR.md) — run pacing (the invisible Stress Director)
- [MAPS.md](docs/MAPS.md) · [RUN-GATES.md](docs/RUN-GATES.md) — Monastery, Rotwood, gates
- [CHAR-CREATE.md](docs/CHAR-CREATE.md) · [APTITUDES.md](docs/APTITUDES.md) — the Remnant + charms

**Journal & discovery**
- [JOURNAL-RECORD.md](docs/JOURNAL-RECORD.md) — tabbed Journal Record: Creatures · Survival · Gear; silhouettes, obscured knowledge, near-miss hints
- [JOURNAL-CONTENT.md](docs/JOURNAL-CONTENT.md) — concrete v1 Journal entries and reveal states
- [JOURNAL-HINTS.md](docs/JOURNAL-HINTS.md) — canonical near-miss hint library and persistence rules
- [DISCOVERY-SOLUTIONS.md](docs/DISCOVERY-SOLUTIONS.md) — authoritative recipe truth, failure precedence, corrections, hint mappings and successful outcomes
- [JOURNAL-IMPLEMENTATION-SPEC.md](docs/JOURNAL-IMPLEMENTATION-SPEC.md) — server authority, persistence, keys, sync protocol and experiment evaluator contract
- [JOURNAL-UI-SPEC.md](docs/JOURNAL-UI-SPEC.md) — layout, silhouettes, tabs, field notes, gear trees and interaction rules
- [JOURNAL-V1-SLICE.md](docs/JOURNAL-V1-SLICE.md) — exact first playable slice and acceptance scenarios
- [JOURNAL-DATA-MODEL.md](docs/JOURNAL-DATA-MODEL.md) — persistence split, monotonic grants, character isolation and migration rules
- [JOURNAL-WIRE-PROTOCOL.md](docs/JOURNAL-WIRE-PROTOCOL.md) — v1 handshake, snapshots, deltas and resync behaviour
- [JOURNAL-KEY-REGISTRY.md](docs/JOURNAL-KEY-REGISTRY.md) — canonical stable discovery/hint/gear keys
- [JOURNAL-TEST-VECTORS.md](docs/JOURNAL-TEST-VECTORS.md) — deterministic backend, protocol and discovery test cases
- [JOURNAL-CODING-PLAN.md](docs/JOURNAL-CODING-PLAN.md) — staged implementation milestones and PR strategy
- [CREATURE-JOURNAL.md](docs/CREATURE-JOURNAL.md) — creature-specific Unknown → Sighted → Engaged progression

**Creatures**
- [CREATURES.md](docs/CREATURES.md) — roster, AI roles, journal prose

**Items · crafting · gear · economy**
- [MATERIALS.md](docs/MATERIALS.md) → [CRAFTING.md](docs/CRAFTING.md) → [ITEMS.md](docs/ITEMS.md) → [GEAR.md](docs/GEAR.md)
- [CRAFTING-IMPLEMENTATION-SPEC.md](docs/CRAFTING-IMPLEMENTATION-SPEC.md) — deterministic stations, attempts, recipes, near-misses, consumption and quality contract
- [ECONOMY.md](docs/ECONOMY.md) — no coin, tiered barter, item bulk
- [NAMES.md](docs/NAMES.md) — locked player-facing names · [CONTENT.md](docs/CONTENT.md) — item band + trades
- [TODO.md](docs/TODO.md) — open work

**Ops & process**
- [DEPLOY.md](docs/DEPLOY.md) — server isolation, ports, tree
- [REPO.md](docs/REPO.md) · [AGENT-INSTRUCTIONS.md](docs/AGENT-INSTRUCTIONS.md)

## Repo layout

| Path | What |
|------|------|
| `docs/` | Design canon (above) + ops |
| `src/mod-thalvaeth/` | AzerothCore module — C++, SQL (`data/sql`), conf, ThalvaethUI addon (`client/`) |
| `scripts/` | Server-tree, module-link, and push helpers |

## Build

Engine only — you need a full **AzerothCore 3.3.5a** checkout plus a client-data extract, with the Thal'vaeth module/scripts/SQL layered on top. See [`docs/DEPLOY.md`](docs/DEPLOY.md). Not buildable from this repo alone.

---

*The earlier co-op "ring network" design is retired; git history preserves it.*
