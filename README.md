# Twin-T Code Practice Oscillator (Amogus Edition)

A battery-powered code practice oscillator for amateur radio operators. The
board is a two-layer, through-hole design built around a twin-T audio
oscillator feeding a pair of NE5532 buffers. The "Amogus" edition is a
novelty layout shaped and silkscreened as a crewmate.

## Overview

The oscillator core is a discrete common-emitter stage (Q2, 2N4401) with a
twin-T notch network in its feedback path. A MOSFET (Q1) buffers the key
input and drives a second MOSFET (Q3) that modulates the notch, giving a
clean keying envelope with a minimum of clicks. Two LEDs provide power and
key-down indication. The audio output is buffered by both halves of an
NE5532 and fed to a 1/8" headphone jack.

## Specifications

| Parameter | Value |
|-----------|-------|
| Supply | 9 V battery (6F22 / PP3) |
| Supply current | ~2 mA quiescent |
| Oscillator frequency | ~480 Hz nominal (1/(2πRC), R = 3.3 kΩ, C = 100 nF); verify by measurement |
| Output | 1/8" (3.5 mm) headphone jack, AC-coupled |
| Board size | 120.6 × 120.0 mm, 2-layer, 1.6 mm FR4 |
| Keying | MOSFET-switched twin-T notch; no audible clicks in normal use |

## Repository contents

| Path | Description |
|------|-------------|
| `pcb/` | KiCad project (schematic, board, Gerbers, drill) |
| `pcb/TwinTCPO.kicad_pcb` | KiCad board file (rev A as built, 2021) |
| `pcb/TwinTCPO.kicad_sch` | KiCad schematic (re-saved 2023, see rev notes) |
| `pcb/TwinTCPO*.gbr`, `pcb/TwinTCPO.drl` | Committed fabrication outputs (2021) |
| `BOM.csv` | Bill of materials, generated from the board's as-built values |
| `docs/REV-A-NOTES.md` | As-built revision notes and known discrepancies |
| `media/` | Artwork and mechanical assets (SVG, PDF, TIF, XCF) |

## Build notes

1. Use `BOM.csv` for parts; it reflects the **board as fabricated** (rev A,
   2021), not the current schematic. The `Notes` column flags every reference
   whose schematic value differs.
2. Read `docs/REV-A-NOTES.md` before ordering parts. In particular, **C11 is
   1 µF on the board but 100 µF in the schematic, and the schematic is the
   correct value** — at 32 Ω the as-built 1 µF rolls off from ~5 kHz and costs
   about 20 dB at the 480 Hz tone. Use 100 µF (or 47 µF) unless you specifically
   intend high-impedance headphones. **R6 is the opposite case: the board's
   3.3 kΩ is correct and the schematic's 330 Ω is the error.**
3. The committed Gerbers and drill file regenerate exactly from
   `TwinTCPO.kicad_pcb`; they are safe to send to a fabricator as-is.
4. The board has no mounting holes. Confirm your enclosure retains it.

## License

This design is licensed under the Creative Commons Attribution-ShareAlike 4.0
International License. See `LICENSE` for the full text. You are free to share
and adapt the design, provided you credit the original author and distribute
derivatives under the same terms.

## Links

- [Etherkit](https://www.etherkit.com)
- [NT7S / Jason Milldrum](https://nt7s.com)
