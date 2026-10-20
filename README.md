# CALibrate

**Make CAL someone else.**

CALibrate is a single-page, in-browser tool for Pokémon Crystal save files. In Crystal's Viridian Trainer House, the trainer named **CAL** isn't fixed: he uses the name and team of whoever you last used Mystery Gift with. CALibrate gives that data to a save file directly. Load two saves and each one's CAL becomes the other player.

Everything runs locally in your browser. Files are never uploaded.

## Usage

1. Open https://codymkw.github.io/CALibrate/ (or `index.html` locally).
2. Drop a Crystal `.sav` into **Save A** and another into **Save B**. No second save? Click **Face Grand Master Claude** instead (see below).
3. Choose what CAL copies:
   - **The other player's party** (default): builds CAL's team from the other save's current party, exactly as a real Mystery Gift link would.
   - **Their stored CAL**: copies whatever Mystery Gift trainer the other save already has stored. Only available if that save has one.
4. Download either or both patched saves (`<name>-calibrated.sav`).
5. Load the patched save in your emulator or flash cart and visit the Trainer House in Viridian City.

Back up your original saves first. The tool never modifies your originals, but the patched file replaces nothing until you copy it over.

## Grand Master Claude

A built-in opponent for when you don't have a second save. Your CAL becomes **GM Claude** (the game limits names to 10 characters), with a Lv100 team and fixed moves:

| Pokémon | Moves |
|---------|-------|
| Tyranitar | Rock Slide, Crunch, Earthquake, Pursuit |
| Starmie | Surf, Psychic, Thunderbolt, Recover |
| Zapdos | Thunder, Hidden Power, Drill Peck, Rest |
| Snorlax | Double-Edge, Earthquake, Curse, Rest |
| Skarmory | Drill Peck, Steel Wing, Rest, Sleep Talk |
| Blissey | Softboiled, Ice Beam, Thunder Wave, Counter |

Only your own save is patched in this mode.

## What it changes

CAL's data lives in SRAM bank 0, in the Mystery Gift block (the game reads it for trainer class CAL, ID 2, the Trainer House fight). That block is **outside** the checksummed game data (`0x2009-0x2B82`), so the save's checksum stays valid and nothing else in the file changes.

Total change: 51 bytes.

| Offset | Size | Field | Written value |
|--------|------|-------|---------------|
| `0x0BFD` | 1 | `sMysteryGiftTrainerHouseFlag` | `1` |
| `0x0BFE` | 11 | `sMysteryGiftPartnerName` | Other player's name, raw 11 bytes |
| `0x0C09` | 1 | `sMysteryGiftUnusedFlag` | `1` |
| `0x0C0A` | 38 | `sMysteryGiftTrainer` | Team data, see below |

### Team format (38 bytes)

Repeated per Pokémon, then terminated:

```
level, species, move1, move2, move3, move4     (6 bytes each, up to 6 Pokémon)
0xFF                                            (terminator, rest is zero)
```

When building from a party, the tool mirrors the game's `StagePartyDataForMysteryGift`: it reads each party slot's level, species and moves, and skips Eggs.

### Reference offsets used for reading

| Offset | Meaning |
|--------|---------|
| `0x2008` | Check value 1, must be `0x63` |
| `0x2D0F` | Check value 2, must be `0x7F` |
| `0x2009-0x2B82` | Checksummed game data (16-bit sum of bytes) |
| `0x2D0D` | Checksum, little-endian |
| `0x200B` | Player name (11 bytes, Gen 2 text encoding) |
| `0x2865` | Party count |
| `0x2866` | Party species list (6 + terminator) |
| `0x286D` | Party Pokémon structs, 48 bytes each (species `+0x00`, moves `+0x02`, level `+0x1F`) |

Offsets are derived from the [pret/pokecrystal](https://github.com/pret/pokecrystal) disassembly and checked against a real save.

## Checks and limits

- Accepts saves of 32 KB or larger. Extra trailing data (for example, emulator RTC data) is left untouched.
- Rejects files without Crystal's two check bytes. If the checksum doesn't match, the tool warns you but still works.
- Refuses to give CAL an empty team (a source save with no battle-ready Pokémon).
- Built for international Crystal. Japanese Crystal and Gold/Silver have different layouts, and Gold/Silver have no Trainer House.
- CAL's team uses the species, level and moves only. Held items, stats and nicknames are not part of the data, so he fights with default stats for those Pokémon.

## Try it

https://codymkw.github.io/CALibrate/

The only external request is the Google Fonts stylesheet. The page falls back to system fonts without it.

## Files

```
index.html   the whole app (HTML, CSS, JS)
README.md    this file
```

## Disclaimer

Unofficial fan project. Not affiliated with or endorsed by Nintendo, Game Freak, Creatures or The Pokémon Company. Pokémon is a trademark of its respective owners. No game assets or ROM data are included.

## Credits

- [pret/pokecrystal](https://github.com/pret/pokecrystal) for the documented save layout and Mystery Gift routines.
