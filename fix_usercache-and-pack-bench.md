Server files: reuse a player's user-cache slot
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

**User cache growth.** `UserCache::safe_add` runs on every join and pushed a new `Info` each time. Replaced entries were never dropped from `infos`, so a long uptime, or an offline-mode server flooded with bot joins, grew the cache without bound. Vanilla's maps replace the old object.

## The fix

An id already cached under the same name reuses its slot. A rename adds a new entry and the old name keeps pointing at the old one until it expires, as in vanilla's `safeAdd`. A new id still adds an entry.

## Tests

`the_same_player_joining_again_takes_no_new_entry`: 100 joins keep one entry; after a rename the old name still finds the old entry, the new name and the id find the new one, and rejoins under the new name add nothing.

The pack-bench workflow change moved to its own pull request (fix/pack-bench-summary-env), as asked in review.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

`GameProfileCache.safeAdd` puts into maps keyed by name and UUID, replacing the old object.

## Other projects' code

None.

## Self-check

`cargo test -p horizon-server-files` (24 passed), `cargo clippy -p horizon-server-files --all-targets -- -D warnings`, `cargo fmt --check`; `pack-bench.yml` parses.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
