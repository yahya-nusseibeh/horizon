Protocol: grow the inflate buffer instead of reserving the declared size
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

With compression on, a frame declares its uncompressed size before the zlib stream. `Inflater::inflate` reserved `size + 1` bytes up front, so a peer sending a few bytes that declare 8 MiB (`MAX_UNCOMPRESSED`) made the reader allocate 8 MiB for every such frame before finding the stream short.

Split out of #15249 at review.

## The fix

`Inflater::inflate` starts at `min(size + 1, input × 1032 + 64)` (DEFLATE expands at most about 1032:1) and reserves more as the stream fills the buffer, still capped at `size + 1`. The checks after it are unchanged: a stream that isn't exactly `size` bytes is refused with vanilla's message.

## Tests

`large_bodies_inflate_whole_and_short_streams_are_refused` (frame.rs): 2 MiB of zeros inflate whole through the growing buffer, a short stream declaring 8 MiB is refused, and the same inflater still reads a good frame after. `a_reused_compressor_writes_the_same_bytes` and the other frame tests still pass.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

`CompressionDecoder` refuses a frame whose inflated length differs from the declared one; this keeps that check.

## Other projects' code

None.

## Self-check

`cargo test -p horizon-protocol` (454 passed), `cargo fmt --check`. (Local clippy stops on an unknown lint in horizon-core on main, from an older toolchain here; CI runs it.)

🤖 Generated with [Claude Code](https://claude.com/claude-code)
