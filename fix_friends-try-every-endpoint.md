Friends: try every endpoint that answers before giving up on a direct join
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

A full invite carries the host's LAN addresses on the default port. `join_direct` dials them all, but only the first one that answered the hello got the handshake, and its error went out with `?`. So neither the invite's other endpoints nor its relay were tried.

On the guest's own network, another Horizon host can sit at the same private address and port, for example a housemate's game sharing a world. It answers first, and a valid code fails with "The code doesn't match".

## The fix

- Every stream that answers before the 5 s deadline gets the handshake in turn, until one takes the code.
- If none does and the invite has a relay (and the path allows it), the join falls back to the relay as before.
- Without a relay, the guest gets `WrongCode` if any endpoint said so (the most useful message), else the first other failure.

A stranger host that answers spends one of its own wrong-code guesses, as it already did.

## Tests

`another_host_answering_first_does_not_end_the_join` (horizon-relay `tests/tunnel.rs`): one endpoint is another real host, and the invite's own host is behind a 300 ms delay. The join now round-trips through the right host. With only the other host listed, the guest still gets `WrongCode`. The test fails without the fix.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Not a vanilla feature (Play with Friends).

## Other projects' code

None.

## Self-check

`cargo test -p horizon-friends -p horizon-relay` (all pass), `cargo clippy -p horizon-friends -p horizon-relay --all-targets -- -D warnings`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
