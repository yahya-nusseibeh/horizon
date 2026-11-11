horizon-server: a space in a config key can't get past the API's deny-list
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

`PATCH /api/v1/config` (`api.rs`) checks each key with `settings::changeable_remotely`, which **doesn't trim** it, and then applies `"{key}={value}"` through `settings::apply`, whose `parse_change` **does trim** it. So a key such as these passed the deny-list and was written to `server.toml`:

- `" engine.path"`
- `"\tapi.allow_public"`
- `" discord.webhook"`
- `" properties.enable-rcon"`

The code's own comment says the API must never change these. Anyone with the API token or a panel session can send such a key, and so can an MCP agent through `set_config`; that agent reads player chat, so players could prompt-inject it. At the next restart (which `/api/v1/restart` can request), the API or panel opens to the internet, chat goes to another webhook, RCON is enabled, or the supervisor runs another program.

## The fix

`changeable_remotely` refuses any key containing whitespace or control characters. What it checks is now exactly what gets applied.

## Tests

`keys_the_api_may_not_change_stay_refused_however_spaced`: leading, trailing and inner spaces, tabs and newlines around denied keys are all refused; ordinary keys (`world.difficulty`, `properties.motd`) still pass.

Found in a security review of Horizon (11 Nov, High).

## Kind

- [ ] feature
- [x] fix (no label)
- [ ] skip-release

## Vanilla reference

None: the supervisor's API is Horizon's own.

## Other projects' code

None.

## Self-check

- `cargo test --locked -p horizon-server -- settings::`: 8 passed
- `cargo clippy -p horizon-server --all-targets --no-deps -- -D warnings`: clean
- `cargo fmt --all -- --check`: clean

🤖 Generated with [Claude Code](https://claude.com/claude-code)
