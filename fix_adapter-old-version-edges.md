Protocol adapter: edge cases with older clients and servers
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

Low findings in the protocol adapter, each small:

1. **26.2 explosions.** `client_session/world.rs` dropped the whole explode packet for a 26.2 client when the blast's own particle had no 26.2 id (a native mod's). That packet also carries the knockback and sound, so the 26.2 player wasn't pushed while others were, then rubber-banded.
2. **1.21.1 day time.** `step_1_21_1::set_time` sent `-0`, which is `0` and reads as "running", for a clock stopped at day time 0. Vanilla 1.21.1 sends `-1`.
3. **Teleport queue.** An older client that never accepts teleports got a new one every 20 ticks, and each was queued for the life of the connection.
4. **26.1 entity lock.** Every serverbound packet of a 26.1 client took the tracked-entity lock, though only spectate looks entities up (by a linear scan).
5. **Old explosions.** An old server's explosion could name about 1M blocks. Each became its own queued `BlockUpdate`, a roughly 25× amplification per packet.
6. **1.21.1 window ids.** A 1.21.1 `container_set_slot` for a negative window other than -1/-2 was a decode error that disconnected the player. The old client ignored it.
7. **Mod lists.** A joining Horizon client's mod list could hold line breaks. The mismatch reason went into the dedicated server's log via `lost()`, forging whole log lines, including the supervisor's "Stopping server".

## The fix

1. Fall back to vanilla's `explosion` particle before dropping.
2. `-1` when stopped at 0.
3. Keep the 64 most recent pending teleports.
4. Lock lazily inside `entity_of`.
5. `MAX_EXPLOSION_BLOCKS = 65_536` for the 1.20.2, 1.20.3 and 1.21.1 readers.
6. Drop such packets (`Ok(None)`).
7. `mod_list::decode` refuses control characters in ids and versions, and `lan/conn.rs` `lost()` replaces any control character with a space.

Not done here: the review's Low on RSA implicit rejection. Its suggested fix assumes a 16-byte plaintext, but the same decrypt also reads the login challenge, so it needs a design of its own.

## Tests

- `a_clock_stopped_at_zero_is_sent_as_minus_one`.
- `a_slot_in_an_unknown_negative_window_is_dropped_not_fatal`.
- `a_line_break_in_a_mod_list_is_refused` (non-ASCII versions still pass).

The other changes are one-line bounds, covered by the existing adapter tests (452 pass).

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

- 1.21.1 `ClientboundSetTimePacket`: a stopped clock negates the time, with 0 sent as -1.
- 1.21.1 `ClientPacketListener.handleContainerSetSlot` ignores other windows.

## Other projects' code

None.

## Self-check

`cargo test -p horizon-protocol --lib` (452 passed), `cargo clippy -p horizon-protocol -p horizon-client-lib --no-default-features --all-targets -- -D warnings` (no new warnings), `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
