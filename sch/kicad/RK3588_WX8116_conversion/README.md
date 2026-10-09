# RK3588 + WX8116 OrCAD → KiCad conversion

**Model-level electrical connectivity checks: PASS. KiCad-native ERC: pending.**

This directory contains the conversion audit and validation evidence for the RK3588/WX8116 schematic.

## Validation summary
- 34 source schematic pages; 35 KiCad sheet files (root + 34 child pages).
- 1,914 unique component references and 1,932 placed instances matched.
- All 7,370 pin connection points matched the source coordinates.
- All 8,381 wire segments, 2,383 junctions, 947 power connection points and 1,159 off-page connector anchors/names matched the source model.
- 1,426 connected source net groups remain single connected components in the generated KiCad schematic graph.
- Model-level check reports 0 open/split source nets, 0 shorts joining distinct source nets, and 0 cross-net label-anchor collisions.
- Converter tests: 26 passed; target project parser issues: 0.
- KiCad native ERC has not run because `kicad-cli` was unavailable in the build environment.

See `ACCEPTANCE_REPORT.md`, `CONVERSION_AUDIT.md`, and `connectivity_validation_summary.csv`.

## Binary package status
The full KiCad archive is `RK3588_WX8116_KiCad_electrical_validation_v2.zip`, size 1,127,617 bytes, SHA-256:
`312ac11520f21fbea18b8a714a8a200f7473fa00603b8c4ef138ff10f58be65c`.

**That ZIP is not stored in this repository.** The active GitHub connector permits text/blob commits but does not provide a binary upload path from the local workspace. This repository commit records validation results and the exact package checksum, not the project archive itself. The archive is attached in the ChatGPT conversation and should be uploaded through GitHub's web UI or a locally authenticated Git client to persist it in the repo.

Do not use this project for engineering changes until KiCad 10 native ERC and review of intentional no-connect pins are completed.
