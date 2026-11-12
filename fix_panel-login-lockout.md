Panel: wrong login codes no longer lock out the right one
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

`Panel::redeem` counted failed logins for everyone and checked the count *before* looking at the code. So ten bad `/login` requests a minute from anyone who could reach the panel made every valid link fail with 429, the owner's included. The codes are 256 random bits, so the limit protects nothing a valid code needs protecting from.

## The fix

The limit is checked only when the code is wrong. A valid link always signs in. Wrong codes are still counted and refused with 429 past ten a minute.

## Tests

`sign_in_with_a_link` goes on after the 429: a fresh link still signs in (303), and the next wrong code is still refused (429). `cargo test -p horizon-server panel` passes.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Other projects' code

None.

## Self-check

`cargo test -p horizon-server`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
