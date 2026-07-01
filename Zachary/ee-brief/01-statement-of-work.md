# Statement of Work — Production PCB (Transcription + Layout)

**One 4-layer board · execution-only · fixed-price**

## What this project is

A consumer voice-assistant device, going to a 1,000-unit production run assembled from
JLCPCB turnkey PCBA (pre-assembled, pre-flashed). The system is **already designed and
validated**: it runs end-to-end on an ESP32-S3 devkit with breakout boards, every
production IC has been selected and verified in JLC/LCSC stock, and every GPIO
assignment is proven in working firmware.

**Your job is the remaining EE execution for the main board only:**

1. **Production schematic** — transcribe the provided, validated devkit-level schematic
   onto the bare production parts (component mapping table provided: devkit → module,
   breakout → IC, every part with an LCSC number), using each part's **public datasheet
   reference circuit**. No part research, no architecture decisions.
2. **4-layer SMT layout** — to the physical spec in `03-physical-spec.md` (target
   envelope 60 × 50 mm; may grow to 60 × 60 mm or a sideways-T, ≤60 × 80 mm bbox, if
   EMC/density needs it; defined connector zones + two-zone height limits; you place
   mounting holes freely).
3. **JLC-ready outputs** — Gerbers + BOM + CPL for direct JLCPCB upload.
4. **One revision** included after the client's prototype bring-up.

## What this project is NOT

- **No firmware** — none changes hands in either direction. The TAS5805M's I2C register
  init, OTA, audio pipeline, etc. are all client-side and happen after delivery. You
  never need to run, read, or write any code.
- **No part selection** — every IC, connector, and module is locked and listed with
  LCSC numbers in `02-technical-spec.md`. (You choose passive values per the reference
  circuits; prefer JLC Basic library.)
- **No daughterboards** — three small auxiliary boards exist in the product (controls,
  microphone, glow-LED); the client designs those. Your board just carries their JST
  headers.
- **No bring-up, validation, or test fixtures** — client's job, after boards arrive.
- **No enclosure work** — the client is the mechanical engineer with in-house printing.
  You get the envelope + a few placement zones; the enclosure adapts to your layout.
- FCC test logistics / SDoC / labeling are **client-side** — but the board **must be
  designed to pass FCC Part 15B** (see `02-technical-spec.md` §6). No production order management.

## Provided to you at kickoff (the complete package)

| File | Contents |
|---|---|
| `02-technical-spec.md` | Component mapping (prototype part → production part + LCSC #), full GPIO pin map, per-block circuit requirements, connector pinouts, production essentials, layout rules |
| `03-physical-spec.md` | Board envelope, mounting-hole policy, placement zones, height limits |
| `04-schematic/` | The validated devkit-level schematic (PDF, GPIO-annotated, + source) |
| `reference/` | All datasheets + the TI EVM user guide + Espressif hardware design guidelines (PDFs — nothing to hunt down) |

## Deliverables

| Deliverable | Format |
|---|---|
| Editable source project (KiCad or EasyEDA Pro) | full project, work-for-hire transfer |
| Schematic PDF | single reviewable PDF |
| Assembly drawing | PDF, top + bottom with refdes |
| BOM | CSV/XLSX: refdes, qty, MPN, LCSC #, Basic/Extended flag |
| CPL (centroid) | CSV, JLC format |
| Gerbers + drill | RS-274X + Excellon, JLC naming, upload-ready |
| Clean DRC/ERC confirmation | note in delivery email is fine |
| One post-prototype revision | updated files, included in fixed price |

## What this board involves (so the scope is clear)

- A 4-layer mixed-signal layout: ESP32-S3-WROOM module (**antenna keepout**), TAS5805M
  Class-D amp (**PowerPAD thermal + LC output filter, mono PBTL**), 2× TPS54331 bucks,
  2× logic level shifters, P-FET/PTC input protection, and JST headers.
- Tool: **KiCad or EasyEDA Pro** — whichever you work in; you deliver the editable source
  (work-for-hire) plus the JLC outputs above.

## Process & terms

- **Scope is locked and fixed-price** — this is a one-board transcription + layout +
  production-prep job; design, validation, and sourcing are already done. Hourly only if
  capped and agreed.
- **Timeline:** ~2–4 weeks to first Gerber drop. Revision turnaround 1–2 weeks after
  client bring-up.
- **Review:** one async schematic/layout review pass with the client before fabrication
  (marked-up PDF is fine).
- **Milestones / payment:** 25% kickoff / 50% Gerber drop / 25% revision sign-off.
- **IP:** work-for-hire; all deliverables fully assigned to the client. Portfolio
  use permitted.
