Friends and relay: bounded handshakes, endpoints, STUN and UPnP input
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

Eight Low findings in Play with Friends (horizon-friends) and its relay:

1. **Host handshake slots.** The inbound handshake count was given back by hand. If the thread couldn't start, the slot was lost, and after `MAX_HANDSHAKES` (8) losses the direct port took nothing more.
2. **Race streams.** `session::settle` started one auth thread per direct-race stream with no cap and only a per-read timeout. Anyone who knew a live session id could open many streams and drip bytes, a thread each, on the host's game process.
3. **Invite endpoints.** `Invite::decode` took any number of endpoints, though `encode` writes at most 8. A crafted code pasted from chat made the guest open a thread and a connection per endpoint, to any address.
4. **STUN.** The read timeout restarted with every datagram, so a socket kept fed kept `stun::query` (and the host's setup that joins it) going forever.
5. **UPnP text.** A UPnP description's `controlURL`, `URLBase` or `serviceType` was only trimmed. A CR/LF inside went into the request line or the `SOAPAction` header.
6. **Guest accept.** The guest's direct-race listener read each inbound stream inline with a per-read timeout, so a stranger dripping bytes held the only accept thread past the race.
7. **Relay connection count.** It was given back by hand after `handle()`; a panic there would leak it.
8. **Attach tokens.** The relay compared attach tokens with an early-exit `==`.

## The fix

1. A `Handshake` guard owns the slot and is moved into the thread, so a failed spawn drops it too.
2. At most `MAX_RACE_STREAMS` (16) streams per session, each under one `crate::deadline` of 6 s.
3. More than `MAX_ENDPOINTS` endpoints is a bad code.
4. One deadline per attempt (`timeout / 3`), with the read timeout set to what is left.
5. A service whose text holds control characters or spaces is skipped.
6. Each accepted stream gets its own thread, with a 2 s `crate::deadline` for the whole handshake.
7. A `ConnSlot` guard (relay).
8. `same_token` compares every byte (relay).

## Tests

- `a_code_with_more_endpoints_than_encode_writes_is_refused`: 8 decode, 9 don't.
- `a_query_kept_fed_with_noise_still_ends`: a silent STUN server sending noise every 20 ms; the query ends within its deadline.

All horizon-friends and horizon-relay tests pass, including the tunnel tests over loopback.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Not a vanilla feature (Play with Friends).

## Other projects' code

None.

## Self-check

`cargo test -p horizon-friends -p horizon-relay` (all pass), `cargo clippy` for both with `--all-targets -- -D warnings`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
