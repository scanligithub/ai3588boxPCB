# Electrical connectivity acceptance result

**Model-level connectivity acceptance: PASS. Full engineering sign-off: PENDING KiCad-native ERC.**

| Check | Result |
|---|---:|
| Source pages / target sheet files | 34 / 35 (root + 34 child sheets) |
| Unique refs / component instances | 1,914 / 1,932; source-target matched |
| Actual target library pin coordinates checked | 7,370; 0 mismatches |
| Wire segments | 8,381 source and target; exact match |
| Junctions | 2,383 source and target; exact match |
| Power connection points | 947; exact coordinate/name match |
| Off-page connectors | 1,159; exact label/coordinate match |
| Connected joined-net groups | 1,426 / 1,426 remained one component |
| Open/split nets | 0 |
| Shorts joining different source nets | 0 |
| Labels crossing more than one source network | 0 |
| Custom target project parser issues | 0 |
| Converter tests | 26 passed |
| KiCad native ERC | Not run; `kicad-cli` unavailable |

There are 726 instance pins not present in the EDF explicit `joined` net records. The audit verified none touch a target wire or label, none is an unassigned hidden power pin, and none shares a target connected component with a pin that belongs to a source joined net. They remain isolated just as represented by the source EDF connection records. Two EDIF names (`HI3516_USB_DM`, `HI3516_USB_DP`) have no joined pin references and are name-only records.

Three labels `GPIO0/1/2` on page P07-CPLD were moved away from unrelated `CPLD_3V3` wire crossings. The updated label-crossing check reports zero labels anchored to more than one distinct source network.

**Limit:** These are source-EDF-to-generated-KiCad structural/geometry/graph comparisons, not a KiCad-native ERC. The project must still be opened in KiCad 10, run through ERC, and reviewed for intentional no-connect pins before final engineering sign-off. The full KiCad ZIP is not stored in this GitHub commit; the commit records text audit results only.
