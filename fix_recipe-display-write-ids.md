Write recipe displays in an older client's own ids
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

When an older client (1.21.3 to 1.21.11) joins a Horizon server, the server's reverse adapter re-encodes `recipe_book_add` and `place_ghost_recipe` for it under `Way::Down`. But `SlotDisplay::write` always wrote 26.3's display ids (0–10), and `HolderSet::write` and `OnlyWithComponent` wrote the pinned item and component ids. Those clients number displays differently (`SLOT_DISPLAY_DOWN` = `(0,0,2),(4,2,3),(8,5,3)`) and lack `with_any_potion`, `only_with_component` and `dyed`. A 1.21.11 client read an item display (4) as a tag and a composite (10) as out of range, so its decoder failed and the player was disconnected as soon as the server sent a recipe book with entries, normally on join. The read paths already map these ids; the write paths did not.

## The fix

- `SlotDisplay::write` writes every kind through `ids::slot_display`.
- Kinds the client lacks are written as what they wrap: `with_any_potion` → its display, `only_with_component` → its display (also when the component is unknown to the client), `dyed` → the dyed item.
- A pre-1.21.5 client (whose trim display named a template, not a pattern) is shown the item being trimmed.
- An item display whose item the client lacks becomes `empty`.
- `HolderSet::write` maps item ids with `ids::item` and drops the ones that map to -1 before writing the count.
- With no renumbering active, all of these are the identity, so the pinned path writes the same bytes as before.

## Tests

`displays_written_for_a_1_21_11_client_read_back_as_that_version`: a composite with an item, a with-any-potion, a dyed and a with-remainder display, plus an item holder set, is written under 1.21.11's down ids and read back under its up ids. The result has the expected degraded displays with the same items and no bytes left over. Before the fix, the first item display was read as a tag.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

The 1.21.11 `SlotDisplays` registry has no `with_any_potion`, `only_with_component` or `dyed` entries; the adapter's `SLOT_DISPLAY_DOWN` tables encode this.

## Other projects' code

None.

## Self-check

`cargo test -p horizon-protocol` (450 passed), `cargo clippy -p horizon-protocol --all-targets -- -D warnings`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
