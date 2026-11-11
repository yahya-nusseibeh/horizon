Server: on Windows, wait for the clean stop when the console closes
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

`signals.rs` promises that closing the console window is a clean stop request. But for `CTRL_CLOSE_EVENT`, `CTRL_LOGOFF_EVENT` and `CTRL_SHUTDOWN_EVENT`, Windows terminates the process as soon as the handler returns (documented `HandlerRoutine` behaviour). The supervisor died right after counting the request and never ran its stop.

The game is started with `CREATE_NEW_PROCESS_GROUP | CREATE_NO_WINDOW`, so it gets no event itself. It kept running with stdin at EOF, unsupervised, holding the world and the port until a logoff or shutdown killed it without a save. A new `horizon-server` start then found the lock free while the old game still had the world.

## The fix

- `signals::stopped()` sets a flag. `main` calls it when `supervisor::run` returns, which happens after the game has stopped.
- For those three events, the Windows handler waits for that flag before returning: up to 4 s for a close (Windows allows about 5) and 15 s for a logoff or shutdown.
- Ctrl-C and Ctrl-Break return at once as before.

## Tests

None automated: this is Windows-only and depends on how Windows ends a process after a console handler returns, which can't be run here. The change is small: the constants come from `windows-sys` 0.61 (`CTRL_CLOSE_EVENT` = 2, `CTRL_LOGOFF_EVENT` = 5, `CTRL_SHUTDOWN_EVENT` = 6), and the wait polls an atomic.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Not a vanilla feature (horizon-server supervisor).

## Other projects' code

None.

## Self-check

`cargo clippy -p horizon-server --all-targets -- -D warnings` and `cargo fmt --check` on Linux. Not compiled for Windows here, so please let CI's Windows job confirm.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
