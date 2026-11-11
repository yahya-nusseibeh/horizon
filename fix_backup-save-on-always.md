horizon-server: a backup always turns saving back on
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

`Shared::back_up` (`api.rs`) sent `console save-off`, then `console save-all flush`, with `?` on each. If the flush failed, the `?` returned **before** `console save-on`, so the game kept running with autosave off. The control connection waits 30 s for a reply, and a flush on a large world or a slow disk can take longer. Scheduled backups take this path (`supervisor.rs`), so a server could silently stop autosaving until the next backup that succeeded, and a crash in that window lost everything played since.

## The fix

- Once save-off has been asked, saving is **always** turned back on:
  - after a good backup, over the same connection;
  - when save-off or the flush failed, over a **fresh** connection, because a reply that timed out leaves the old one out of step (a save-off whose reply was lost may still have applied).
- The flush gets its own reply timeout of 300 s (`FLUSH_TIMEOUT`); other requests keep 30 s (`engine::REPLY_TIMEOUT`, `Control::set_reply_timeout`).

## Tests

`a_backup_whose_flush_fails_turns_saving_back_on`: a fake game endpoint answers everything but drops the connection on the flush. The backup fails, and the game is asked `save-off`, `save-all flush` and then `save-on` on a new connection.

Found in a security review of Horizon (11 Nov, High).

## Kind

- [ ] feature
- [x] fix (no label)
- [ ] skip-release

## Vanilla reference

None: the supervisor's backups are Horizon's own (they drive vanilla's `save-off` / `save-all flush` / `save-on`).

## Other projects' code

None.

## Self-check

- `cargo test --locked -p horizon-server -- api:: engine::`: 25 passed
- `cargo clippy -p horizon-server --all-targets --no-deps -- -D warnings`: clean
- `cargo fmt --all -- --check`: clean

🤖 Generated with [Claude Code](https://claude.com/claude-code)
