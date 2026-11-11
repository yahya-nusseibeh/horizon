Server: a --system unit runs as the account that ran sudo, never root
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

The help says to run `horizon-server service install --system` with sudo. The unit's `User=` came from `$USER`, and sudo's `env_reset` sets `USER` to the target user. So the unit always got `User=root`. That had two effects:

- The network-facing dedicated server (game port, RCON, management API, web panel) ran as root, so any remote bug in it ran with root's rights.
- It created world, log and backup files owned by root in the operator's folder, so a later `horizon-server run` as that operator failed with permission errors.

## The fix

`system_user` picks `SUDO_USER` (the account that ran sudo), else `USER`, skipping empty and `root`. When neither names an account, as from a root shell, it refuses with a message saying to run it with sudo from that account. `docs/hosting.md` says the unit runs as your own account.

## Tests

`a_system_unit_runs_as_the_account_that_ran_sudo`: sudo's environment (`SUDO_USER=ana`, `USER=root`) gives `ana`, plain `USER` gives that account, and root-only environments are refused.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Not a vanilla feature.

## Other projects' code

None.

## Self-check

`cargo test -p horizon-server` (all pass), `cargo clippy -p horizon-server --all-targets -- -D warnings`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
