Server files: reuse a player's user-cache slot; pack-bench takes its input from the environment
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

Two small Low findings:

1. **User cache growth.** `UserCache::safe_add` runs on every join and pushed a new `Info` each time. Replaced entries were never dropped from `infos`, so a long uptime, or an offline-mode server flooded with bot joins, grew the cache without bound. Vanilla's maps replace the old object.
2. **pack-bench input.** The workflow's Summary step pasted `${{ inputs.base || 'main' }}` into its `run:` script, so a crafted dispatch input ran commands on the self-hosted runner. Only users with write access can dispatch it, so this is hardening. The step before already reads the same value through `env`.

## The fix

1. An id already cached under the same name reuses its slot. A rename adds a new entry and the old name keeps pointing at the old one until it expires, as in vanilla's `safeAdd`. A new id still adds an entry.
2. The Summary step reads `BASE` from `env:`.

## Tests

`the_same_player_joining_again_takes_no_new_entry`: 100 joins keep one entry; after a rename the old name still finds the old entry, the new name and the id find the new one, and rejoins under the new name add nothing. The workflow still parses as YAML.

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
