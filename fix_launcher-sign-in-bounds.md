Launcher: bound the sign-in services' interval and lifetimes
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

The Microsoft device-code sign-in used the services' numbers unchecked: the poll `interval` (and `slow_down`'s +5), the device code's `expires_in` and the Minecraft token's lifetime. `Instant + Duration` with a huge interval panics, so a bad or hostile reply killed the sign-in thread and left the window showing the code forever.

Split out of #15247 at review.

## The fix

`account.rs`: `interval` is clamped to 1–60 s, slow-down included; the device code's expiry to an hour; the Minecraft token's lifetime to a week. Each is added with `saturating_add`. Real values (5 s, 900 s, 86,400 s) are unchanged.

## Tests

No new test: the change is three clamps on values from the network, and the code paths need the live services. All 664 launcher library tests pass.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Other projects' code

None.

## Self-check

`cargo test --lib` and `cargo clippy --lib --tests -- -D warnings` in `launcher/`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
