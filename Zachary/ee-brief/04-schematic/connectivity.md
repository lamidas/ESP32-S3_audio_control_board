# Devkit-level connectivity (the schematic you transcribe)

This is the **validated, as-built connectivity** for the main board — every MCU pin and
every off-board interface, GPIO-annotated. It is the contract document this folder
promised. It was confirmed end-to-end on the bench (ESP32-S3-DevKitC-1 + TAS5805MEVM +
breakouts), 2026-06.

**Files in this folder**
- `connectivity.svg` / `connectivity.pdf` / `connectivity.png` — the **visual** block/connectivity diagram (same content as below)
- `connectivity.dot` — Graphviz source (edit/re-render if needed: `dot -Tpdf connectivity.dot -o connectivity.pdf`)
- this file — the **net list** (copy net names from here)

**How to use it:** the net tables below are authoritative for *MCU pin usage + off-board
interfaces*. Everything marked **[EE adds]** (buck FB dividers, P-FET/PTC, level-shifter
support, all decoupling, the amp's PVDD/bootstrap/output-filter passives) you draw from
the **datasheet reference circuits** per `../02-technical-spec.md` §3 — those are not in
this connectivity view by design. If anything here disagrees with `02-technical-spec.md`
§2, **the pin map wins** (tell the client).

Refdes: **U1** = ESP32-S3-WROOM-1-N16R8, **U2** = TAS5805M, **U3a/U3b** = TPS54331 bucks,
**U5a/U5b** = SN74AHCT1G125 shifters, **Q1** = AO3401A, **F1** = PTC.

---

## 1. Power nets

| Net | Source | Loads | Notes |
|---|---|---|---|
| `12V` | J1 (barrel) | F1 → Q1 | center-positive |
| `12V_FUSED` | F1 (PTC) → Q1 (P-FET) | U2 PVDD, U3a IN, U3b IN | reverse-protected 12 V **[EE adds F1/Q1]** |
| `3V3` | U3a (TPS54331) | U1, U2 DVDD, I2C pull-ups, J3 pin1, J10 pin1 | **[EE adds buck]** |
| `5V` | U3b (TPS54331, Buck #2) | U5a/U5b VCC, J5 pin1, J9 pin1 | **[EE adds buck]** only 5V loads are LEDs+shifters; COB branch is high-current/animated (optional ferrite split) |

---

## 2. Audio amp — U2 TAS5805M (mono PBTL)

| MCU pin | Net | → U2 | Notes |
|---|---|---|---|
| GPIO6 | `I2S1_BCLK` | SCLK | no MCLK |
| GPIO7 | `I2S1_LRCLK` | LRCLK | |
| GPIO8 | `I2S1_DOUT` | SDIN | ESP out → amp in |
| GPIO47 | `I2C_SDA` | SDA | **external 4.7k pull-up to 3V3 [EE adds]** |
| GPIO48 | `I2C_SCL` | SCL | **external 4.7k pull-up to 3V3 [EE adds]** |
| GPIO21 | `AMP_PDN` | PDN | high = enable |
| GPIO14 | `AMP_FAULTZ` | FAULTZ | open-drain; S3 internal pull-up (no ext R) |
| — | `SPK+` / `SPK-` | OUT_A∥OUT_B → filter → J2 | **[EE adds]** PBTL parallel + LC filter (10µH+0.68µF); J2 = **4 Ω** speaker, ~10–15 W |
| — | DVDD=`3V3`, PVDD=`12V_FUSED`, ADR→DVDD (0x2C) | — | **[EE adds]** bulk/bootstrap/decoupling + PowerPAD vias |

## 3. Mic — J3 (PDM daughterboard connector; no mic on this board)

| MCU pin | Net | J3 pin | Notes |
|---|---|---|---|
| GPIO9 | `MIC_PDM_CLK` | 3 (CLK) | **33–100Ω series R at the header** (clock 1–4.8 MHz) |
| GPIO11 | `MIC_PDM_DATA` | 4 (DATA) | input |
| — | `3V3` | 1 | |
| — | `GND` | 2 | |

## 4. LEDs (two independent WS2812 data outputs, each via its own shifter gate)

| MCU pin | Net | Path | Connector |
|---|---|---|---|
| GPIO5 | `LED_COB` | → 100Ω → U5a (VCC=5V) → | J5 pin2 (COB strip) |
| GPIO38 | `LED_GLOW` | → ~120Ω → U5b (VCC=5V) → | J9 pin2 (glow daughterboard, 3× WS2812) |

U5a/U5b: SN74AHCT1G125, **OE tied to GND (always enabled, no GPIO)**.

## 5. Controls — J10 (daughterboard pigtail; pots/button off-board)

| MCU pin | Net | J10 pin | Notes |
|---|---|---|---|
| GPIO1 | `POT1_ADC` | 3 (POT1w) | ADC1_CH0; **[EE adds]** 1k + 100nF RC |
| GPIO2 | `POT2_ADC` | 4 (POT2w) | ADC1_CH1; **[EE adds]** 1k + 100nF RC |
| GPIO4 | `BUTTON` | 5 (BTN) | active-low, internal pull-up |
| — | `3V3` / `GND` | 1 / 2 | |

## 6. ESP32-S3 (U1) support — [EE adds all per Espressif guidelines]
Module decoupling; strapping (GPIO0 10k pull-up, GPIO3/45/46 per guide); USB GPIO19/20
(native, no connector — pogo only); **antenna keepout**; pogo-pad group (3V3, GND, EN,
IO0, TX/GPIO43, RX/GPIO44). **GPIO26–37 unusable (flash/PSRAM) — do not connect.**

---

## Connector pinouts (mirror of `02-technical-spec.md` §4)

| Ref | Pins | Pinout |
|---|---|---|
| J1 | barrel | +12V / GND |
| J2 | 2 | SPK+ / SPK− (**4 Ω** speaker) |
| J3 | 4 | 3V3 / GND / `MIC_PDM_CLK` / `MIC_PDM_DATA` |
| J5 | 3 | 5V / `LED_COB` / GND |
| J9 | 3 | 5V / `LED_GLOW` / GND |
| J10 | 5 | 3V3 / GND / `POT1_ADC` / `POT2_ADC` / `BUTTON` |

*(There is no J4/J6/J7/J8 — the controls were consolidated from old per-pot connectors
into the single 5-pin J10.)*
