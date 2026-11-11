Mods: refuse a mod id that is a reserved namespace
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

`Manifest::parse` checks only that `id` is `[a-z0-9_]{1,64}`. `RESERVED_NAMESPACES` (`minecraft`, `c`, `horizon`) was checked against the extra `namespaces` list but never against the id. In `Content::check`, the id's namespace counts as the mod's own (`own = namespace == id`).

So an `.hmod` whose `mod.toml` says `id = "minecraft"` (from the registry, a link, a modpack or installed by hand) passed. Every `assets/minecraft/…` and `data/minecraft/…` file counted as its own. It could replace any vanilla recipe, loot table, worldgen file, lang file or texture, and declare `minecraft:stone`, breaking the fence's rule that vanilla ids can't be redefined. With `id = "c"` it could put any file under the shared convention namespace.

## The fix

- `Manifest::parse` refuses an id that is a reserved namespace ("id minecraft is a reserved namespace; pick another id").
- The fence also never treats a reserved namespace as a mod's own.
- No first-party `mod.toml` or `horizon-mod.toml` in the tree uses these ids (checked with grep).

## Tests

`refuses_bad_ids_and_apis` now also checks that `minecraft`, `c` and `horizon` are refused as ids.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Not a vanilla feature (Horizon content mods).

## Other projects' code

None.

## Self-check

`cargo test -p horizon-mods` (all pass), `cargo clippy -p horizon-mods --all-targets -- -D warnings`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
