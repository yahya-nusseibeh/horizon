Supervisor: apply queued restores and updates before every start
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

Three supervisor bugs from the same review:

1. **Queued restores and updates waited for an unrelated restart.** `POST /api/v1/restore` (and `horizon-server restore`, MCP restore) stores the request and sets `restart`. The request was only consumed on `Exit::Restart`. When the game was stopped, `wait_stopped` returned and the loop started the game on the current world. The restore stayed queued and replaced the world at the next restart, maybe days later, losing the progress since.
   - The CLI meanwhile printed "The game is running on the restored world".
   - The same happened when the restart flag arrived during a stop.
   - A queued update behaved the same, and `update.busy` stayed set, so every later update was refused.
2. **A huge update timeout panicked the supervisor.** `{"timeout": 18446744073709551615}` was accepted. `Instant + Duration` panics on overflow, inside `switch()` right after `current` was moved, so the supervisor exited mid-update with no rollback watch.
3. **A repeated API stop killed the game mid-save.** A second `server/stop` (management), `POST /api/v1/stop` or MCP stop while stopping `SIGKILL`ed the game during its save. Vanilla ignores a second halt.

## The fix

1. `apply_queued` runs at the head of every loop iteration, before the program is resolved. It does the queued restore and the queued release switch (returning its `Watch`). `Exit::Restart` now just continues.
2. `MAX_UPDATE_TIMEOUT` (one day): the API clamps to 1 s ..= 1 day, `switch` clamps too, and the CLI's `--timeout` refuses values above it.
3. The immediate kill on a second stop is kept only for a second signal (a person pressing Ctrl-C again). A repeated programmatic stop does nothing more, and `STOP_TIMEOUT` still bounds the wait.

## Tests

- `a_queued_restore_is_done_before_the_next_start`: a queued restore is applied by `apply_queued` and no longer queued, and nothing happens when nothing is queued.
- `an_update_timeout_past_a_day_is_refused`: the CLI's timeout parser.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

`MinecraftServer.halt` while already stopping does nothing more, which is the behaviour a repeated programmatic stop now has.

## Other projects' code

None.

## Self-check

`cargo test -p horizon-server` (177 passed), `cargo clippy -p horizon-server --all-targets -- -D warnings`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
