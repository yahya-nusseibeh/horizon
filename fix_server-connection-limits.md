Server: per-address RCON places, write timeouts for event streams, digest token compare
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

Four Low findings in horizon-server's network endpoints:

1. **RCON.** Sixteen idle connections from one address took every RCON place (`MAX_CLIENTS`) for 30 s each, renewable forever. The operator's own RCON client was then refused before it could send a password.
2. **Management notifications.** One thread delivered every notification to every management client with blocking writes and no write timeout. A client that stopped reading filled its TCP buffer, the write blocked, notifications stopped for everyone, and the channel feeding the thread grew.
3. **Event stream.** `GET /api/v1/events` (SSE) wrote with no write timeout, so a client that stopped reading pinned a thread forever.
4. **Token compare.** `same_token` returned early on a length difference, leaking a hand-set token's length through timing.

## The fix

1. `rcon::Place` now also takes one of its address's `PER_ADDRESS` (4) places. Both counts are given back when it drops. The global cap is unchanged.
2. Management connections get a 10 s write timeout. When a notification write fails, that socket is closed (1008), so its reader ends and unregisters it.
3. The SSE stream sets a write timeout of twice its keep-alive (30 s).
4. `same_token` compares SHA-256 digests of both sides (`sha2` is already a dependency).

## Tests

- `one_address_takes_only_its_share_of_places`: one address gets 4, another address still gets in, everything is given back on drop, and the global cap still holds across addresses.
- `tokens_compare_whole`.

The existing RCON, management and events tests pass (177 in all).

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Vanilla's RCON has no connection limits at all; Horizon's are its own hardening.

## Other projects' code

None.

## Self-check

`cargo test -p horizon-server` (177 passed), `cargo clippy -p horizon-server --all-targets -- -D warnings`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
