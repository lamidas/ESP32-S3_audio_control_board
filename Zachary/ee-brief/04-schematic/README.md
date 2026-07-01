# Devkit-level schematic / connectivity (provided)

This folder is the **validated devkit-level schematic** the freelancer transcribes onto
production parts. It is provided as a **net list + rendered connectivity diagram** (a
transcription job needs copyable net names + an unambiguous picture more than it needs a
hand-drawn page):

| File | What it is |
|---|---|
| `connectivity.md` | **Authoritative net list** — every MCU pin + every off-board interface, GPIO-annotated, grouped by block. Copy net names from here. |
| `connectivity.svg` / `.pdf` / `.png` | The **visual** block/connectivity diagram (same content). |
| `connectivity.dot` | Graphviz source (re-render: `dot -Tpdf connectivity.dot -o connectivity.pdf`). |

## How to read it
- The net tables cover **MCU pin usage + off-board interfaces** only.
- Items tagged **[EE adds]** (bucks, P-FET/PTC, level-shifter support, all decoupling,
  the amp's PVDD/bootstrap/output-filter passives) are **yours to draw from the datasheet
  reference circuits** per `../02-technical-spec.md` §3 — they are intentionally not in
  the connectivity view.
- **Authority:** if anything disagrees with `../02-technical-spec.md` §2 (pin map), the
  pin map wins.
