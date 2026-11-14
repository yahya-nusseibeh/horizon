Pack bench: pass the base ref to the summary through the environment
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

The `Summary` step of `.github/workflows/pack-bench.yml` pasted `${{ inputs.base || 'main' }}` straight into its shell script. A dispatched base name with shell syntax in it would run as a command on the runner. The step before already passes the same input through `env`.

Split out of #15258 at review: workflow files need council review.

## The fix

The base goes in through `env: BASE`, and the script reads `$BASE`. The output is the same.

## Tests

None: a workflow-only change. The step's output is unchanged for a normal base name.

## Kind
- [ ] Fix
- [ ] Feature
- [x] Skip release

## Other projects' code

None.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
