# Technical Spec — Component Mapping, Pin Map, Block Requirements

This is the authoritative parts + connectivity spec. The schematic in `04-schematic/`
shows the validated system on devkit/breakout hardware; **this document maps every
prototype part to its production part** and states each block's circuit requirements.
All reference PDFs are in `reference/`.

---

## 1. Component mapping (prototype → production)

| # | On the provided schematic (prototype) | Production part | LCSC # | Package | Reference circuit source |
|---|---|---|---|---|---|
| U1 | ESP32-S3 DevKitC-1 (devkit) | **ESP32-S3-WROOM-1-N16R8** (bare module) | C2913202 | SMD 25.5×18mm | `esp32-s3-wroom-1-datasheet.pdf` + `esp32-s3-hw-design-guidelines.pdf` (decoupling, antenna keepout, strap pins) |
| U2 | I2S amp breakout / EVM | **TAS5805MPWPR** — I2S in, integrated DAC + DSP, **mono PBTL** | C478472 | HTSSOP-28 PowerPAD | `tas5805m-datasheet.pdf` + **`tas5805m-evm-user-guide.pdf` (transcribe the EVM reference schematic)** + `tas5805m-thermal-slaa880.pdf` (PowerPAD layout) |
| U3 (×2) | Bench supply / generic buck modules | **TPS54331DR** ×2 — buck #1: 12V→3.3V, buck #2: 12V→5V (FB divider sets each output) | C9865 | SOIC-8 | `tps54331-datasheet.pdf` (datasheet reference design; one part number, two instances) |
| — | SPH0645 / INMP441 mic breakout | **Not on your board** — the microphone (PDM MSM261DDB019) lives on a client-designed daughterboard, gasket-mounted to the enclosure wall. You only place its 4-pin JST header (J3, §4) |  — | — | — |
| U5 (×2) | Adafruit 74AHCT125 breakout | **SN74AHCT1G125DCKR** level shifter ×2 (3.3→5V) — one gate for the COB ring data (GPIO5), one for the glow WS2812 data (GPIO38) | C350557 | SC-70-5 | `sn74ahct1g125-datasheet.pdf` |
| Q1 | — (new for production) | **AO3401A** P-FET, reverse-polarity protection on 12V input | C15127 | SOT-23 | standard P-FET reverse-protection topology (JLC **Basic** part) |
| F1 | — (new for production) | **SMD1812P300SLR/24** PTC, 3A hold | C2892380 | 1812 | series element on 12V input, before the P-FET |
| J1 | Bench 12V input | **PJ-102AH** barrel jack, 2.1×5.5mm center-positive | C3096093 | THT | direct 12V input; cable is captive inside the enclosure (jack is not load-bearing) |
| — | Generic RGB LED | **Not on your board** — the bust glow is **3× addressable WS2812B** on a client glow daughterboard. You place its **3-pin** JST header (J9, §4) + a 2nd shifter gate; pixels + caps are on that board |
| J2–J9 | Jumper wires | **JST PH 2.0mm headers**: B2B-PH-K-S (C131337), B3B-PH-K-S (C131339), B4B-PH-K-S (C131334) | see §4 | THT vertical | connector table in §4 |
| — | Pots / button on breadboard | **Not on your board** — they live on a client-designed daughterboard; you only place its JST pigtail header (§4, J10) | — | — | — |

Passives (decoupling, FB dividers, buck L/C, filters, pull-ups, TVS): your values per
the reference circuits. Prefer JLC **Basic** library parts.

---

## 2. Pin map (validated in firmware — do not reassign)

ESP32-S3-WROOM-1-**N16R8** (Octal PSRAM: **GPIO 26–32 and 33–37 are internally used —
do not connect**).

| GPIO | Net / function | Dir | Notes |
|---|---|---|---|
| 0 | Boot strap | — | 10kΩ pull-up; pogo-pad access (download mode) |
| 1 | POT1_ADC (ADC1_CH0) | In | from J10 pigtail, via 1kΩ + 100nF RC |
| 2 | POT2_ADC (ADC1_CH1) | In | from J10 pigtail, via 1kΩ + 100nF RC |
| 3 | JTAG strap | — | float |
| 4 | BUTTON | In | from J10 pigtail; active-low, internal pull-up (optional 100nF to GND) |
| 5 | LED_DATA (RMT) | Out | → 100Ω series R → U5 level shifter → J5 |
| 6 | **I2S1**_BCLK | Out | → TAS5805M SCLK (no MCLK used) — amp is **I2S1** (see note) |
| 7 | **I2S1**_LRCLK | Out | → TAS5805M LRCLK |
| 8 | **I2S1**_DOUT | Out | → TAS5805M SDIN |
| 9 | MIC_PDM_CLK | Out | → mic CLK — mic is **I2S0** (PDM is I2S0-only on the S3) |
| 10 | spare | — | (freed — PDM needs no word-select) |
| 11 | MIC_PDM_DATA | In | ← mic DATA (I2S0 PDM RX) |
| 12, 13, 15–18, 41, 42 | spare | — | route to test pads if convenient |
| 14 | AMP_FAULTZ | In | ← TAS5805M FAULTZ (open-drain; S3 internal pull-up — no external R) |
| 19 / 20 | USB D− / D+ | — | **reserved, native USB-Serial-JTAG — do not repurpose** |
| 21 | AMP_PDN | Out | → TAS5805M PDN (high = enabled) |
| 38 | Glow WS2812 data (RMT) | Out | → 2nd U5 shifter gate → ~100-150Ω → J9 (glow daughterboard, 3× WS2812) |
| 39 / 40 | spare (freed) | — | were glow R/G/B PWM; freed by the addressable switch |
| 43 / 44 | UART0 TX / RX | — | pogo pads (factory flash) |
| 45 | VDD_SPI strap | — | do not connect |
| 46 | Boot-log strap | — | float or 10kΩ pull-down |
| 47 | I2C_SDA | I/O | → TAS5805M SDA; **external 4.7kΩ pull-up to 3V3 required** |
| 48 | I2C_SCL | Out | → TAS5805M SCL; **external 4.7kΩ pull-up to 3V3 required** (2.2k if 400kHz) |

> **I²S controller note (firmware, no PCB impact):** the ESP32-S3 supports **PDM on
> I²S0 only**, so the **mic uses I²S0** and the **amp uses I²S1**. GPIO routing is
> unaffected. Bench-validated 2026-06.

---

## 3. Per-block requirements

### Power input
`J1 (12V) → F1 (PTC 3A) → Q1 (AO3401A reverse-polarity P-FET) → 12V_FUSED`.
12V_FUSED feeds: TAS5805M PVDD (directly) + both bucks.

### Bucks (2× TPS54331DR)
- **Buck #1 → 3V3**: ESP32-S3, TAS5805M DVDD, I2C pull-ups, mic daughterboard (via J3).
- **Buck #2 → 5V** (single output): the **only** 5V loads are LEDs + their shifters —
  COB strip (J5), glow daughterboard (J9), and both SN74AHCT1G125 VCC (U5a/U5b). Nothing
  audio or logic runs on 5V. The COB strip is the high-current/animated branch — you may
  optionally split it from the buck output with a **ferrite bead + local bulk** to contain
  switching noise; otherwise it is one rail.
- Keep the two switch-node loops minimal and **physically separated**; keep the 5V LED
  branch and its ground return **away from the amp input and the mic (J3) lines**. Follow
  the TPS54331 datasheet reference layout for both instances.

### Audio amp (TAS5805M, mono PBTL)
- Transcribe the EVM reference: PVDD = 12V_FUSED with bulk (~390µF electro + 22µF
  ceramic) close to the part; DVDD = 3V3; **PBTL** output configuration (OUT_A ∥ OUT_B)
  → **LC output filter (10µH + 0.68µF — not a bare ferrite-bead/filterless output)** for
  EMC and to keep Class-D switching noise off the nearby **PDM mic (J3)** → J2 speaker
  (**4 Ω**, ~10–15 W PBTL @12V).
- ADR address resistor to DVDD (e.g. 4.7k → addr 0x58). PDN ← GPIO21, FAULTZ → GPIO14.
- PowerPAD: generous ground pour + thermal via array per `tas5805m-thermal-slaa880.pdf`.
- Register init is client firmware — nothing for you here beyond the wiring.

### Mic interface (J3 — daughterboard connector; no mic on your board)
The PDM microphone lives on a client daughterboard. Your board carries **J3** (4-pin
JST PH): 3V3 / GND / MIC_PDM_CLK (GPIO9) / MIC_PDM_DATA (GPIO11). Place a **33–100Ω
series resistor on CLK at the header** (PDM clock runs 1–4.8MHz over a ≤150mm
harness). TVS on CLK/DATA like the other cabled signals. (PDM decode is client
firmware — not your concern.)

### LED data path
GPIO5 → 100Ω → U5 (SN74AHCT1G125, VCC = 5V) → J5 DATA. Keep short.

### Bust glow (J9 — daughterboard connector; no LEDs on your board)
The glow up-light is **3× addressable WS2812** on a client daughterboard (placed under
the bust glow zone). Your board carries **J9** (3-pin JST): 5V / DATA / GND, where DATA
is **GPIO38** (own RMT channel) through a **2nd 74AHCT1G125 gate** (3.3→5V) + a
~100-150Ω series R. The 3 WS2812 pixels + their decoupling caps are on the
daughterboard — your board just routes 5V/GND + the level-shifted data line to J9.
(GPIO39/40 freed.)

### Production essentials checklist
- Per-datasheet decoupling on the module (don't shortcut)
- **Antenna keepout** per Espressif guidelines: module at board edge, antenna section
  overhanging/outward, zero copper under the antenna
- TVS/ESD on externally-cabled signals: LED data, button, pot wipers, speaker, mic CLK/DATA
- **Pogo-pad group** (bottom, 2.54mm pitch, no headers): 3V3, GND, EN, IO0, TX(43), RX(44)
- **Test points** (bottom pads, silkscreened): 12V_FUSED, 3V3, 5V, I2S1_BCLK (amp),
  MIC_PDM_DATA, LED_COB, EN, GND
- 3+ fiducials for pick-and-place
- **No user-accessible programming ports** (no USB connector, no headers)

---

## 4. Connector table (JST PH 2.0mm, THT vertical; J5 part TBD per note)

| Ref | Function | Pins | Part / LCSC | Pinout |
|---|---|---|---|---|
| J1 | 12V in (barrel) | — | PJ-102AH / C3096093 | center +12V, sleeve GND |
| J2 | Speaker | 2 | B2B / C131337 | SPK+ / SPK− (PBTL output, post-filter; **4 Ω** load) |
| J3 | **Mic daughterboard** | 4 | B4B / C131334 | 3V3 / GND / CLK / DATA — PDM; series R on CLK at the header (§3) |
| J5 | COB LED strip | 3 | B3B / C131339 | 5V / DATA / GND — **see note** (pin order matches the BTF strip lead) |
| J9 | **Glow daughterboard** | 3 | B3B / C131339 | 5V / DATA / GND (3× WS2812 chain; data = GPIO38 level-shifted, ~100-150Ω series R on this board) |
| J10 | **Controls daughterboard** | 5 | B5B-PH (same family) | 3V3 / GND / POT1w / POT2w / BUTTON (single pigtail) |

Headers grouped along edges with ≥8mm mating clearance (zones in `03-physical-spec.md`).

> **J5 (COB LED) — connector TBD (bounded):** 3-pin, 5V/DATA/GND. Baseline **JST PH 2.0mm
> (B3B-PH-K-S, C131339)**; **may change to JST XH 2.5mm 3-pin** pending the strip vendor's
> peak-current confirmation — treat as a possible late **drop-in swap** (both are small
> wire-to-board JST headers in the same edge zone). Everything else about J5 is fixed. We'll
> confirm the exact part before routing/BOM freeze.

---

## 5. Layout rules

- **Stackup:** 4-layer JLC standard 1.6mm — L1 signal/components, L2 **solid GND (do
  not split)**, L3 power (12V region / 3V3 region), L4 signal
- Audio return currents: no high-current returns through signal ground; star or
  partitioned grounding at the amp
- Audio digital runs short (<50mm): amp I2S (GPIO6/7/8); mic PDM lines (GPIO9/11) short to J3
- I2C with its pull-ups near the amp
- Buck inductors/loops away from the amp input side and J3 (mic lines)
- DRC: JLC 4-layer standard rules; ERC clean

---

## 6. EMC (FCC Part 15B) — a design target

The assembled product is sold in the US and must pass **FCC Part 15 Subpart B**
radiated-emissions limits. Treat EMC as a **first-class layout objective, not final
polish** — the two switching bucks and the Class-D amp are the emission sources, and
**layout decides pass/fail**:

- **Switching loops:** keep the buck switch-node loops and the Class-D pre-filter output
  loops tight and small; follow the TPS54331 and TAS5805M reference layouts.
- **Output filter:** keep the amp's **LC filter (10µH + 0.68µF)** — do not ship a bare
  ferrite-bead/filterless output.
- **Ground:** uninterrupted L2 ground plane (no splits under switching or audio); short
  return paths.
- **Isolate the noise:** route buck switch nodes, Class-D outputs, and LED data **away
  from the WROOM antenna edge and the mic (J3) lines**; don't run them on the outer layer
  near the board edge.
- **Provision for mitigation:** leave room/footprint for a **shield can** (or guard
  fence) over the amp/buck area, in case a pre-compliance scan needs it.

(FCC test booking, the SDoC, and labeling are client-side — your deliverable is a board
*designed to pass*.)
