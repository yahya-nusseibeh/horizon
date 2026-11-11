Port-pack: only a plain NeoForge version makes the loader jar's path
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

When the launcher imports and ports a modpack (`.mrpack` from the mod browser, or CurseForge / a mod list), the pack's NeoForge version (`modrinth.index.json` `dependencies.neoforge`, any string) went unchecked into `loader_jar`'s file name and URL.

A version like `21.1.1/neoforge-21.1.1-universal.jar#/../../../../<dir>/x`:

- made `fetch` create every directory on that path (`create_dir_all` walks the `..` components);
- downloaded a real NeoForge jar (the `#` ends the URL);
- renamed it over `<dir>/x-universal.jar`, overwriting anything there.

The `.sha1` lookup got the binary jar and failed, so `sha1` was `None` and nothing was checked. The bytes were limited to what maven.neoforged.net serves, but the place, the directories and the overwrite were the pack author's choice.

## The fix

- `loader_jar` accepts only a plain version: 1–64 of `[A-Za-z0-9._+-]`, without `..`. Anything else gets no loader jar, and the port goes on with vanilla's biome tags as it already does when the jar is missing.
- `fetch` downloads the loader jar only with a 40-hex SHA-1 read from Maven's `.sha1`. Without one it logs `…: no SHA-1` and skips the jar.

## Tests

`only_a_plain_neoforge_version_makes_a_loader_path`: real versions (`21.1.77`, `21.1.0-beta`) give the expected path and URL. The traversal string from the review, `../x`, `#`, slashes, backslashes and the empty string give none, and so does a non-NeoForge loader.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Not a vanilla feature (modpack porting).

## Other projects' code

None.

## Self-check

`cargo test -p horizon-port-pack --lib` (all pass), `cargo clippy -p horizon-port-pack --all-targets -- -D warnings`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
