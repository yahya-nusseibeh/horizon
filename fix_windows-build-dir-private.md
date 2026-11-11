Build native mods in a private ProgramData folder on Windows
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

When the launcher's data folder path is not ASCII (a Cyrillic or accented Windows user name), every native-mod build runs in `C:\ProgramData\horizon-launcher-<key>`, where `<key>` is 4 bytes of a SHA-256 of the data folder's path. This includes the Rust toolchain, its Cargo home and the temp folder. The code's own comment says every account may create folders in ProgramData, and its inherited ACL also lets every account add files below folders that others made. The name can be worked out from `C:\Users`.

So another local account could pre-create that folder with `rust\<version>-<target>\.installed` and its own `bin\cargo.exe` (`installed()` trusts the marker), or add `cargo\config.toml` with a `rustc-wrapper` or `linker` after the player's own install. The player's next native build would then run that account's program as the player.

## The fix

- `private_build_dir` names the folder `horizon-launcher-<128 random bits>` and creates it with `fs::create_dir`, so a folder that already exists, maybe made by someone else, is never used.
- It locks the folder to this account with `icacls /inheritance:r /grant:r <domain>\<user>:(OI)(CI)F` (icacls from `%SystemRoot%\System32`).
- It remembers the name in the data folder's `build-folder.txt`, which only this account can write. A remembered name that isn't of that exact shape (for example `..\..`) is ignored.
- If the folder can't be made or locked, it is removed. The fallback is the data folder's short (8.3) name, or the data folder itself, never a predictable public folder.
- Players affected get a new build folder once, so the toolchain downloads again on their next native build.

## Tests

`a_cyrillic_data_folder_builds_in_a_private_ascii_folder` (replaces the old "stable name" test) checks, with a stand-in lock:

- the folder is ASCII, inside ProgramData and locked when it is made;
- the same folder comes back next time without being made again;
- another data folder gets its own folder;
- a folder that can't be locked is neither used nor left behind;
- a remembered `..\..\Windows` is not followed;
- the `%ProgramData%` fallbacks still apply.

The `icacls` call itself is Windows-only and not run in this test.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Not a vanilla feature (launcher).

## Other projects' code

None.

## Self-check

`cargo test` and `cargo clippy --all-targets -- -D warnings` in `launcher/` on Linux, `cargo fmt --check`. I could not build for Windows here (no Windows target installed), so please let CI's Windows job confirm.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
