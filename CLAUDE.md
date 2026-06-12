# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Helldivers Build Randomizer is a **single-file HTML application** (`index.html`, ~2700 lines). There is no build system, no package manager, no dependencies, and no compilation step. The entire app — HTML structure, CSS styling, and JavaScript logic — lives in one file. Open it directly in a browser.

## Development Workflow

- **Run the app**: Open `index.html` in a browser (no server required)
- **No build, lint, or test commands** — there are no tooling configs, no package.json, no test framework
- **Debug panel**: Expand the "Tag Audit / Debug" section at the bottom of the page to inspect tag inference, audit issues, and build validation details

## Architecture

### File Structure

```
index.html   — entire application (HTML + CSS + JS in one file)
README.md
LICENSE
```

The JavaScript inside `index.html` is organized into clearly marked sections using `═══` banner comments.

### Data Layer (`DB` object)

All game items live in a global `DB` object with these pools:

| Key | Contents |
|---|---|
| `DB.support` | Support weapons (heavy hitters, mechs) |
| `DB.backpack` | Backpack/utility stratagems |
| `DB.vehicle` | Vehicles |
| `DB.orbital` | Orbital stratagems |
| `DB.eagle` | Eagle air support |
| `DB.sentry` | Sentries, mines, emplacements |
| `DB.primary` | Primary weapons |
| `DB.secondary` | Sidearms |
| `DB.grenade` | Grenades |
| `DB.armor` | Armor sets (with `weight` and `passive` fields) |
| `DB.booster` | Personal boosters |

Each item has at minimum: `name`, `tags[]`, `desc`. Armor adds `weight` (Light/Medium/Heavy) and `passive`. Some weapons add `hasBackpack`, `closer`, `isMech`, `isVehicle`. These raw records are enriched at runtime with computed `stats` and `derived` fields.

### Tag System

Tags are hand-authored on each item and fall into two categories:

- **Role tags** (gameplay function): `at`, `chaff`, `support`, `stealth`, `mobility`, `close`, `denial`, `defense`, `melee`, `balanced`
- **Theme tags** (weapon type): `ballistic`, `explosive`, `laser`, `fire`, `arc`, `plasma`, `gas`, `stun`, `melee`

### Stat Derivation Engine

`buildItemStats()` parses each item's `desc` string to infer numeric capability scores (e.g. `penetration`, `damage`, `areaCoverage`, `staggerForce`, `demolitionForce`). `deriveRolesFromStats()` and `deriveThemes()` use those scores to suggest additional tags beyond the hand-written ones. This means an item's effective tag set at runtime is broader than what's written in `DB`.

### Randomization Pipeline

1. **`randomize()`** — top-level entry; calls `generateBuildOnce()` up to 60 times, validates each result with `buildSatisfiesRules()`, and keeps the best attempt
2. **`generateBuildOnce()`** — builds one candidate: picks 4 stratagems (respecting locks, vehicle/sentry constraints, fallback pools), then picks loadout (primary, secondary, grenade, armor) and booster
3. **`pickScored(pool, slotType, usedNames)`** — filters a pool, scores each candidate via `scoreItem()` based on the active role/theme selections, then randomly selects from the top-scoring band
4. **`buildSatisfiesRules()`** — validates the complete build against `BUILD_RULES` minimum tag requirements

### Build Preferences & Scoring

`BUILD_PREFERENCES` maps each role to preferred/discouraged tags and preferred armor weights. `scoreItem()` applies bonuses and penalties accordingly. Items are not deterministically ranked — the picker selects randomly from the top band, so the same role can produce varied results.

`BUILD_RULES` defines minimum tag counts (total and stratagem-specific) that a complete build must satisfy. The 60-attempt retry loop exists to satisfy these constraints.

### State

Global variables track current UI state:

- `activeBuildTags` (Set) — selected build role(s)
- `activeThemeTags` (Set) — selected weapon themes (max 2)
- `lockedSlots`, `lockedLoadout`, `lockedBooster` — pinned items that survive rerandomization
- `currentBuild`, `currentLoadout`, `currentBooster` — last generated result
- `disabledItems` (Set) — persisted to `localStorage` under key `hd2_disabled`

### Persistence

Only `disabledItems` (the arsenal manager's enabled/disabled state) is persisted via `localStorage`. Everything else is ephemeral per page load.

## Key Conventions

- **Extend data by adding to `DB`** — new items follow the same `{ name, tags, desc }` shape; the stat engine will infer additional scores automatically from the description text
- **Tag carefully** — hand-written tags are the primary signal for role-based scoring; adding or removing tags directly affects which builds an item appears in
- **`closer: true`** marks items that can destroy bug holes/fabricators — the generation logic requires at least one closer per build if no other objective-clearing item is present
- **`hasBackpack: true`** on support weapons signals that the weapon consumes the backpack slot, which is used to avoid slot conflicts
- **No abstraction for its own sake** — the project is intentionally monolithic; resist splitting into modules unless there is a clear, concrete reason
