Launcher: save accounts.json atomically and merge what a launch renewed
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

When the player presses Play (and the same for `play --account`, Java instances and the game download), the launcher:

1. loads `accounts.json`;
2. asks the services about each account (several requests, 20 s timeout each), taking the account lock per account only;
3. saves its whole copy without the lock.

A sign-in finished meanwhile (the window's sign-in thread or the CLI) was dropped from `accounts.json`, with its secrets left orphaned in the keychain. A sign-out made meanwhile came back as "no longer signed in".

`save_to` also truncated and rewrote `accounts.json` in place. A crash or full disk mid-write left a file that `load_from` reads as empty (`unwrap_or_default`), so every account vanished until each was signed in again.

## The fix

- `save_to` writes `accounts.json.<pid>.tmp` and renames it over the file, removing the temporary file on failure.
- New `Accounts::save_renewed`: under `session::lock`, it loads the file as it is now, applies the launch's renewed profiles with `update` (only accounts still there), keeps the current selection (or takes the launch's if none is valid), and saves.
- `session_for_game`, `session_for_game_as`, `java_player` and the game download use `save_renewed` instead of `save`.

## Tests

`a_launch_saving_late_keeps_sign_ins_and_outs_made_meanwhile`: a launch's copy with a renewed skin is saved after another account signed out and a new one signed in. The result has the new account, not the removed one, plus the renewed skin and the selection, with no temporary file left.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Not a vanilla feature (launcher accounts).

## Other projects' code

None.

## Self-check

In `launcher/`: `cargo test --lib -- account` (68 passed), `cargo clippy --lib --bins --tests -- -D warnings`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
