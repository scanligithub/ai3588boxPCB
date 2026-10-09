# RK3588 + WX8116 OrCAD → KiCad conversion audit

**Status: MODEL-LEVEL CONNECTIVITY VALIDATION PASS; KiCad-native ERC pending.**

## Input
- File: `RK3588_WX8116-SCH-V2T.EDF`
- Size: 19,506,888 bytes
- SHA-256: `3cfa7e49391ec5fa5503bd3a8d252c6c3664a71ce7d962ae3585c42c51e502ae`
- OrCAD Capture EDIF 2.0.0 export.

## Structure and component checks
- Source: 34 schematic pages, 1,932 placed instances, 1,914 unique references, 313 symbol definitions.
- Target: one root schematic + 34 child sheets; custom parser read 35 sheets with 0 parser issues.
- Source and target unique reference sets: identical (0 missing, 0 extra).
- Instance count mismatches: 0.
- Pin-number inventory mismatches: 0.
- Value and footprint field mismatches: 0.

## Pin and geometry checks
- Actual embedded target library pin coordinates checked against OrCAD source: 7,370; local coordinate mismatches: 0; transformed world position mismatches: 0.
- Wire segments: 8,381 source / 8,381 target; exact endpoint multiset differences: 0.
- Junctions: 2,383 source / 2,383 target; exact coordinate differences: 0.
- Power connection points: 947 / 947; net-name/position differences: 0.
- Off-page connector labels: 1,159 / 1,159; canonical name/anchor differences: 0.
- Target labels whose anchor falls on multiple distinct source net geometries: 0.

## Connectivity graph check
- EDF explicit joined groups: 1,428; two are name-only groups with no pin references: `HI3516_USB_DM`, `HI3516_USB_DP`.
- Connected groups with pin references: 1,426.
- Explicitly joined pin tokens: 6,644; every token is present in the generated target graph.
- Connected source nets whose pins remain one component: 1,426 / 1,426.
- Open/split nets: 0.
- Target components merging pins from distinct explicit source nets: 0.

## Source-unjoined pins
There are 7,370 instance pins overall; 726 are not referenced by any explicit EDF joined-net record. All 726 remain isolated in the target graph: none touches a wire, none overlaps a label anchor, none is an unassigned hidden power pin, and none shares a connected component with a pin assigned to an explicit source net. These are preserved as source-level unjoined pins; native ERC may report them and they should be reviewed for intentional No Connect treatment.

## Converter warnings
- Six input instances lack `libraryRef`; a local unique-resolution fallback found their cell in the design library.
- Multiple duplicate/differing symbol-library definitions were merged or emitted as variants; conversion messages are retained in the local converter log.
- Three GPIO alias labels originally coincided with a different-net `CPLD_3V3` wire and were moved along their own signal wire; after the fix no label-anchor cross-net collisions were detected.

## Test and sign-off boundary
- Converter unit tests: 26 passed.
- KiCad `kicad-cli` was not installed, so KiCad-native ERC, native netlist export and GUI smoke test were not run.
- The model-level connectivity checks above pass; full sign-off remains pending until native KiCad ERC/unconnected-pin review is completed.

The full local ZIP has `RK3588_WX8116.kicad_pro`, root/child `.kicad_sch` files, `orcad.kicad_sym`, `sym-lib-table` and the JSON/CSV evidence. The GitHub commit contains audit text only; it does not contain the binary project ZIP.
