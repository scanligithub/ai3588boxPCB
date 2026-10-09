# RK3588 + WX8116 OrCAD → KiCad conversion audit

**Status: PRELIMINARY CONVERSION GENERATED — electrical connectivity NOT yet certified.**

## Input

- Source: `RK3588_WX8116-SCH-V2T.EDF`
- Bytes: 19,506,888
- SHA-256: `3cfa7e49391ec5fa5503bd3a8d252c6c3664a71ce7d962ae3585c42c51e502ae`
- Format: OrCAD Capture EDIF 2.0.0 export.

## Generated KiCad project (local preliminary package)

- Project to open: `RK3588_WX8116.kicad_pro`
- Root schematic: `RK3588_WX8116.kicad_sch`
- Page sheets: 34 child sheets + root sheet (35 sheet files total).
- OrCAD model: 34 source pages, 1932 placed instances, 1914 unique references, 313 source symbol definitions, 8381 wire segments.
- Parsed target model: 2879 symbol instances including power/auxiliary symbols, 1914 unique component references, 0 parser issues.

## Comparison checks performed locally

- Unique component reference sets: PASS (source 1914, KiCad 1914; source-only 0, target-only 0).
- Pin-number sets per reference: PASS (refs with missing source pin numbers in target: 0; refs with extra target pin numbers: 0).
- Parsed component values per reference: all match.
- Parsed footprint fields per reference: all match.
- KiCad project parser issue count: 0.

The ref/pin checks confirm identity and pin-number inventory only; they do **not** prove that every pin is connected to the correct net.

## Remaining blockers / risks

1. `kicad-cli` was not installed in the execution environment. KiCad's native parser, Electrical Rules Checker (ERC), native netlist export and PDF export were therefore not run.
2. No independent reference netlist (`.asc` or IPC-D-356) was available. The converter's reference-netlist verification stages were skipped.
3. An internal diagnostic compared EDIF `joined` connections with connectivity inferred independently from page-wire geometry after case-folding net names. This check did **not** pass: 43 shared normalized net names have pin-set differences, 5 normalized names appear only in the EDIF joined model, and 42 appear only in the geometry model. See `connectivity_validation_summary.csv`. These differences may include page-port/global-alias and geometric mapping limitations; they must be resolved or confirmed against a true reference netlist before calling the electrical conversion correct.
4. The input EDIF omits `libraryRef` in six instances. A local compatibility fallback resolved these from the design library; all six resolved without an unknown-cell error. Other warnings about duplicate/inconsistent symbol definitions and canonical power-net aliases need review.
5. The project has not been opened in KiCad GUI in this runtime.

## Local preliminary package contents

The ZIP generated in the conversation contains `RK3588_WX8116.kicad_pro`, root `.kicad_sch`, 34 child `.kicad_sch` sheets, `orcad.kicad_sym`, `sym-lib-table`, and supporting CSV/log files including `component_pin_audit.csv`, `edif_joined_netlist.csv`, `wire_geometry_netlist.csv` and `edif_geometry_diff.csv`.

## Generation note

This run used orcad2kicad 1.0.2 source. A local compatibility fallback was added for the six instances whose EDIF `cellRef` has no `libraryRef`. The original EDIF and the user's uploaded converter source ZIP were not modified.
