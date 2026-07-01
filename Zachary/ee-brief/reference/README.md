# Reference documents

| File | What it's for |
|---|---|
| `tas5805m-datasheet.pdf` | Amp: pinout, typical application circuit (the schematic reference), PBTL config |
| `tas5805m-thermal-slaa880.pdf` | Amp: PowerPAD thermal layout guidance |
| `tps54331-datasheet.pdf` | Bucks ×2: reference design + layout |
| `esp32-s3-wroom-1-datasheet.pdf` | Module: pinout, electrical |
| `esp32-s3-hw-design-guidelines.pdf` | Module: decoupling, antenna keepout, strap pins |
| `sn74ahct1g125-datasheet.pdf` | LED data level shifter (U5 ×2) |
| `AO3401A-datasheet.pdf` | Q1 P-FET: SOT-23 pinout + ratings for the 12V reverse-polarity protection |
| `tas5805mevm-user-guide-slou510.pdf` | **EVM schematic** — supplements the datasheet's typical-application circuit for transcription (PVDD bulk, bootstrap, output filter, PDN/ADR/FAULTZ wiring). |

**Transcribe the bare chip, not the EVM's eval scaffolding.** The EVM adds an input mux /
I2S-SEL jumper and drives PDN/FAULTZ from an onboard I2C expander — those are evaluation
conveniences, **not** part of the production design. On the production board the amp's
**PDN and FAULTZ are the chip's own pins**, driven directly by the MCU (`GPIO21` / `GPIO14`
per `../02-technical-spec.md` §2). Use the EVM schematic only for the analog/power
reference (PVDD bulk + bootstrap caps + output filter + ADR strap).
