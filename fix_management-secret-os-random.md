Server files: the management secret comes from OS randomness, not the seed's stream
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

`SystemEntropy` (`horizon-server-files/src/properties.rs`) is splitmix64 seeded from 8 bytes, and the dedicated server uses **one** of them for both the world seed and the management secret (`dedicated.rs`). On a first start with no `management-server-secret` and an empty `level-seed` (both defaults):

- the 40 secret characters are outputs 1–40 of that stream;
- the world seed is output 41.

splitmix64's output mixer is a bijection and its state steps by a constant, so the world seed gives back the state and, stepping back, every secret character. The world seed isn't secret: ops read it with `/seed`, and seed-cracking tools recover vanilla-generated seeds from terrain. When the owner later turns `management-server-enabled` on, `management.rs` reuses the secret already in `server.properties`, so anyone who knows the seed holds the management API (op, ban, stop…). On Windows there is no `/dev/urandom`, so the seed fell back to the clock and the secret depended only on the start time. Vanilla draws the secret from `SecureRandom`.

## The fix

- New `OsEntropy`: every draw straight from `getrandom`, with rejection sampling (no modulo bias), as `SecureRandom`.
- The management secret (`ServerProperties::new`, `management.rs`) and the TLS keystore password (`management_tls.rs`) come from `OsEntropy`, never from the seed's stream.
- `SystemEntropy` itself is now seeded from `getrandom` (the clock only if that fails) instead of `/dev/urandom` or the clock.
- `horizon-server-files` gains `getrandom = "0.3"`, already in the lock.

**Owners who already generated a secret this way should rotate it** (delete `management-server-secret` from `server.properties` and restart).

## Tests

- `the_secret_never_comes_from_the_seed_stream`: two first starts from the same seed stream get the same world seed but different secrets, neither equal to the stream's own first 40 draws.
- `defaults_match_vanilla_init_settings` now checks the secret's length and alphabet rather than a value fixed by the test entropy.

Found in a security review of Horizon (11 Nov, High).

## Kind

- [ ] feature
- [x] fix (no label)
- [ ] skip-release

## Vanilla reference

`SecurityConfig.generateSecretKey` draws from `SecureRandom`, apart from `RandomSource.create()`.

## Other projects' code

None.

## Self-check

- `cargo test --locked -p horizon-server-files --lib -- properties`: 9 passed
- `cargo test --locked -p horizon-server -- management`: 22 passed
- `cargo clippy -p horizon-server -p horizon-server-files --all-targets --no-deps -- -D warnings`: clean
- `cargo fmt --all -- --check`: clean

🤖 Generated with [Claude Code](https://claude.com/claude-code)
