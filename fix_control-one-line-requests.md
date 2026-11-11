horizon-server: a management kick reason can't inject control requests
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

`players/kick` (`management_methods.rs`) took the reason from the management client's `message` (`literal` or `translatable`), newlines included, and built `console kick <name> <reason>`. `Control::ask` (`engine.rs`) writes that with `writeln!` to the game's control endpoint, which reads **one request per line** and keeps reading after the supervisor drops the connection. So the request

```json
{"player":{"name":"Kim0"},"message":{"literal":"x\nconsole op Mallory"}}
```

ran a second request: any console command (`op`, `stop`, `fill`, `kill @e`…) or any other control verb. Vanilla's management API offers none of these. The control API's own kick already refused control characters; this path didn't.

## The fix

- The kick reason drops control characters.
- `Control::ask` refuses a request containing `\n` or `\r` with `InvalidInput`, **before sending anything**. Every caller is covered, not only the kick.
- The API's `/command` now refuses `\r` too (it already refused `\n`).

## Tests

`a_request_with_a_line_break_is_refused_unsent` (`engine.rs`): against a fake control endpoint, requests with `\n` or `\r` are refused, and the endpoint only ever receives `auth` and the next good request.

Found in a security review of Horizon (11 Nov, High).

## Kind

- [ ] feature
- [x] fix (no label)
- [ ] skip-release

## Vanilla reference

Vanilla's `players/kick` takes a component as the reason and offers no other command; a reason never runs anything.

## Other projects' code

None.

## Self-check

- `cargo test --locked -p horizon-server -- engine:: management api::`: 47 passed
- `cargo clippy -p horizon-server --all-targets --no-deps -- -D warnings`: clean
- `cargo fmt --all -- --check`: clean

🤖 Generated with [Claude Code](https://claude.com/claude-code)
