Client: private Realms temp files, a bounded pax record, and no leaked boot-screen font
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

Three Low findings on the client side:

1. **Realms temp files.** Realms downloads (`backup-<pid>-<n>.tar.gz`, `resources-<pid>-<n>.tar.gz`) and uploads used predictable names in the shared temp folder and opened them with `File::create`. On a multi-user Linux machine, another local account could create the file or a symlink first. The player's download then went into that file, or truncated whatever the link pointed at, and the resource pack could be swapped between its SHA-1 check and its copy.
2. **pax records.** `pax_path_of` checked only `length != 0 && length <= rest.len()`. A record like `1 x` sliced `rest[2..1]` and panicked the "Realms" worker, so the download screen never finished.
3. **Boot-screen font leak.** The Windows boot-screen painter restored its three fonts in turn. But `old_tagline` is `mark` and `old_console` is `tagline`, so `tagline` stayed selected, `DeleteObject` refused it, and one HFONT leaked per repaint (about three a second while the renderer boots).

## The fix

1. `download::unguessable` adds a per-process random key (`RandomState`) to the name, and `download::create_private` opens with `create_new` (never a file that was already there) and mode `0600`. Downloads and the upload archive use both. With the file owned by us in a sticky temp folder, no one else can replace it between the hash and the copy.
2. The length must reach past its own digits and space (`length > space + 1`), as xtask's reader already checks.
3. Restore only the DC's original font, which unselects all three before they are deleted.

## Tests

- `a_pax_record_shorter_than_its_own_prefix_is_not_a_panic` (and a good record still reads).
- `a_download_never_writes_into_a_file_that_was_there`: names differ, a planted file is refused, and a new file is owner-only.

The boot-screen change is Windows-only GDI code and is not covered by a test here.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

`FileDownload` writes to a temp file too; vanilla uses `File.createTempFile`, whose names are random.

## Other projects' code

None.

## Self-check

`cargo test -p horizon-client-lib --no-default-features --lib -- realms::download` (pass), `cargo clippy -p horizon-threads -p horizon-client-lib --no-default-features --all-targets -- -D warnings` (no new warnings), `cargo fmt --check`. The boot-paint change was not compiled for Windows here.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
