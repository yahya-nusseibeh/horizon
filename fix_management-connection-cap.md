Cap management and API connections, and give a request one deadline
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

The management server (vanilla's server management protocol, meant for remote tools) spawned one thread per connection with no cap. Before any authentication, it read the upgrade request with only a 10 s timeout per read. So:

- sending a byte every few seconds held a connection, and its thread, forever;
- a `Content-Length` of 1 MiB made each connection allocate 1 MiB up front.

An unauthenticated client could open thousands of connections and exhaust the supervisor's memory and the user's thread limit, which the game process shares. The web panel and control API use the same `http::serve`; they default to loopback but are exposed the same way with `panel.allow_public`.

## The fix

- `http::read_request` reads the whole request (line, headers, body) under one deadline, `REQUEST_DEADLINE` (10 s), through a small `Deadline` reader wrapper.
- The body grows as bytes arrive (`take(length).read_to_end`) instead of `vec![0; length]`.
- `http::Slot` is an RAII slot over an `AtomicUsize`, taken with a compare-exchange loop. Both the management listener and `http::serve` hold at most `MAX_OPEN` (64) connections; past that a new one is closed unanswered. Authenticated management sockets also hold a slot, so 64 tools at once remains the limit.

## Tests

- `a_request_trickled_past_its_deadline_is_refused`: one byte every 20 ms is refused under a 200 ms deadline, and the same request read quickly parses.
- `a_body_that_never_comes_is_not_allocated_up_front`: a 1 MB `Content-Length` with 3 bytes of body gets a 400.
- `slots_cap_open_connections_and_come_back`: the cap holds and dropped slots come back.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Vanilla's management server is off by default (`management-server-enabled=false`); this changes only how Horizon's implementation bounds its connections.

## Other projects' code

None.

## Self-check

`cargo test -p horizon-server` (178 passed), `cargo clippy -p horizon-server --all-targets -- -D warnings`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
