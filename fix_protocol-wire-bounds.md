Protocol: wrap wire ids as Java does, and keep vanilla's caps on frames, cookies and profiles
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

Seven Low findings in `horizon-protocol`, all about bounds on what a peer sends:

1. **Overflow on holder ids.** `OptVarInt` and many holder reads did `v - 1` (and writes `v + 1`). A peer sending i32::MIN panics debug builds; release builds wrap, as Java does. The sites:
   - `buf.rs`;
   - `packets/recipes.rs` (HolderSet, TrimPattern read and write);
   - `packets/play.rs` (HolderId, DialogHolder, PackedSignature, SoundHolder);
   - `item/v1_21_1.rs`, `item/v1_21_4.rs`, `item/v1_21_5.rs` (holder set count);
   - `item/hash/raw.rs`, `item/hash/gameplay.rs`;
   - `adapter/client_session.rs` (an old client's action).
2. **Frame allocation.** With compression on, a few-byte frame declaring 8 MiB made the reader reserve 8 MiB before inflating.
3. **Cookies.** Vanilla's cookie payload is `ByteBufCodecs.byteArray(5120)`. Horizon accepted anything up to the packet's length, and the client kept and forwarded it across transfers.
4. **Profile properties.** A login `GameProfile` accepted any number of properties. Vanilla's `GAME_PROFILE_PROPERTIES` takes 16, as Horizon's own entity-data reader already did.
5. **AgeableMob shift.** For 1.21.11-and-older servers, entity-data index 254 was shifted to 255, the list terminator.
6. **Quiet commands.** `quiet::encode` cut at Unicode scalars while `decode` counts UTF-16 units, so a line of emoji passed encode and got the whole payload refused.
7. **Old dust colours.** Pre-1.21.2 dust colours were packed without masking each channel, so a component outside 0..1 bled into its neighbours.

## The fix

1. `wrapping_sub(1)` / `wrapping_add(1)` at each site, which matches Java and release builds. The 1.21.5 equip-sound line is left to the `fix/protocol-unmapped-ids` pull request, which rewrites it.
2. `Inflater::inflate` starts at `min(size + 1, input × 1032 + 64)` and grows as the stream fills it, still capped at `size + 1`.
3. New `MAX_COOKIE = 5120` with `BytesN<MAX_COOKIE>` for `StoreCookie` and both `CookieResponse`s.
4. New `buf::ListN<C, N>` (`ByteBufCodecs.list(N)`); `GameProfile.properties` uses `ListN<Property, 16>`.
5. Only indices `17..0xFE` are shifted.
6. `utf16_prefix` cuts the lines and the mod id in `encode`.
7. `(c & 0xFF)` for each channel.

## Tests

- `an_optional_var_int_wraps_as_java_does` and `a_capped_list_refuses_one_more` (buf.rs).
- `large_bodies_inflate_whole_and_short_streams_are_refused` (frame.rs): 2 MiB inflates whole through the growing buffer, a short stream declaring 8 MiB is refused, and the inflater still works after.
- `cookies_and_profile_properties_keep_vanillas_caps` (login.rs): 5120 / 5121 bytes, 16 / 17 properties.
- `lines_are_cut_in_utf16_units_so_decode_takes_them` (quiet.rs).
- `a_legacy_dust_colour_keeps_each_channel_in_its_byte` (entity_data.rs).

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

- `ClientboundStoreCookiePacket` / `ServerboundCookieResponsePacket`: `byteArray(5120)`.
- `ByteBufCodecs.GAME_PROFILE_PROPERTIES`: `list(16)`.
- `ARGB.color` masks each channel.
- Java int arithmetic wraps.

## Other projects' code

None.

## Self-check

`cargo test -p horizon-protocol --lib` (455 passed), `cargo clippy -p horizon-protocol --all-targets -- -D warnings`, `cargo check -p horizon-client --no-default-features`, `cargo fmt --check`.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
