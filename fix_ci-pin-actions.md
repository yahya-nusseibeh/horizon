CI: pin the third-party Rust actions to reviewed commits
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

`dtolnay/rust-toolchain@stable` is a branch and `Swatinem/rust-cache@v2` a movable tag. Both run inside `game-build.yml`'s `build` job, which compiles and packages the Linux, Windows and macOS archives players download. The macOS rows run on the self-hosted Mac mini. The action code, and rust-cache's post step, run before the archives are uploaded, and `nightly-build.yml` then signs `SHA256SUMS` over them. So anyone who could move one of those refs, for example by compromising one of those repositories, could change released binaries and get a valid Horizon signature. The same refs appear in the other workflows.

## The fix

- Every `dtolnay/rust-toolchain@stable` is now `@b725eda525362e7cf34af9a7243cac1e3f01a7ec # stable`, the head of its `stable` branch today.
  - With a commit ref the action can no longer read the toolchain from the ref name, so each step passes `toolchain: stable`, added to an existing `with:` or as a new one.
  - Steps with an `if:` keep it.
- Every `Swatinem/rust-cache@v2` is now `@d1527c8f67ef9af357290475d4af4b0fab8f12e0 # v2.9.3`, the commit `v2` and `v2.9.3` point at today.
- Files: `cargo-horizon-mod`, `ci`, `game-build`, `launcher`, `mod-registry`, `nightly`, `pack-bench`, `parity-main`, `parity`, `telemetry-server`.
- GitHub's own `actions/*` are left as they are.

The SHAs were read with `git ls-remote` against each action's repository. Future bumps need a person (or Dependabot's `github-actions` ecosystem, if the maintainer wants it) to update the SHA.

## Tests

- Every workflow still parses as YAML.
- `python3 -m unittest discover -s tests`: 848 tests, OK, including `test_nightly` (one rust-cache use in game-build) and `test_ci_runners`.

## Kind
- [x] Fix
- [ ] Feature
- [x] Skip release

## Vanilla reference

Not a vanilla feature (CI).

## Other projects' code

None (references to the same actions, pinned).

## Self-check

YAML parse of every workflow, `python3 -m unittest discover -s tests` (OK). No Rust changes.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
