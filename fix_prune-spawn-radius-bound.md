Prune: a spawn radius past i32 keeps everything instead of nothing
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

`prune_chunks.spawn_radius` is a `u32` that `run` cast to `i32`. A value above `i32::MAX` wrapped negative, so no chunk counted as near spawn and a delete run pruned the spawn area like any other. The section can be changed over the control API.

## The fix

The radius saturates at `i32::MAX` (`i32::try_from(..).unwrap_or(i32::MAX)`): a radius as large as asked for, which keeps every chunk. Nothing a valid setting does changes.

## Tests

`claims_homes_and_the_spawn_radius_keep_chunks` adds a dry run with `spawn_radius = u32::MAX`, which must keep every chunk (" 0 of "). `cargo test -p horizon-server prune` passes.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Other projects' code

None.

## Self-check

`cargo test -p horizon-server`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
