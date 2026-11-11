Relay: keep real codes joinable and registrations bounded per site
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

Three ways one client could degrade the Play with Friends relay for everyone:

1. **The global unknown-code budget locked out real codes.** Every join was refused when the relay-wide unknown-code bucket (120/min, burst 30) was empty, including joins with a valid code. One site may spend 24/min, so about five /24s or /48s sending random nameplates kept it empty, and every guest on the relay got `RateLimited`.
2. **`RELAY_PROXY_FROM` was checked only after the first line.** With `RELAY_PROXY_PROTOCOL=1`, a connection from outside the load balancers was counted nowhere and closed only after its first line or `first_line` (10 s). The docs say such connections are "closed unanswered". If the port is reachable directly, one address could hold all 4,096 connection slots.
3. **Registrations were limited per /64 only.** One IPv6 /48 could register and disconnect quickly. Each ended nameplate stays for 15 minutes, and `ended.retain` scanned the whole map under the global lock on every registration. That slowed joins and `/health` relay-wide and could fill the nameplate space.

## The fix

1. The per-address and per-site budgets are still checked before the lookup. The relay-wide one now refuses only a code that turns out unknown. The trade-off: while it is spent, a refusal tells a prober the code is unknown. Guessing a code's secret half stays capped by its host (5 per code, 50 per session).
2. Peers outside `proxy_from` are closed before anything is read.
3. A per-site register budget (`site_register_per_min` 30, `site_register_burst` 10, swept with the other site budgets). An address refused by its own budget spends nothing of its site's. `ended` gains an insertion-ordered queue and expires oldest first, stopping at the first entry still kept.

## Tests

- `a_spent_global_budget_refuses_unknown_codes_but_not_real_ones` (new `tests/global_unknown.rs`): with a one-code global budget, a second unknown code is rate limited but the host's real code is not.
- `anyone_else_is_closed_before_the_first_line` (`tests/proxy_from.rs`): a silent connection from outside `proxy_from` is closed at once with `first_line` at 30 s.
- `one_site_registers_only_so_often_across_its_addresses` (limits.rs): 100 /64s of one /48 get the site's burst, another site is unaffected, and an address over its own budget doesn't spend its neighbours'.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Not a vanilla feature (Horizon relay).

## Other projects' code

None.

## Self-check

`cargo test -p horizon-relay` (all pass), `cargo clippy -p horizon-relay --all-targets -- -D warnings`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
