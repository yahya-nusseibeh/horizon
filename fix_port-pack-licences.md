Port-pack: licence text stays one comment line, and families match whole words
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

Two Low findings about licences in modpack porting:

1. **Carried mod's manifest.** `carry.rs` writes the jar's licence text and file name into a `#` comment of the carried mod's `horizon-mod.toml`, as they were. The licence comes from the jar's `mods.toml` or `fabric.mod.json`, written by its author. A line break in it ended the comment and added keys of its own above the generated ones (`description`, `authors`, `namespaces`…), or broke the manifest so the carried mod didn't load.
2. **Licence families.** `license_kind` matched families with `contains`. So "Custom, limited redistribution" (holds `mit`) was called permissive, "see template" (`mpl`) copyleft, and anything mentioning Discord (`isc`) permissive. `coverage.md` then counted a restrictive custom licence among the families its own note calls publishable.

## The fix

1. Control characters in the licence and the jar's file name are replaced with spaces.
2. Family keywords match whole alphanumeric words of the lowercased text. SPDX ids still match because `-` and `.` split words (`GPL-3.0-or-later`, `LGPL-2.1`, `Apache-2.0`, `BSD-3-Clause`, `CC0-1.0`). A word may also run a version on to the family's name, with an optional `v` (`GPLv3`, `LGPLv3`, `AGPLv3`, `MPLv2`, `GPL3`), so those stay copyleft. `general public` and `cc-by` keep their phrase match.

## Tests

The `license_kind` test gains `Apache-2.0`, `BSD-3-Clause` and `GPL-3.0-or-later`, and checks that "Custom, limited redistribution", "See the template" and "Ask on Discord" are `custom`. `GPLv3`, `LGPLv3`, `AGPLv3`, `MPLv2`, `GPL3`, `LGPLv2.1` and `gplv2+` are `copyleft`, `Apache2` is `permissive`, and `mplvx` is `custom`.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Not a vanilla feature (modpack porting).

## Other projects' code

None.

## Self-check

`cargo test -p horizon-port-pack --lib` (154 passed), `cargo clippy -p horizon-port-pack --all-targets -- -D warnings`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
