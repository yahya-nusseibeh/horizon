Friends: bound router answers, and keep a bad one from crashing or freezing the game
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

Two findings in Play with Friends' router mapping (UPnP), where the "router" is whoever answers SSDP on the LAN:

1. **A crafted chunk could crash the game.** `dechunk` sliced `&rest[..n]` with the device's chunk size, on a `from_utf8_lossy` string. A chunk ending inside a multi-byte character panicked. On the 30-minute renewal path that happened while `share()` held `shared.reach`, poisoning it. `Host::stop()`'s `reach.lock().unwrap()` then panicked on the game thread (and in `Drop`).
2. **Stopping could freeze the game.** `http()` limited each read to 5 s, not the answer. A device sending a byte every few seconds kept a request going for up to 1 MiB, which is weeks. The renewal made that request under `shared.reach`, which `Host::stop()` takes on the game thread after joining the sharing thread. SSDP discovery also accepted any number of gateway locations, each tried in turn.

## The fix

1. `dechunk` uses `get(..n)` / `get(n..)` and stops at a chunk it can't take. `stop()` and the renewal loop take the lock with `unwrap_or_else(|e| e.into_inner())`.
2. `http` reads the whole answer under one deadline (`HTTP_DEADLINE`, 10 s), still capped at 1 MiB. Discovery keeps at most `MAX_GATEWAYS` (8). The mapping is taken out of `reach` (`take_if(due)`), renewed without the lock and put back. `stop()` joins the sharing thread before it releases, so the mapping is always back by then.

Not changed: `release()` still runs on the game thread at stop, bounded by the same deadline.

## Tests

- `a_chunk_ending_inside_a_character_stops_the_body`: split characters and short chunks stop the body cleanly, and whole multi-byte chunks still decode.
- `a_router_trickling_its_answer_runs_out_of_time`: a local server sending one byte every 20 ms is cut off at a 300 ms deadline with `TimedOut`.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Not a vanilla feature (Play with Friends).

## Other projects' code

None.

## Self-check

`cargo test -p horizon-friends` and `cargo test -p horizon-relay --test tunnel` (all pass), `cargo clippy -p horizon-friends --all-targets -- -D warnings`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
