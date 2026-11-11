Bound legacy smithing-trim previews from old servers
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

When Horizon joins an older server (1.20.2 to 1.21.4), the protocol adapter turns that server's smithing-trim display into one `SmithingTrim` per trim pattern, and each one gets its own copy of the base and material displays. Those copies can themselves be trim displays, expanded the same way, so the size multiplies at every level. A packet of a few hundred bytes could ask the client for billions of nodes and kill it with out-of-memory. The 1.21.1 recipe path had the same problem with a large base or addition ingredient (one 4-byte stack became many 80-byte nodes per pattern).

## The fix

- `SlotDisplay::weight()` counts the displays and listed items in a tree (saturating), so it shows what a copy costs.
- `MAX_TRIM_PREVIEW = 4096`: a legacy trim preview whose `patterns × (base + material weight)` is above this is refused with `legacy trim preview too large`. The check runs in `SlotDisplay::read` (1.21.2–1.21.4 servers) and in the 1.21.1 trim recipe translation (1.20.2–1.21.1 servers). The 1.21.1 path also builds each ingredient display once and clones it, instead of rebuilding it per pattern.
- Vanilla's trim recipes stay far below the cap (about 18 patterns × a few dozen items).

## Tests

- `a_legacy_trim_display_that_would_balloon_is_refused` (packets/recipes.rs): a 1.21.4 trim display nested 4 levels deep still reads; one nested 24 levels deep is refused.
- `a_trim_recipe_whose_previews_would_balloon_is_refused` (v1_21_1/recipes.rs): a trim recipe with 40 base stacks translates; one with 5000 is refused.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Vanilla's own old clients never expand these displays: this is only about the adapter's translation.

## Other projects' code

None.

## Self-check

`cargo test -p horizon-protocol` (451 passed), `cargo clippy -p horizon-protocol --all-targets -- -D warnings`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
