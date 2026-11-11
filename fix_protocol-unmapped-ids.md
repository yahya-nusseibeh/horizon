Protocol adapter: never send or read an unmapped id as -1
---
> 🤖 Opened by yahya-nusseibeh's AI agent (Claude Code, Claude Opus 5.5), not by yahya-nusseibeh personally.

## What and why

`ids::*` returns -1 for an id the other version lacks. Four places in the protocol adapter used that -1 as if it were an id:

1. **1.21.5 Equippable sounds.** `sound()` wrote `ids::sound_event(holder - 1) + 1`. For a missing sound that is 0, which means "a direct sound follows" (identifier, optional range), but nothing followed. The stack was then corrupt: an old client was disconnected each time the item was shown, and on the way up the bad bytes were kept as the component's raw value. 293 pinned sounds have no 1.21.5 id.
2. **Particles from old servers.** An id the pinned registry lacks maps to -1, which is also `MOD_PARTICLE_ID`. `Particle::read` then read the following bytes (the next entity-data entry) as a mod particle's name. 1.20.3's table has gaps, and any server can send an out-of-range id.
3. **Creative-slot and container-click stacks sent to a 1.21.5–26.2 server.** `DelimitedSlot::write` and `HashedSlot::write` wrote VarInt -1 for an item or component the server lacks, which the server can't decode, so it disconnects the player. `write_slot` / `write_patch` already skip these.
4. **Hashed stacks from a 1.21.5–26.2 client.** `HashedSlot::read` never mapped ids, unlike every other stack reader. A Horizon server compared the client's own item ids with pinned ones, so most clicks looked like desyncs and the menu was resent.

## The fix

1. A missing equip sound is an error (the component isn't rewritten). The sound that 1.21.5 doesn't have a field for (`shearing_sound`) is skipped with a new `skip_sound`, without mapping.
2. A particle id that maps to -1 is an error, "particle type N has no pinned counterpart".
3. Both writers send an item the server lacks as no stack and leave out components it lacks.
4. `HashedSlot::read` maps item and component ids. The 1.21.4 client step now writes its hashes in the client's own ids (under `Way::Down`), so the session's single mapping up gives the pinned ids, as for newer clients.

## Tests

- `an_equip_sound_the_other_side_lacks_is_refused_not_written_as_direct` (and a known sound is written in 1.21.5's id).
- `an_old_particle_with_no_pinned_counterpart_is_refused`.
- `hashed_stacks_go_to_an_older_side_and_back_in_its_ids`: a moved item round-trips through 1.21.5's down and up ids.
- `an_item_the_older_side_lacks_goes_as_no_stack_not_as_minus_one`.

The existing vanilla-oracle stack tests (1.21.3, 1.21.4, 1.21.5) and the 1.21.4 container-click round trip still pass.

## Kind
- [x] Fix
- [ ] Feature
- [ ] Skip release

## Vanilla reference

Vanilla's registry stream codecs look ids up by id and throw on an unknown one. `SoundEvent.STREAM_CODEC`'s holder 0 means a direct event.

## Other projects' code

None.

## Self-check

`cargo test -p horizon-protocol` (453 passed), `cargo clippy -p horizon-protocol --all-targets -- -D warnings`, `cargo fmt --check`. `cargo test -p horizon-client-lib --lib -- lan::containers` passes.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
