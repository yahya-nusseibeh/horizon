Launcher: hardening for the server jar, MCP commands, cmd quoting and settings
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

Six Low findings in the launcher:

1. **Server jar.** The Mojang server jar was cached without checking it, and `!jar.exists()` meant a bad copy was never fetched again, for example a captive portal's page or a cut-off body. Every later Prepare then failed horizon-prepare's SHA-1 check until the player deleted the file.
2. **MCP `server_command`.** It passed a `\n` into the server's stdin console, so one call ran several commands while the tool call showed one.
3. **`mod_api_level`.** It read the whole game program while holding a global lock, and the window calls it on the UI thread.
4. **`complete_reports`.** Its temporary name depended only on the process id. The start-up thread and Play could copy the same report at once and one could write into the file the other had just renamed into place.
5. **`settings.json`.** It can hold proxy credentials but was written `0644`.
6. **Pre-launch command quoting.** On Windows, game words were quoted only for spaces, tabs and quotes. A `Rock&Roll` folder split the cmd.exe line, so the game didn't start and the rest ran as a command.

## The fix

1. The download is checked against `downloads.server.sha1`, and a cached jar that doesn't match is downloaded again.
2. A line break in `server_command` is an error.
3. The lookup reads the program without the lock and takes the lock only to look up and record.
4. Each copy gets a per-call counter in its temporary name.
5. The temporary file is created `0600` (`create_new`) before the rename.
6. `& | < > ^ ( )` also make a word quoted.

Moved to their own pull requests at review: the sign-in bounds (fix/launcher-sign-in-bounds) and the data-folder link check (fix/launcher-data-link-above-root).

Not done here: the Mods page's per-render file reads (a UI refactor of its own).

## Tests

- `windows_words_are_quoted_as_the_c_runtime_reads_them`, extended with `&`, `|`, `<`, `>`, `^` and parentheses.

All 667 launcher library tests pass (main merged).

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Not vanilla features (launcher).

## Other projects' code

None.

## Self-check

In `launcher/`: `cargo test --lib` (665 passed), `cargo clippy --lib --bins --tests -- -D warnings`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
