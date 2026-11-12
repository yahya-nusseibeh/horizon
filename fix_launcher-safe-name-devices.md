Launcher: no Windows device names or disguising marks in downloaded file names
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

`browser::safe_name` checks the file name a Modrinth, CurseForge or registry download is saved under, and `browser/download.rs` joins it to the mods or packs folder. It let through two kinds of names:
- **Windows device names.** `CON`, `NUL.jar`, `COM1.zip` and so on: on Windows the joined path names a device, so the rename from `.<name>.part` fails or acts on that device.
- **Bidirectional and invisible marks.** For example U+202E, which `is_control` doesn't catch, so a name shown in the launcher could read as another.

## The fix

`safe_name` also refuses:
- a name whose part before the first dot (trailing spaces trimmed, any case) is `CON`, `PRN`, `AUX`, `NUL`, or `COM`/`LPT` followed by one digit or ¹²³.
- zero-width and direction marks: U+200B–U+200F, U+202A–U+202E, U+2066–U+2069 and U+FEFF.

## Tests

`file_names_must_be_plain` gains `CON`, `nul.jar`, `Com1.zip`, `LPT9`, `AUX .zip`, `COM¹.jar`, a name with U+202E and one with U+200B as refused. `Console.zip`, `COM10.jar`, `nullify.jar` and `Ünïcode pack.zip` are still accepted.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Other projects' code

None.

## Self-check

`cargo test --lib` and `cargo clippy --lib --tests -- -D warnings` in `launcher/`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
