Registry: check an edited id, owners by ID, and versions' hashes in CI
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

Moved here from horizon-rust/horizon#15253, as asked there: mod pull requests go to this repository. Ported to this fork's layout (`horizon-mods` is `horizon-mod-package` here).

## What and why

Three Low findings in the mod registry's service and CI tool:

1. **Edit project id.** `page::edit` ("Edit project") put the form's `id` straight into the GitHub contents URL (`/contents/mods/{id}.toml`), the branch name, the commit message and the pull request text. `publish()` checks a new submission's id, but `edit()` didn't, so an id with `/`, `..`, `?` or `#` changed which API path was fetched and written.
2. **Owners by login.** `registry review` decided "isn't one of its owners" by login only. `Entry::owner_ids` exists because GitHub gives a renamed or deleted login to the next account that claims it, and the registry server already uses IDs. A pull request from whoever took over a former owner's login got no warning note.
3. **Version paths in CI.** `registry check` joined each new version's `sha256` onto `--files` without checking its shape. A pull request naming `sha256 = "/proc/self/environ"` or `"../../.git/config"` made CI open and inspect that file and print the outcome to the public run summary. It also never compared the file's hash with the one named. `registry list` printed `file` unchecked into its line-based list, where a newline adds a line.

## The fix

1. `edit()` refuses an id that isn't `horizon_mods::index::valid_id`, as `publish()` does.
2. `review_with_revoked` takes `author_id: Option<u64>`. When it's given and the entry records `owner_ids`, ownership is by ID; otherwise by login as before. `cargo horizon-mod registry review` gains `--author-id <id>` for CI to pass the event's numeric ID; `review()` passes `None`.
3. A new `well_formed` check requires a sha256 of 64 lowercase hex digits and a file name without `/`, `\`, CR or LF.
   - `registry check` skips and counts a malformed version, and fails a file whose computed hash isn't the one named.
   - `registry list` refuses a malformed version (or a version string with a line break) before printing.

## Tests

- `an_owners_login_taken_by_another_account_is_noted`: the owner's ID gives no note, another ID with the same login does, and no ID falls back to the login.
- `only_a_hash_names_a_file` (cargo-horizon-mod).

horizon-mod-package (index feature) and cargo-horizon-mod tests pass on this fork.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Not a vanilla feature (mod registry).

## Other projects' code

None.

## Self-check

On this fork's main: `cargo test -p horizon-mod-package --features index --lib` (182 pass) and `cargo test -p cargo-horizon-mod` (all pass), `cargo fmt --check`. The `crates/horizon-mod-registry` change (page.rs) applied unchanged; its workspace was not rebuilt here.

The registry repository's CI can start passing `--author-id ${{ github.event.pull_request.user.id }}` to `registry review` after this merges.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
