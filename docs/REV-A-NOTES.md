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
| C11 | 1 µF | 100 µF | U1B output (pin 7) to J3. 1 µF into 32 Ω corners at **4.97 kHz**; 100 µF at **49.7 Hz**. If J3 parallels both earpieces (~16 Ω), the corner rises to ~10 kHz. **The as-built 1 µF attenuates the 480 Hz tone by roughly 20 dB** — the schematic's 100 µF (or 47 µF) is the correct value unless high-Z headphones were intended. |
| R6 | 3.3 kΩ | 330 Ω | Shunt leg of the keying twin-T. The board's **3.3 kΩ is correct** — it matches R2/R7 (3.3 kΩ) and C1/C5 (100 nF). 330 Ω is 10× low and detunes the C-C branch upward: combined notch moves from ~0.4 kHz to ~0.8 kHz and rejection at 480 Hz drops from ~18 dB to ~15 dB. **The schematic's 330 Ω is the error here.** |
| R1 | 10 kΩ | 100 kΩ | Pull-up to VCC (not a gate pull-down). Benign either way. |
| R3 | 10 kΩ | 100 kΩ | Q3 gate pull-up — Q1 drain and Q3 gate are the same node. Benign; changes key-down current only. |
| R13 | 4.7 kΩ | 10 kΩ | Red LED current, ~1.5 mA vs ~0.7 mA. Cosmetic. |

`BOM.csv` uses the board values and flags each discrepancy in its `Notes` column.

## 3. No supply decoupling

The VCC net has eight nodes — two LEDs, five resistors, the power switch, and
U1 pin 8 — and **no capacitor to ground**.

Corrected node detail: **C2 is not on the mid-rail bias node.** C2 (1 µF) is the
Q2 collector supply filter — VCC → R4 (47 Ω) → C2 → R5 (3.3 kΩ) → Q2 collector.
The op-amp mid-rail is the R10/R11 (100 kΩ) divider with C6/C7 around it. So the
oscillator transistor's collector *is* filtered, and the "shares the raw 9 V
rail" framing was too strong — but VCC itself still has no capacitor, so U1
sits directly on the battery rail.

The board works, but supply impedance modulates U1 and keying transients ride
the rail into both op-amp stages. A 100 nF close to U1 pin 8 plus 10 µF bulk
near the supply entry is the conventional fix and is recommended for rev B. If
the mid-rail wants stiffening too, add 10 µF across the R10/R11 divider.

## 4. No ground plane

Zero copper zones. 184 track segments, all 0.5 mm wide — fine for this current.
A bottom-side ground pour is worth adding on rev B for return-path impedance and
shielding, with two caveats: keep the pour out from under the high-impedance
twin-T nodes and the op-amp feedback nodes, and stitch it to top-side ground.
At 480 Hz the electrical benefit is modest, so treat this as good practice
rather than a fix for a measured problem.

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

1. Resolve the five value mismatches individually — they do not all point the
   same direction. **C11 and R6 are cases where the schematic is right and the
   board is wrong**; R1, R3 and R13 are benign either way. Bump the schematic to
   rev B for the C11/R6 changes and re-run "Update PCB from Schematic", then
   re-plot Gerbers.
2. Change C11 on the next spin to 100 µF (47 µF also works) unless high-Z
   headphones were intended — as built, 1 µF costs roughly 20 dB at 480 Hz.
3. Add 100 nF + 10 µF decoupling at U1, and a stitched bottom-side ground pour
   kept clear of the twin-T and feedback nodes.
4. Remove the `*-bak` files and the duplicate legacy schematic.
5. Add PWR_FLAG symbols to VCC and GND so ERC is meaningful.
