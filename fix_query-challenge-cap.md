Query: cap the challenges held between prunes
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

Every query handshake from a new address adds a challenge token, and the map is cleared only every 30 seconds (vanilla's `pruneChallenges`). A 7-byte UDP handshake is easy to send from spoofed addresses, so a flood grew the map with the packet rate: millions of entries per window.

## The fix

At 65,536 addresses holding a token, a handshake from a new address gets no answer until the next prune. An address that already holds a token still gets a fresh one. Replies to real clients don't change: one that gets no answer asks again, and the 30-second prune makes room.

## Tests

`handshakes_past_the_cap_wait_for_the_next_prune`: 65,536 addresses are answered and the next new one is not. An address already holding a token still gets a new one, and after the prune the new address is answered. `cargo test -p horizon-server query` passes.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

`QueryThreadGs4` keeps one challenge per handshake until `pruneChallenges`. The answers are unchanged below the cap.

## Other projects' code

None.

## Self-check

`cargo test -p horizon-server`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
