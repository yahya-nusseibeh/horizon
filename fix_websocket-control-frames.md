WebSocket: control frames must come whole and short
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

RFC 6455 section 5.5 has control frames (close, ping, pong) unfragmented, with payloads of at most 125 bytes, and Netty's frame decoder, which vanilla's management server uses, fails the connection otherwise. `Socket::read_text` accepted a 64 KiB ping and echoed it (a 64 KiB allocation and write each), and answered a ping without FIN.

## The fix

After the length is read, a control frame (opcode 8 or more) that isn't final or is longer than 125 bytes ends the connection with "a control frame must be whole and at most 125 bytes". Data frames are unchanged.

## Tests

`control_frames_must_be_whole_and_short`: a 125-byte ping is answered and the text after it arrives. A 126-byte ping, a ping without FIN and a close without FIN are refused. The existing tests (pings, fragmented text, close, oversized and unmasked frames) still pass.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Other projects' code

None.

## Self-check

`cargo test -p horizon-server websocket`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
