Registry: count native-build limits on a header the deployment names
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

The mod registry limits native-build requests (12 an hour per address) and polls (120 an hour). It keyed those limits on `X-Real-IP`, else the last `X-Forwarded-For` hop, read from any request. Unless the proxy in front always overwrites `X-Real-IP`, a client could send a new value each time, escape the per-address limit and spend the global budget alone: 120 build dispatches an hour, each a GitHub Actions run. After that nobody else could request a build.

The telemetry server already solves the same problem safely with `HORIZON_TELEMETRY_CLIENT_IP_HEADER`; the registry did not.

## The fix

- New setting `REGISTRY_CLIENT_IP_HEADER` (`Config::client_ip_header`) names the one header the proxy overwrites. Only that header is read (its last comma-separated entry).
- Otherwise the TCP peer's address is used. The server now runs with `into_make_service_with_connect_info`, and a small `Peer` extractor reads it, falling back to "anonymous" when there is no connect info, as in the router tests.
- The Dockerfile sets `REGISTRY_CLIENT_IP_HEADER=x-real-ip`, the header Railway's edge overwrites (as the telemetry README says), so the deployment keeps counting the same address. The README's settings table documents it.

## Tests

`a_client_cannot_pick_its_own_rate_limit_address` now checks:

- with no header named, the peer counts whatever the client sent;
- only the named header is read (`x-real-ip`, or the last hop of `x-forwarded-for`);
- the peer is used when the named header is missing.

`tests/test_service_images.py` still passes.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Not a vanilla feature (mod registry service).

## Other projects' code

None.

## Self-check

In `crates/horizon-mod-registry`: `cargo test` (all pass), `cargo clippy --all-targets -- -D warnings`, `cargo fmt --check`. `python3 -m unittest tests.test_service_images` passes.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
