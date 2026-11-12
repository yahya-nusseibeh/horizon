Service: refuse paths a systemd unit would misread
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

`service install` writes the server folder and the program path into the systemd unit as they are, in `Description`, `WorkingDirectory` and a quoted `ExecStart`. systemd reads some characters differently:
- A `%` is a specifier (`%h`, `%u`…).
- A line break starts a new directive.
- A `"` or `\` changes `ExecStart`'s quoting.

So a folder named like that gives a unit that does something other than what was written.

## The fix

`unit_safe` refuses such a path, or one that isn't UTF-8, before the unit is made, with a message saying to move the server or the program to a plainer path. Spaces and non-ASCII letters are still fine. Running a `--system` unit as the account that ran sudo was already fixed on main.

## Tests

`paths_systemd_would_misread_are_refused`: a `%`, a line break with an `ExecStartPre=`, a quote and a backslash are refused, while `/srv/my server/Ünïcode` is accepted. `cargo test -p horizon-server service` passes.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Other projects' code

None.

## Self-check

`cargo test -p horizon-server`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
