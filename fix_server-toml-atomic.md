Server: write server.toml atomically under one lock, and keep the world if a restore fails
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

**server.toml.** Several paths write `server.toml` with a read, a change and an in-place write, and nothing serialises them:

- management settings (one thread per connection);
- `PATCH /api/v1/config`;
- `horizon-server config set`;
- the prune mode reset;
- `sync_back`.

When two overlap, the later write drops the other's change, for example two management clients setting `motd` and `difficulty`. `std::fs::write` truncates first, so a crash in that window leaves an empty file. Every section is `#[serde(default)]`, so an empty `server.toml` reads as all defaults. The next start then writes those defaults into `server.properties`: whitelist, online mode, port and world name. A torn read at the supervisor's loop head also ends the supervisor.

**Restore.** A whole-world restore whose final rename fails (Windows with a scratch file held by antivirus or an indexer, or permissions) returned with the current world already moved to `backups/replaced-*`. The supervisor logged the error and started the game, which made a brand-new world. `restore_dimension` already put its folder back; `restore` didn't.

## The fix

- `config::replace` writes to a temporary file next to the target (named with the pid and a counter), syncs it, copies the old file's permissions and renames it over the old file. This is used for `server.toml` (`update_all`, `sync_back`) and `server.properties` (`write_properties`). The rename is what protects against the separate CLI process.
- A process-wide `WRITING` mutex is held across read, change and write in `update_all` and `sync_back`.
- `backup::restore` moves the old world back if the backup can't be put in place.

## Tests

`settings_changed_at_once_are_all_kept`: eight threads each set a different key at once. All eight are kept and no temporary file is left behind. Without the lock this test fails every time (3 of 3 runs). The restore rollback isn't covered: making only the second of two renames in one folder fail isn't portable in a test.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Not a vanilla feature (horizon-server's own settings file).

## Other projects' code

None.

## Self-check

`cargo test -p horizon-server` (176 passed), `cargo clippy -p horizon-server --all-targets -- -D warnings`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
