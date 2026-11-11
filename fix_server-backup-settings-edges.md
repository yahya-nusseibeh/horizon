Server: backup and settings edge cases
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

Six Low findings in horizon-server:

1. **Backup permissions.** Backup zips were created `0644`, though each holds `server.toml` (API token, Discord webhook, `[properties]`) and `server.properties` (RCON password, management secret). The supervisor already writes `.horizon/api.json` owner-only for that reason.
2. **Pruning.** It deleted any backup ending in `-auto.zip`. So a backup labelled by hand `nightly-auto` (or `pre-auto` through the API) was deleted, though manual backups are documented as never deleted.
3. **`backups.every`.** `split_at(len - 1)` panicked on `"6é"`, reachable from `PATCH /api/v1/config` and a hand-edited `server.toml`. `n * secs` overflowed for a huge `n`.
4. **Discord.** A rate-limit header of `NaN` parsed, stayed NaN through `clamp`, and `Duration::from_secs_f64` panicked, ending the Discord thread.
5. **`world.name`.** `"."` was accepted, also remotely, making the world the server folder itself. Backups then zipped `backups/` and their own `.part` file, and a restore tried to move the server folder.
6. **init.** `init` ignored a `whitelist.json` / `ops.json` it couldn't read, then `add` saved at once, replacing the existing list with only the players named to init.

## The fix

1. Backups are opened with mode `0600` on Unix.
2. Prune matches only names of exactly `<stamp>-auto.zip`.
3. The unit is the last character by `char_indices`, and the product uses `checked_mul`.
4. Non-finite values fall back to the 2 s default.
5. Refuse `.`, `..`, `backups`, `logs` and `.horizon` as world names.
6. `init` stops with "fix it or move it away, then run init again". `server.toml` is written after this step, so init can be run again.

## Tests

- `prune_keeps_backups_labelled_by_hand_whatever_their_label`.
- `a_backup_is_readable_by_its_owner_only` (Unix).
- `a_nan_rate_limit_waits_the_default`.
- `a_whitelist_init_cannot_read_is_left_alone`.
- Extended: `interval` (`6é`, `é`, `u64::MAX d`) and the world-name validation test.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Not vanilla features (horizon-server's backups and settings).

## Other projects' code

None.

## Self-check

`cargo test -p horizon-server` (179 passed), `cargo clippy -p horizon-server --all-targets -- -D warnings`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
