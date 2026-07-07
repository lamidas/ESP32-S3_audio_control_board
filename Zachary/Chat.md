Hi,
I'm a PCB designer, and I can help you with your project.
I was TOP RATED PLUS on Upwork 2 years ago, but I lost the badge because of my 2-years-personal project.
I'm looking forward to your reply.
Best regards,
Derek.

View details
Zachary Berkson
12:36 AM
Hi Derek — I'd like to move forward with you on this board. I've attached the full brief.
Before we kick off, a few quick confirmations and one short exercise:

1. Tool & files. Which tool will you use for this board? (EasyEDA Pro or KiCad both fine.)
   Confirm I receive the editable source files (mine to own) plus JLCPCB Gerbers, BOM, and CPL.
2. Audio. Have you laid out a Class-D audio amp or an I2S DAC/codec output stage before?
   If you have any board example (even redacted), I'd like to see it.
3. Two boards you personally laid out — ideally one dense, multi-connector board and one with a wireless module or switching regulators — with your exact role on each. (Images fine.)
4. Terms. Scope is main board only (transcribe my bench-validated schematic + lay it out, no firmware); you work from my pin map, connectivity net list, and datasheets. All parts stay JLCPCB-turnkey / LCSC-stocked (no consignment) — you flag anything that isn't. Fixed price + timeline with milestones (schematic → placement → final Gerbers) and one revision. You mentioned ~$5k / ~4 weeks — please confirm.
5. Quick exercise (in the attached brief): reply with the 3 most important things you'd question, change, or flag in THIS net list/spec before laying it out. Not a trick — I just want to see how you approach this specific board.

The board is built around an `ESP32-S3-WROOM` module + a `TI TAS5805M` Class-D amp (`mono PBTL into 4Ω`), a `single 5V rail for LED`s, and an `LC output filter on the amp`. The PDM microphone sits on a separate daughterboard, but its CLK/DATA lines come back to this board on a JST header (J3) and route near the Class-D — so it still drives layout/EMC decisions. FCC: module pre-certified;
whole board must pass Part 15B.

ee-brief.zip
13 MB

Derek Francavilla
2:13 AM
Hi,
Thanks for the detailed information, this is a very clear production-style board. I'll answer the questions.

1. I can work in both of KiCad and EasyEDA you prefered. for this project I'll use KiCad, and you'll receive all the files you want.
2. Yes. I have laid out Class-D & mixed-signal audio systems, including esp32, multi mics, analog/digital partitioned audio, I2S DAC/codec output.But I'm afraid of can't show you because of the NDA. anyway I'm sure my experience will help you in this project.
3. I'll add my recent project and you can find them in my portfolios.
4. Confirmed and aligned.
   Schematic and PCB only,
   no firmware
   Gerber and BOM with outstock flag
   3 ~ 4 weeks is realistic, weekly revision
   5K budget
5. the exercise

1) Class-D output stage vs EMC / FCC Part 15B risk
   output switching loop area, LC filter placement vs connector distance, speaker trace routing acting as antenna, return current containment through ground structure should be tightly controlled, if not, FCC fail
2) PDM mic CLK/DATA routing near Class-D domain (J3)
   this is a high-risk coupling and needs strict zoning.
   PDM should stay in RF quiet region. and cross Class-D area only with tuned impedance and orthogonal crossing(if unavoidable)
3) Ground / return path partitioning strategy
   for this I have to confirm the followings : single solid GND plane strategy vs split zone, star-point existing position

overall this is a solid architecture and very buildable, but EMC-sensitive rather than logic complex

Best Regards.
Derek

0617_ESP32-S3 Audio Processing and Control.png
ZB

Zachary Berkson
12:20 PM
Thanks Derek. Let's proceed in KiCad. Your read is spot-on; EMC is exactly the risk, and the PDM/Class-D zoning + return-current points are what I care most about.

Two confirmations:

- Deliverables: Gerbers + BOM (out-of-stock flags) + CPL/centroid (for JLC turnkey assembly) + editable KiCad source.
- Revision: the fixed $5k includes one revision round after my prototype bring-up (weekly check-ins during the work are welcome).

On your ground question: my intent is EMC-first — a solid L2 reference plane (avoid split-plane slot-antenna returns), with the high-current Class-D power return handled by a star/partitioned point at the amp, and the noisy (bucks/Class-D) vs quiet (mic/antenna) domains separated by placement rather than a plane cut. That's the direction — but you're the EMC specialist, so I defer to your judgement.

Once you confirm those two, I'll set up the contract with milestones — Milestone 1 = production schematic + component placement to kick off.

Derek Francavilla
1:23 PM
Hi Zachary
Thank you. this is clear and I'm aligned with your approach.

I confirm the followings.
Deliverables: KiCad source + production schematic/PCB + Gerbers + BOM (with LCSC/out-of-stock flags) + CPL/centroid for JLC turnkey assembly
Revision: included one post-layout revision cycle within the fixed $5K scope, based on feedback from your prototype bring-up and test results

Your EMC intent is the correct direction. and I agree with the followings
-Solid L2 reference plane (no split-plane return interruption / no slot antenna risk)
-Star/partition strategy localized around the Class-D power stage
-Domain separation by placement rather than plane cutting

I would just add one refinement for robustness.
the star point must be implemented carefully so it does not increase return path impedance for Class-D switching currents. I'll insure low inductance return continuity while still isolating noisy and sensitive domains(especially mic and RF section)
Once you're ready, we can proceed with milestone 1 milestone 1 is for the production schematic validation , initial placement, and EMC zoning plan.
looking forward to it.
Best, Derek.
please share your feedback

ZB
Zachary Berkson
2:24 PM
Derek — this all looks right, and I appreciate the refinement.
Let's proceed — I'm setting up the contract and funding Milestone 1 (schematic validation + initial placement + EMC zoning plan) now.
A few last minute changes from the brief I sent, before we begin:
Power input:

1. Drop Q1 (AO3401A reverse-polarity P-FET) — the 12 V supply is captive (no user-accessible jack; polarity is fixed and verified at assembly), so the input becomes 12 V → PTC (F1) → 12V_FUSED.
2. Add a TVS on 12V_FUSED → GND (unidirectional, ~13–15 V standoff, clamp < 28 V, JLC-stocked) to protect the bucks + amp PVDD from cable-borne transients/ESD.
3. Input EMI filtering (ferrite + cap on the 12 V in) — your call as part of the EMC plan.
   Module:
4. Include the WROOM EN (CHIP_PU) pull-up + ~1 µF power-on-reset RC per the Espressif guidelines (the devkit board provided this; the bare module needs it).

I will send an updated brief for your convenience in a moment.
Looking forward to working with you!
Best,
Zach

ZB
Zachary Berkson sent an offer

2:34 PM
I need an experienced contractor to take a validated design + complete brief and produce the production schematic + 4‑layer layout. The design is fully bench‑validated; you receive a complete brief (pin map, per‑block requirements, connectivity diagram, all datasheets). This is transcription + layout + production‑prep, not architecture. One revision included. The design includes an ESP32-S3 and TI Class-D audio components. The project requires expertise in PCB design and layout, particularly with these components. The final output should be ready for JLCPCB turnkey production and assembly. Must design for FCC Part 15B; ESP32 antenna‑keepout and Class‑D thermal/EMC experience required.

Est. Budget: $5,000.00

Milestone 1: Schematic + placement + EMC zoning: Production schematic validated against the provided connectivity net list + initial component placement + EMC zoning plan. Accept when: schematic net-matches connectivity.md; placement respects WROOM antenna keepout + LC filter at the amp; zoning plan addresses Class-D / PDM / buck separation.

Project funds: $1,250.00

View offer
2 files
ee-brief 2.zip
13 MB
Derek Francavilla accepted an offer

2:37 PM
View contract
Derek Francavilla
2:47 PM
Hi Zachary.
thank you for your offer.
I'll do my best.

Best regards
Derek.
