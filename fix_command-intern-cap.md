Commands: a server's registry names can't leak without bound
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

Every `Commands` packet becomes a dispatcher (`remote.rs`), and every argument node that the vanilla tree doesn't match and that uses a resource-style parser goes through `argument_type` → `intern` (`horizon-command/src/network.rs`). `intern` **`Box::leak`s** its registry name for the life of the game. The name comes straight from the server: up to 32 767 bytes, unchecked. A malicious server sending Commands packets over and over, each with hundreds of nodes carrying new 32 KiB names, leaks megabytes per packet, never freed, until the client runs out of memory. Vanilla keeps these as `ResourceKey`s owned by the tree, freed when the tree is replaced.

## The fix

`intern` keeps a name only when it:

- is a resource location (`a-z 0-9 _ - . / :`);
- is at most 256 bytes;
- fits in the set, which holds at most 1024 names (a game knows a few hundred registries).

Anything else is kept as `""`: that argument gets no suggestions, and the leak stops. The logic is in `intern_in`, so the test can use a set of its own instead of the game-wide one shared with other tests.

## Tests

`a_servers_registry_names_are_kept_within_bounds`:

- a real name is kept once, as the same pointer;
- a 32 KiB name and a non-resource-location name are refused;
- after 2048 distinct names the set holds exactly 1024 and refuses more;
- kept names stay.

Found in a security review of Horizon (11 Nov, High).

## Kind

- [ ] feature
- [x] fix (no label)
- [ ] skip-release

## Vanilla reference

Vanilla's argument types hold `ResourceKey`s owned by the command tree; nothing outlives the tree. Suggestions for real registries are unchanged.

## Other projects' code

None.

## Self-check

- `cargo test --locked -p horizon-command --lib`: 54 passed
- `cargo clippy -p horizon-command --all-targets --no-deps -- -D warnings`: clean
- `cargo fmt --all -- --check`: clean

🤖 Generated with [Claude Code](https://claude.com/claude-code)
