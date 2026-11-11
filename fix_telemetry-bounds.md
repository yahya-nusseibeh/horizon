Telemetry: refuse an impossible official player count
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

`record_official` stored the official server's `players.online` unchecked. That value is any `u64` from a plain, unauthenticated status ping. `history.rs` casts it to `i64`:

- above `i64::MAX` it wraps negative;
- near it, adding a second minute of the same bucket makes SQLite's `SUM()` fail with "integer overflow". `GET /v1/live/history?source=official` (24h, 7d, 30d) then fails until those rows are pruned, which takes 7 or 90 days.

Triggering it needs control of that server's answer or of the network path to it.

## The fix

A count above `MAX_OFFICIAL_PLAYERS` (1,000,000; no real server has more) is treated like no answer: nothing is written for that minute.

Not done here: the review's two other telemetry Lows (hub votes per address, and the benchmark summary's install count). Both need a schema change, a migration and a choice about players sharing one address, which is better left to the maintainer.

## Tests

`an_impossible_official_count_is_not_written` (tests/live.rs): a stand-in server answering `18446744073709551615` isn't recorded, and the official history still answers 200.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Not a vanilla feature (telemetry server).

## Other projects' code

None.

## Self-check

In `crates/horizon-telemetry-server`: `cargo test` (all pass), `cargo clippy --all-targets -- -D warnings`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
