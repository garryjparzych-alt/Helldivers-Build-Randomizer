# Helldivers Build Randomizer

A browser-based Helldivers 2 loadout randomizer built as a single HTML file.

This project started as a personal tool for generating themed builds and testing unusual combinations. It now includes build-role preferences, stricter derived scoring for things like anti-tank and chaff clearing, and curated armor/loadout data.

## Why I Built It

I find that I always default to the same exact loadout every single time I play Helldivers 2.  I might change it up to a secondary loadout every once in awhile, but I wanted something that would encourage me to try different weapons, armor, and strategems.  

This was also a way for me to find out how to write an app as simply as I could make one.  I had no HTML experience before making this, and my goal was for personal development.  I did use Claude and Chat-GPT to help me build it, so I cannot claim I made this all by myself.

## Highlights

- Single-file HTML app with no framework dependency
- Role-aware randomization for anti-tank, support, defense, mobility, stealth, and more
- Stat-backed derived scoring instead of relying only on hand-written tags
- Arsenal management for enabling and disabling owned items

## Tech Notes

The app is intentionally lightweight and self-contained:

- `HTML` for structure
- `CSS` for styling
- `JavaScript` for the data model, filtering, scoring, and randomization logic

The project evolved through multiple refactor passes, including:

- cleanup of inconsistent tags
- migration toward stat-backed item records
- derived role scoring
- improved armor data modeling


## Attribution

Original concept and implementation by Garry Parzych.

Released under the MIT License. See [LICENSE](./LICENSE).

## Disclaimer

This is a fan-made utility project and is not affiliated with Arrowhead Game Studios or Sony. Helldivers and related names are the property of their respective owners.
