Server: players can't forge the game's log lines or flood the log
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

Two findings about the dedicated server's stdout, which the supervisor reads:

1. **Chat could fake "Stopping server".** The supervisor matched `"/INFO]: Stopping server"`, `"/INFO]: Stopping in "` and `"/INFO]: Done ("` anywhere in a line. A player typing `/INFO]: Stopping server` in chat made management clients (and a transfer proxy) get `notification/server/stopping`. Since `management::stopping` latches, the real notice at the real stop was then never sent.
2. **Quiet commands could write whole log lines, and flood the log.** The `horizon:quiet_commands` payload's mod id was only length-capped. The batch summary wrote it unescaped to stdout and the log, so `"x\n[12:00:00] [Server thread/INFO]: Stopping server"` forged a complete line. Every decoded batch was also queued and logged, even an empty one (3 bytes) and before any permission check. At the hub's 400 packets a tick, one player could write thousands of lines a second. The command lines also skipped the illegal-character check a chat command gets.

## The fix

- The supervisor takes the message after the first `/INFO]: ` (`info_message`) and checks that it starts with `Done (`, `Stopping server` or `Stopping in `. Chat comes after `<name> `, so it can't match.
- `quiet::decode` refuses control characters, DEL and `§` in the mod id and every line, as `StringUtil.isAllowedChatCharacter` does.
- `guest_quiet_commands`:
  - ignores an empty batch;
  - with no room left, adds the dropped count to the player's last waiting batch instead of queuing a new one, so it isn't logged again;
  - logs a refused payload once per connection (`LanHost::quiet_refused`).

## Tests

- `chat_cannot_pass_for_the_games_own_stopping_line` (supervisor): real stopping and done lines match, chat containing them doesn't.
- `control_characters_are_refused_in_the_mod_id_and_lines` (quiet.rs): a newline in the id, a newline, `§` or DEL in a line are refused, and accented text passes.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

`ServerGamePacketListenerImpl.handleChatCommand` disconnects a player whose command holds a character `StringUtil.isAllowedChatCharacter` refuses.

## Other projects' code

None.

## Self-check

`cargo test -p horizon-protocol --lib -- quiet` and `cargo test -p horizon-server` pass. `cargo check` and `cargo clippy` pass for horizon-client (no default features), horizon-protocol and horizon-server; the only clippy warnings left are older ones in other files. `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
