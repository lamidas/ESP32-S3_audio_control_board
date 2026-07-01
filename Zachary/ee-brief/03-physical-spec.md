# Physical Spec — Board Envelope & Placement Zones

The enclosure geometry is now partly fixed: the envelope, height zones, and connector
zones below are real constraints, not loose hints. Optimize the internal layout within
them and report final positions back at delivery.

## Envelope

- Target **60mm (W) × 50mm (H)** rectangle. Coordinate convention: top-down view,
  **X = width 0(left)→60(right), Y = 0(bottom)→50(top)**.
- **Growth budget — if EMC/density needs it (raise at the M1 placement review, not after
  routing):**
  - **T0 (target):** 60 × 50 rectangle.
  - **T1:** 60 × 60 rectangle (grow H uniformly).
  - **T2:** **sideways-T** — the LEFT column (X 0–30, the power/audio cluster) extends to
    **80mm H**; the RIGHT column (X 30–60, ESP + antenna) stays **60mm H, vertically
    centered**. Bounding box 60 × 80 with two **30 × 10 notches** (top-right + bottom-right).
    Use only if the rectangle can't hold clean EMC spacing — the dense/hot parts live on
    the left, so the extra room is given there. Do not exceed the 60 × 80 bounding box.

```
        X=0      X=30     X=60
 Y=80  ┌────────┐                 left col +10 up
       │        │
 Y=70  │  LEFT  ├────────┐        right col top (notch above-right)
       │  col   │ RIGHT  │
       │ 30×80  │  col   │ ))) ANTENNA (right edge, Y 10–70)
       │ (pwr/  │ 30×60  │
       │ audio) │ (ESP)  │
 Y=10  │        ├────────┘        right col bottom (notch below-right)
       │        │
 Y=0   └────────┘                 left col +10 down
```

- Corner radius ≥2mm preferred (printed-bay friendly), not critical.

## Height (two zones)

- Interior ceiling: **LEFT half ~40mm, RIGHT half ~20mm**; minus a ~6mm board standoff →
  - **LEFT half parts ≤ ~34mm** (generous — put the PVDD electrolytic + tall parts here)
  - **RIGHT half parts ≤ ~14mm** (keep low — the flat WROOM module + low passives)
- **Bottom side ≤ 3mm** (board on ~6mm bosses; pogo pads + test points + low passives only).

## Mounting

- **4× M3 mounting holes**, ~one per quadrant, ≥3.5mm hole-center from any edge; coexist
  with edge connectors. Plated or unplated, your call; if plated, tie to GND.
- The client adapts the printed bosses to your final hole coordinates. Put the hole table
  in the assembly drawing.

## Placement (top-down, 60 W × 50 H)

```
 X=0                                    X=60
Y=50 ┌──[J1]─────[J5]───────[J10]──────┐   TOP
     │ jack(12V)  LEDstrip   controls   │
     │  bucks +amp +bulk  ┌──────────┐  │
[J2] │  (TALL parts,      │ ESP32-S3 │ ))) ANTENNA (right edge, MUST)
spk  │   left half)       │  module  │  │
     │                    └──────────┘  │
Y=0  └────[J9]──────────[J3?]──────────┘   BOTTOM
          glow         (EE's call)
```

- **Antenna (MUST):** WROOM at the **RIGHT edge**, antenna section overhanging outward,
  zero copper beneath, per the Espressif guidelines PDF. Module sits right-center; the
  keep-out consumes the right ~15–20mm — treat the right edge as reserved. State the edge
  used (expected: right).
- **Keep ALL noise** (buck switch nodes, Class-D outputs, LED data) away from the right
  (antenna) edge; don't run them on the outer layer near that edge.
- **Tall/hot cluster** (2 bucks + amp + PVDD bulk) → **LEFT half** (matches the 40mm height
  + isolates from the antenna).
- **J1 power jack:** TOP-LEFT, plug enters from the **TOP edge** — leave top-edge clearance
  for the plug body + a captive-cable service loop. PTC F1 + P-FET Q1 + buck inputs near J1.
- **J2 speaker:** LEFT edge (speaker is physically left of the board). Post-LC speaker leads
  tolerate some length; amp + LC filter placed by you — keep the pre-filter switching loop
  tight.
- **J5 COB LED strip:** TOP edge, biased **LEFT/center**, AWAY from the antenna corner
  (carries LED data).
- **J10 controls:** TOP edge, may sit further right (benign pot wipers + button).
- **J9 glow:** BOTTOM edge, left.
- **J3 mic:** YOUR call — the mic daughterboard is physically **LEFT-and-DOWN** on a ≤150mm
  harness, so place J3 wherever minimizes exposed PDM length. Priority: keep PDM CLK/DATA
  (harness AND on-board trace) away from buck switch nodes + the Class-D output; ground-guard
  the on-board portion; series R on CLK at the header (§3 technical spec). The unshielded
  harness is the bigger EMI antenna — bias toward the shorter harness.
- **LED data** (GPIO5/38 → shifters → J5/J9) and **I2S/I2C** (ESP↔amp): short; route away
  from the antenna edge and the mic PDM lines.

## Always-present (place even though not in the connector map)

- **Bottom-side pogo programming pad group** (3V3/GND/EN/IO0/TX43/RX44), 2.54mm pitch,
  reachable by a bed-of-nails fixture.
- **Bottom test points:** 12V_FUSED, 3V3, 5V, I2S1_BCLK, MIC_PDM_DATA, LED_COB, EN, GND.
- **3+ fiducials.** No user-accessible programming/USB connector.

## Report-backs at delivery (so the enclosure can adapt)

1. Final outline W × H + mounting-hole coordinate table
2. Antenna edge used (expected: right)
3. Connector positions (from the assembly drawing is fine)
4. Final envelope tier used (T0 60×50 / T1 60×60 / T2 sideways-T) + why, if beyond T0 —
   should have been flagged at M1
