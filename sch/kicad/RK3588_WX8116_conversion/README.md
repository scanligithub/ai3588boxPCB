# RK3588 + WX8116 OrCAD → KiCad conversion records

**Status: preliminary conversion only; electrical connectivity is not certified.**

This directory contains the conversion audit and diagnostic summary derived from the user's OrCAD Capture EDIF 2.0.0 export.

## Findings

- Source schematic: 34 pages; 1,914 unique component references.
- The local conversion generated a KiCad project, root sheet, 34 child sheets and a custom symbol library.
- Parsed component-reference sets, pin-number inventories, values and footprint fields matched between the source model and parsed KiCad model.
- These checks do not prove correct pin-to-net connectivity.
- Internal comparison between EDIF joined connections and independently inferred wire-geometry connectivity remains unresolved: 43 shared normalized net names have pin-set differences, 5 names appear only in the EDIF joined model, and 42 appear only in the geometry model.
- KiCad-native parser/ERC validation was not run because `kicad-cli` was not installed in the execution environment.

## Files in this commit

- `CONVERSION_AUDIT.md`: detailed counts, checks and remaining risks.
- `CONVERTER_PATCH_NOTE.txt`: input-specific compatibility fallback for six EDIF instances missing `libraryRef`.
- `connectivity_validation_summary.csv`: machine-readable validation counts and current status.

## Important packaging note

The generated KiCad project ZIP is **not included in this GitHub commit**. The archive was produced locally in the conversation session; the available GitHub connector supports text/blob writes but does not expose a local-binary upload operation. No placeholder is presented as if it were the project. The project archive remains available from the ChatGPT conversation attachment link while this session is active.

Do not use the preliminary conversion for engineering changes until the connectivity discrepancies are resolved and KiCad-native validation is run.
