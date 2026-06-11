# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file, zero-dependency browser app (`index.html`) that generates randomized Helldivers 2 loadouts. No build step, no package manager, no framework — open `index.html` directly in a browser to run it.

## Development Workflow

**Running the app:** Open `index.html` in a browser. There is no dev server, build step, or compilation required.

**Testing:** No automated test suite exists. All verification is done by loading the page in a browser and exercising the UI manually.

**Linting/Formatting:** No tooling configured. The entire codebase is in one ~2700-line HTML file.

## Architecture

The entire application lives in `index.html`, organized roughly as:

| Lines | Contents |
|-------|----------|
| 1–580 | `<style>` block — CSS custom properties, layout, card/button styles |
| 582–880 | `DB` object — item database (support weapons, primaries, secondaries, grenades, armor, backpacks, vehicles, orbitals, eagles, sentries, boosters) |
| 883–1166 | Tag inference engine (`buildItemStats`, `suggestTags`, `normalizeDatabaseTags`) |
| 1168–1420 | Build rules, scoring weights, role/theme constants |
| 1422–1521 | `scoreItem()` and `pickScored()` — weighted random selection |
| 1523–1650 | Helper utilities and filtering logic |
| 1650–1907 | `generateBuildOnce()` and `randomize()` — main build generation |
| 1909–2100 | Render functions (`render`, `renderLoadout`, `renderBooster`) |
| 2103–2543 | `ITEM_SRC` and `ITEM_PAGE` maps (warbond/source metadata) |
| 2545–2705 | Arsenal manager UI (`toggleItem`, `saveDisabled`, `renderArsenal`) |
| 2707–end | `<body>` HTML — UI panels, role pills, theme checkboxes, stratagem cards |

## Core Systems

### Tag Inference (not hand-curated)

Item tags are **not manually assigned per-item**. At startup, `normalizeDatabaseTags()` calls `suggestTags()` on every item, which reads the item's `name` and `desc` fields and infers tags using regex patterns. `buildItemStats()` then derives numeric stat scores (`penetration`, `rateOfFire`, `areaCoverage`, etc.) from those same text fields. This means adding a new item only requires setting `name`, `tags` (base tags), and `desc` — the rest is derived automatically.

### Build Roles and Themes

- **Roles** (mutually exclusive, stored in `activeBuildTags` Set): `any`, `balanced`, `at`, `chaff`, `support`, `stealth`, `mobility`, `close`, `denial`, `defense`
- **Themes** (multi-select up to 2, stored in `activeThemeTags` Set): `ballistic`, `fire`, `laser`, `explosive`, `arc`, `plasma`, `gas`, `melee`, `stun`

`BUILD_RULES` defines minimum tag requirements that a completed build must satisfy per role. `buildSatisfiesRules()` validates after generation; if it fails after 60 attempts, a warning is shown but the build is returned anyway.

### Scoring and Selection

`scoreItem(item, role, themes, armorWeight)` returns a numeric score. `pickScored(pool, scoreFunc)` does a weighted random selection (not just highest score) so results vary. Items disabled via the Arsenal UI are excluded from all pools via the `disabledItems` Set, persisted to `localStorage` under key `hd2_disabled`.

### Item Database Schema

```javascript
{
  name: "AR-23 Liberator",     // unique key, used in ITEM_SRC/ITEM_PAGE lookups
  tags: ["ballistic"],         // base tags; suggestTags() adds more at runtime
  desc: "...",                 // used by tag inference — content matters
  // armor only:
  weight: "Light|Medium|Heavy",
  passive: "string",
  // support weapons:
  hasBackpack: true,           // skips backpack slot when selected
  closer: true,                // can close bug holes/bot fabricators
  isMech: true,                // vehicle slot item
}
```

### Adding New Items

1. Add the item object to the appropriate `DB` array (e.g., `DB.primary`, `DB.orbital`).
2. Set `name`, `tags`, and `desc` — ensure `desc` contains keywords the inference engine uses (e.g., "armor-piercing", "fire damage", "stagger").
3. Add an entry to `ITEM_SRC` (maps `name → warbond/source string`).
4. Add an entry to `ITEM_PAGE` (maps `name → page number within warbond`).
5. Reload the page — tags and stats are computed at runtime.

## State Management

All state is in global JS variables:
- `activeBuildTags` / `activeThemeTags` — current role/theme selections
- `lockedSlots`, `lockedLoadout`, `lockedBooster` — user-locked items (not re-rolled)
- `currentBuild`, `currentLoadout`, `currentBooster` — last rendered build
- `disabledItems` (Set) — persisted to `localStorage`; items excluded from randomization

## CSS Conventions

CSS custom properties are defined on `:root` (lines 12–25). The color palette uses deep grays with a gold accent (`--accent`). Card grids use `repeat(auto-fit, minmax(190px, 1fr))`. Tag/role pills use class names like `.tag-chaff`, `.tag-at`, `.role-active`.
