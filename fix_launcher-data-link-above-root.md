Launcher: follow links above the data folder when moving it
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

Moving the launcher's data folder was refused when any folder *above* a data root was a symbolic link, for example `~/.local/share` linked to another disk, or Fedora Atomic's `/home` (a link to `/var/home`). Only the roots themselves are copied, so only they need to be real folders.

Split out of #15247 at review.

## The fix

`checked_path` (`data_location.rs`) applies `reject_link` only to the last component, the root itself. The folders above it are still canonicalized, so the path that is used is the real one.

## Tests

`a_link_above_the_root_is_followed` (Unix): a root reached through a linked parent is accepted and resolves to the real folder; a root that is itself a link is still refused. All 665 launcher library tests pass.

Windows: not yet run with a junction above the target; the reviewer asked for that check and it will be posted here.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Other projects' code

None.

## Self-check

`cargo test --lib` in `launcher/`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
