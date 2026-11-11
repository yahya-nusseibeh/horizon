Client: hold the servers.dat lock on every save, each process with its own temp file
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

`servers::lock()` exists so that a ping storing a server's icon and a save of the list don't undo each other's change. Three writers loaded and saved `servers.dat` without it:

- adding or editing an entry on the multiplayer screen;
- `ServerList.saveSingleServer` for a server's resource-pack choice (`game/server_packs.rs`);
- the same for its Code of Conduct (`game/code_of_conduct.rs`).

A favicon saved by the ping thread at the same moment could drop the user's edit, or the edit could drop the icon. All writers also used one fixed temporary file, `servers.dat_tmp`. Overlapping writes (another game window, or the unlocked paths) could rename an interleaved file into place. That file loads as an empty list, and the next save writes the empty list, losing every server.

## The fix

- All three paths now hold `servers::lock()` across load and save. The screen drops the guard before it reloads the list, since loading may save under the same lock and std's `Mutex` isn't reentrant.
- The temporary file is `servers.dat_tmp.<pid>`.

## Tests

None new: the race needs two threads writing at once. The existing `servers` tests pass, and the gameplay library type-checks.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

`ServerList.save` and `saveSingleServer` (vanilla runs them on the render thread, so its writes don't overlap).

## Other projects' code

None.

## Self-check

`cargo test -p horizon-protocol --lib -- servers`, `cargo check` and `cargo clippy` for horizon-client (no default features) and horizon-protocol with no new warnings, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
