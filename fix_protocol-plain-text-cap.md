Protocol text: a disconnect reason can't expand past 1 MiB
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

`horizon_protocol::text::plain` flattens a component into one `String`. When a translation key is unknown, it uses the server's own `fallback` as the template, and `format_java` copies an argument **once per placeholder**. The client calls it on the play and configuration `Disconnect` packets (`net.rs`, `Reason::tag`), and the login-phase JSON reason goes through `json_text`, a second copy of the same expansion with no depth limit at all.

A fallback of 16 000 `%1$s` (64 KB, one NBT string) whose argument is the same construct, nested three levels deep (well inside the depth-32 guard), asks for about 16000³ ≈ 4·10¹² bytes. The allocation fails and Rust aborts the whole process, so a malicious server, or anyone who can inject a kick reason, crashes the client instead of disconnecting it.

## The fix

- New `text::MAX_TEXT` (1 MiB) and `push_capped`, which appends up to the cap and cuts at a character boundary.
- `plain`/`append` stop once the output reaches the cap, and every append in them is capped.
- `format_java` stops at the cap, so each nested argument is at most 1 MiB.
- `json_text` (`net.rs`) caps strings, arrays and `extra` the same way.

A disconnect screen needs a few hundred bytes, so nothing real is cut.

## Tests

- `a_fallback_repeating_its_argument_stays_bounded` (`text.rs`): the 16 000 × 3-level payload fills up to the cap and no further, within seconds; a cut never splits a character; an ordinary fallback still formats.
- `a_login_reason_repeating_its_argument_stays_bounded` (`net.rs`): the same payload in login JSON stays under the cap.

Found in a security review of Horizon (11 Nov, High).

## Kind

- [ ] feature
- [x] fix (no label)
- [ ] skip-release

## Vanilla reference

`TranslatableContents.decompose` keeps references to its arguments, so vanilla's cost stays linear; any real reason renders exactly as before.

## Other projects' code

None.

## Self-check

- `cargo test --locked -p horizon-protocol --lib -- text::`: 10 passed
- `cargo test --locked -p horizon-client-lib --no-default-features --lib -- net::tests::a_login`: passed
- `cargo clippy -p horizon-protocol --all-targets --no-deps -- -D warnings`: clean
- `cargo fmt --all -- --check`: clean

🤖 Generated with [Claude Code](https://claude.com/claude-code)
