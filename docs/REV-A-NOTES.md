# Revision A — As-Built Notes and Known Discrepancies

**Board:** rev A, fabricated 2021-07-21 · **Fabrication outputs:** Gerbers and
drill plotted 2021-07-22 by KiCad 5.1.10 · **Schematic:** title block rev A,
dated 2021-05-17, re-saved in KiCad 7 format (version 20230121) at some point
after fabrication.

This note records what was actually built, what disagrees with the schematic,
and what should change on a rev B. It exists because the schematic and the
board no longer agree, and both still carry rev A.

## 1. Fabrication data is faithful

Regenerating Gerbers and drill from `pcb/TwinTCPO.kicad_pcb` reproduces the
committed outputs exactly:

- Copper flash coordinates identical on both layers (42/42 on F.Cu and B.Cu).
- 89 drill hits across the same 10 tools.
- Track count consistent: 184 segments + 2 vias = 186 as declared in the file header.

The committed fab outputs are therefore a valid snapshot of the board file and
can be sent to a fabricator as-is.

## 2. Schematic ↔ board value discrepancies

Five references differ between the board (as built) and the schematic. The
schematic is the later file, so these are likely intended changes that never
reached the board — but that is unconfirmed.

| Ref | Board (as built) | Schematic | Effect |
|-----|------------------|-----------|--------|
| C11 | 1 µF | 100 µF | Output coupling into headphones. 1 µF into 32 Ω gives a −3 dB corner near 5 kHz; 100 µF gives ~50 Hz. **Verify against the intended headphone impedance.** |
| R6 | 3.3 kΩ | 330 Ω | Shunt leg of the keying twin-T. A 10× change alters notch depth and keying click/envelope. |
| R1 | 10 kΩ | 100 kΩ | Q1 gate pull-down. Benign either way. |
| R3 | 10 kΩ | 100 kΩ | Q1 drain pull-up driving Q3 gate. Benign; changes key-down current only. |
| R13 | 4.7 kΩ | 10 kΩ | Red LED current, ~1.5 mA vs ~0.7 mA. Cosmetic. |

`BOM.csv` uses the board values and flags each discrepancy in its `Notes` column.

## 3. No supply decoupling

The VCC net has eight nodes — two LEDs, five resistors, the power switch, and
U1 pin 8 — and **no capacitor to ground**. The NE5532 and the oscillator
transistor share the raw 9 V rail. C2 (1 µF) is on the mid-rail bias/filter
node, not the supply.

The board works, but supply impedance modulates the oscillator and keying
transients ride the rail into both op-amp stages. A 100 nF in parallel with
10 µF across VCC at U1 is the conventional fix and is recommended for rev B.

## 4. No ground plane

Zero copper zones. 184 track segments, all 0.5 mm wide. A bottom-side ground
pour would improve noise performance and return-path impedance on an audio
board this size, and would reduce etch cost. Recommended for rev B.

## 5. Layout and DFM observations

- **83 DRC violations**, of which 45 are "library not found"
  (`EtherkitKicadLibrary`, `Package_DIP`). The board embeds all footprints, so
  fabrication is unaffected; the design simply cannot be re-verified against
  its libraries. Vendoring the library or exporting the footprints would fix
  reproducibility.
- **4 solder-mask bridges**: all are bottom-side logo artwork mask apertures
  near VCC copper. The `G***` logo footprints carry **zero pads and no copper**,
  so these are cosmetic mask slivers, not electrical shorts.
- **14 silk-over-copper, 20 silk-overlap**: none over pads. Refdes text
  colliding with the Among Us artwork. Cosmetic only.
- **Stray unannotated footprint** `Ref**` / `EtherkitLogoSmall` on F.Cu with
  zero pads — dead artwork left on the board. Delete on rev B.
- **No mounting holes** on a 120 mm board. Confirm the enclosure retains it.
- **Drill file is in inches** (KiCad 5 default) while the board is metric. Fabs
  accept it; metric is cleaner for rev B.
- **2 ERC `power_pin_not_driven` errors**: no PWR_FLAG on VCC or GND. Benign,
  but it prevents ERC from validating the power nets. Add PWR_FLAG on rev B.

## 6. Repository hygiene

- Two schematic sources are committed: `TwinTCPO.sch` (legacy format, 874 lines)
  and `TwinTCPO.kicad_sch` (KiCad 6/7 format, 2 940 lines), plus `*-bak` files.
  Keep one schematic format and remove the backups.
- The repo is ~133 MB for a single-commit history, dominated by 24 media files
  and a Gerber ZIP. Consider moving large artwork out of the repository or
  using Git LFS.
- `pcb/TwinTCPO.pdf` is a plot of the schematic; verify it matches the current
  schematic before relying on it.

## Recommended actions before rev B

1. Decide which revision is authoritative. If the schematic values are
   intended, bump the schematic to rev B and re-run "Update PCB from
   Schematic" and re-plot Gerbers. If not, revert the schematic to match the
   board.
2. Confirm C11 against the intended headphone load — this is the change with an
   audible consequence.
3. Add 100 nF + 10 µF decoupling at U1 and a ground pour.
4. Remove the `*-bak` files and the duplicate legacy schematic.
5. Add PWR_FLAG symbols to VCC and GND so ERC is meaningful.
