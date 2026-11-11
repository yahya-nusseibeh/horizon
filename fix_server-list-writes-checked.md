Server: list edits report a failed save, server.properties is written atomically, plain params
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

Three Low findings in the dedicated server's own files and messages:

1. **List edits claimed success.** `UserList::add` / `remove` save the list but only log a failed save, then return true. With a read-only or full disk, `horizon-server op/ban/whitelist…` and the stopped server's API answered "Banned X" or "Made X a server operator", but the file didn't change and the player got in at the next start.
2. **`server.properties` writes.** `ServerProperties::store`, called on every start, wrote `server.properties` in place with the default mode. A crash or full disk mid-write left a short file, which the next start reads as defaults: a new management secret, and every non-default setting dropped. The file holds `rcon.password` and `management-server-secret`, and was readable by other local accounts.
3. **Message parameters.** `system_message` and `players/kick` copied `translatableParams` (any JSON) into a tellraw component. A management client could pass `{"selector":"@e"}`, `{"score":…}` or `{"nbt":…}`, which tellraw resolves with the console's rights. Vanilla's `Message` takes these parameters as strings.

## The fix

1. `lists.rs` calls a new `saved(&list)?` after each add or remove that changed something (op, deop, whitelist add/remove, ban, pardon, ban-ip, pardon-ip). A failed write is now an error with the file's path.
2. `store` writes `server.properties.<pid>.tmp`, syncs it and renames it over. A new file is `0600`; an existing file keeps its permissions, so an operator who loosened them for a reason isn't overridden.
3. Each parameter becomes `{"text": "<the string>"}`, and non-strings become empty text.

## Tests

- `translatable_parameters_are_plain_text`.
- `a_new_properties_file_is_its_owners_and_an_old_one_keeps_its_mode` (Unix; also checks no temporary file is left).

The failed-save path of the lists isn't tested: the test runner is root, which writes through read-only modes.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

The `Message` DTO of the server management protocol (`translatableParams`: a list of strings).

## Other projects' code

None.

## Self-check

`cargo test -p horizon-server -p horizon-server-files` (all pass), `cargo clippy` for both with `--all-targets -- -D warnings`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
