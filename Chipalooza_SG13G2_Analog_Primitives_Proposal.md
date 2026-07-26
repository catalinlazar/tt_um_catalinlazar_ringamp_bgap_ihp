# Chipalooza Challenge Proposal
## An SG13G2 Analog Primitives Library on a Single Tiny Tapeout Analog Tile

**Proposer:** Catalin Lazar
**Target PDK:** IHP SG13G2 (130 nm SiGe BiCMOS), via Tiny Tapeout
**Target vehicle:** One Tiny Tapeout analog tile (1x2, expandable to 2x2 if needed)
**Target supply:** ~1.0 V for all seven blocks
**Submission track:** Open Circuit Design / Chipalooza Challenge, Round 1 (IHP)
**Repository (in progress, behavioral models + ngspice sanity checks committed):** https://github.com/catalinlazar/tt_um_catalinlazar_ringamp_bgap_ihp

**On the July 27 deadline:** this document is intended as an early draft. Since Chipalooza is coordinated informally through the fossi-chat.org/#chipalooza channel and challenge meetings rather than a fully specified formal review process, the plan is to circulate this draft there ahead of July 27 and incorporate any feedback before final submission, using the 2-week grace period (to ~Aug 10) as a buffer if changes are needed. This isn't a documented formal step in the challenge announcement - it's a reasonable use of the community channel that's explicitly named as the way to stay engaged with the challenge. **Note: the public Chipalooza announcement does not specify an exact submission mechanism (no email address, form, or portal is given) - confirm the actual "how to send" step on fossi-chat.org/#chipalooza before the deadline.**

---

## 1. Motivation

Chipalooza's stated goal is a library of high-quality, reusable analog components for all three open PDKs, enabling cross-platform SoC porting. Rather than propose a single monolithic circuit, this proposal targets **seven small, independently characterizable primitives that share circuit DNA**, so the deliverable reads as a genuine *library* rather than a demo chip - while also being a deliberate, honest first step: this is the proposer's first custom analog layout in an open-source flow (25+ years of full-custom analog design in Cadence/Virtuoso, first time in Magic/KLayout/xschem/ngspice). The block selection and sequencing below is designed so that layout risk is front-loaded onto the easiest blocks, and schedule slip degrades gracefully rather than endangering the whole submission.

Four of these blocks are, not coincidentally, the exact building blocks of a cyclic/pipeline ADC (comparator -> quantizer element, ring-amp -> residue amplifier, bandgap -> reference generation). This proposal is explicitly framed as **Phase 1** of that longer-term goal: get each primitive proven and open-sourced individually first, then propose the composed ADC in a later Chipalooza round with real open-source layout mileage behind it. Three additional blocks (Section 3, Tier 2) extend this into an informal **residue-amplifier bake-off**: two more structurally distinct modern low-voltage OTA architectures alongside a conventional folded-cascode/telescopic backup, so the eventual ADC proposal has proven silicon options if the ring-amp underperforms.

## 2. Architecture Overview

- One **1x2 analog tile**, three analog pins used (`ua[0:2]`) of six available, three spare (`ua[3:5]`).
- A **3-bit digitally controlled analog mux** (`sel[2:0]`, driven from three `ui_in` pins - free digital resource) routes whichever of six blocks is under test onto a shared analog bus (`ua[0]` input, `ua[1]` output, `ua[2]` second input/reference), followed by a **single shared output buffer** before reaching `ua[1]`.
- **Mux never floats:** every one of the 8 possible select codes maps to a real, defined block; unused codes default to the HBT bandgap (Block 3), chosen specifically because it is the least sensitive of the mux-arbitrated blocks to an unknown/undefined external load (a simple resistor-set DC output, unlike the ring-amp/FIA/ZCBC's timing- or load-tuned behavior, or the SC bandgap's charge-domain sensitivity).
- **Shared output buffer, one instance:** isolates the sensitive block outputs from unpredictable off-chip test-setup loading (pad + ESD + cable + scope capacitance can be 10-20x the on-chip load these circuits are tuned for). A mode-select bit (`ua_raw_sel`) lets this same buffer instance's input be swapped between the mux output (normal) and a direct `ua_in` bypass (buffer-only characterization) - and separately, the same pad can show either the buffer's output or the bare mux output (raw debug mode), all without a second physical pad, which would otherwise permanently load the sensitive nodes with pad/ESD parasitics.
- **The comparator (Block 1) is not part of this mux at all.** Its input pair taps `ua[0]`/`ua[2]` directly and permanently, independent of `sel[2:0]`, and its digital decision goes straight to a dedicated digital pin (`uo[0]`) - it is always independently testable regardless of which other block is currently selected.

## 3. Block List (priority order = build/verification order)

### Tier 0 - Core deliverables (must-ship)

**Block 1: Dynamic latched comparator (StrongARM / double-tail latch)**
A clocked, regenerative comparator: cross-coupled inverter latch, input differential pair, tail switch. No bias currents to trim, no compensation network, minimal matching sensitivity beyond the input pair - the layout is close to a digital standard-cell exercise, making it the ideal first custom-layout block for the open-source flow. Directly reuses the proposer's SAR-quantizer design experience.
*Risk: low. Reuse: universal (SAR/Sigma-Delta/flash ADC quantizers, zero-crossing detectors). Target supply: ~1.0 V comfortably (dynamic, no static bias headroom to fight).*

**Block 2: Ring-amplifier / FEA-style OTA**
An open-loop or lightly-wrapped gain cell built from cascaded inverter-like stages - no Miller compensation, no heavy bias network. The only layout-sensitive element is the input differential pair (offset/matching); the rest is straightforward inverter-chain layout. This is a genuinely under-represented block in the current open-PDK ecosystem (Sky130/GF180 Chipalooza submissions to date lean on classical two-stage Miller OTAs), so it's a differentiated contribution.
*Risk: medium (one matched input pair; everything downstream is low-risk). Reuse: residue amplifiers, general-purpose gain stages, SC amplifier cores. Target supply: ~1.0 V comfortably - ring amps were introduced specifically to work well below the headroom classical two-stage/telescopic op-amps need (Hershberg et al., IEEE JSSC, Dec 2012).*

### Tier 1 - Stretch deliverables (builds on Tier 0)

**Block 3: SiGe HBT bandgap reference - current-summed, resistively-scaled architecture, ~1.0 V supply**
SG13G2's defining advantage over Sky130/GF180MCU is real vertical SiGe HBTs, and this block is built specifically to exploit that rather than port a CMOS-only approach. A naive Brokaw/Kuijk-style bandgap generates its output as VBE + K*deltaVBE in series, landing near the physical silicon bandgap voltage (~1.2 V) - a value that doesn't shrink with technology scaling, so a literal implementation needs roughly 1.3-1.5 V of supply headroom, incompatible with our 1.0 V target. Instead, this block sums the PTAT and CTAT branches as **currents** into a single output resistor (rather than stacking voltages in series), so the output level is set by a resistor ratio and can be placed anywhere below the ~1.2 V physical constant while every internal node stays within 1.0 V-friendly headroom. This is the same principle published in P. Malcovati, F. Maloberti, C. Fiocchi, U. Pruzzi, "Curvature-Compensated BiCMOS Bandgap with 1-V Supply Voltage," IEEE JSSC, July 2001 - adapted here to SG13G2's HBTs.
*Risk: low-medium. Reuse: any SoC needing a reference; a strong, distinctive IHP-only library entry. Target supply: ~1.0 V by design (the entire point of the current-summed architecture).*

**Novelty:** based on a search of the existing Tiny Tapeout / Chipalooza ecosystem, no IHP-shuttle Tiny Tapeout submission to date (TTIHP0p2, TTIHP25a, TTIHP25b, TTIHP26a) has used SG13G2's HBTs for a bandgap reference. The bandgap references that do exist in the TT ecosystem (e.g. `tt_um_bgr_agolmanesh` on TT08, and one design among TT07's analog entries) are Sky130-based and necessarily CMOS-only, since Sky130 has no dedicated BJTs. IHP's own direct-customer MPW tapeout stream separately includes a `bandgap_ref_cmos` project (IHP-GmbH/TO_Apr2025, testfield T586) - not a Tiny Tapeout submission, and notably CMOS rather than BJT-based, consistent with the headroom argument above. This block would be, as far as can be determined, the first HBT-based, sub-1.2V-headroom bandgap reference contributed to any open-PDK Tiny Tapeout/Chipalooza library.

**Block 4: Switched-capacitor bandgap ("end-of-life" variant)**
A resistor-free bandgap: PTAT/CTAT information is generated via capacitor ratios and correlated double sampling / chopping instead of poly resistors, using a design inspired by Block 2's amplifier core and Block 1's clock/chopping controller (implemented as dedicated instances, not shared physical hardware with those blocks - see the note under Block 5). Two things make this attractive for the library specifically: (a) it has no dependency on HBTs or precision resistors, so - unlike Block 3 - it is directly portable to Sky130 and GF180MCU, precisely the "cross-platform porting" goal the challenge calls out; and (b) it demonstrates the composability story end-to-end.
*Risk: medium. Target supply: ~1.0 V - since the output level here is set by capacitor ratio rather than a forced series voltage stack, it is not subject to the same ~1.2 V physical-constant constraint as Block 3's naive form.*

### Tier 2 - Residue-amplifier bake-off (stretch, builds on Tier 0/1)

**Block 5: Zero-Crossing-Based discharge amplifier (ZCBC)**
Instead of continuous-time feedback, the output node is discharged through a controlled current source; a comparator watches for the output crossing the target value and shuts the discharge current off at that instant - no static bias current, power spent only during the discharge transient. Sepke, Fiorenza, Sodini, Holloway, Lee, "Comparator-Based Switched-Capacitor Circuits for Scaled CMOS Technologies," IEEE JSSC, Dec 2006. Conceptually reuses the Block 1 comparator's design (instantiated as its own dedicated copy, not the shared physical Block 1 instance - preserving Block 1's independent testability) as the zero-crossing detector. Known non-ideality: finite comparator delay causes a systematic overshoot/gain error proportional to (discharge current x comparator delay); the standard mitigation is a two-phase coarse/fine discharge.
*Risk: medium (comparator-delay-induced gain error is a well-documented, well-mitigated non-ideality, not a novel risk). Target supply: ~1.0 V comfortably (dynamic, no static bias headroom fight).*

**Block 6: Floating Inverter Amplifier (FIA)**
A single CMOS inverter stage used as the gain element, with its "rails" being floating nodes pre-charged via capacitors during a reset/auto-zero phase, landing the inverter's bias point at peak gain regardless of input common-mode - class-AB-like push-pull efficiency without the ring-amp's multi-stage dead-zone tuning, and without a classical OTA's static tail current. (Citation not independently verified at proposal-draft stage - to be confirmed against IEEE Xplore before final submission.)
*Risk: medium (new self-biasing mechanism, comparable personal-risk profile to the ring-amp). Target supply: ~1.0 V comfortably.*

**Block 7: Classic OTA/op-amp (folded-cascode or telescopic as needed)**
Deliberately the least novel block on the tile: a conventional topology, included specifically as proven insurance for the eventual cyclic-ADC residue amplifier if none of Blocks 2/5/6 perform as hoped in silicon. At ~1.0 V, folded-cascode headroom is tight but well-documented (roughly one VDSsat per cascode device plus the input pair); a simpler telescopic or two-stage design may suffice and will be checked first. This is the proposer's strongest personal territory (25+ years with exactly this class of circuit), making it - despite being schematically conventional - one of the *lowest-risk* additions on the tile.
*Risk: low-medium (conventional topology, personally familiar non-idealities; main open question is whether cascoding is actually necessary at this supply). Target supply: ~1.0 V, the one block where this is a genuine design constraint rather than an architectural given.*

## 4. Area and Pin Budget (pre-layout estimate)

Rough device-count estimate across all seven blocks plus mux/buffer glue logic: **~150-180 active devices** plus bandgap resistors/HBTs and a handful of small capacitors - roughly double the 4-block estimate for this proposal's earlier draft, consistent with the block count increase. This is expected to fit within a 1x2 analog tile with adequate but not generous margin (matching-critical structures - differential input pairs, bandgap resistor/HBT arrays - typically need more area per device than raw transistor count suggests, due to guard rings and common-centroid layout); 2x2 remains the accepted fallback if real layout says otherwise. Pin budget is comfortable throughout: 3 of 6 analog pins used, 6 of 8 digital-in pins used, 1 of 8 digital-out pins used, 0 of 8 bidirectional pins used.

## 5. Delivery Plan, Risk Tiering, and Time Estimate

| Tier | Blocks | Fallback if schedule slips | Rough time (part-time pace) |
|---|---|---|---|
| 0 (must-ship) | Comparator, Ring-amp OTA | Ships regardless - simplest, first learning vehicles | ~5-7 weeks |
| 1 (stretch) | HBT bandgap, SC bandgap | Ships if Tier 0 lands with margin; otherwise documented as schematic/sim-validated only | ~5-7 weeks |
| 2 (stretch, bake-off) | ZCBC, FIA, Classic OTA | Ships if Tier 0+1 land with margin; otherwise deferred to a follow-up submission | ~6-8 weeks |

Plus a one-time **~1-3 weeks** ramping up on the open-source flow (xschem/ngspice/Magic/KLayout, following IHP's own IHP-AnalogAcademy tutorials) before any block-specific work begins. **Total, sequential, all tiers: roughly 16-22 weeks (~4-5.5 months)** of part-time (evenings/weekends) effort from end of tool ramp-up to a fully characterized, tapeout-ready set of all seven blocks - a rough planning estimate, not a committed schedule.

Even in the worst case, the tile ships two fully silicon-proven, independently useful library primitives (comparator + ring-amp OTA) - never zero. Only Tier 0 needs to be ready by whenever the next IHP shuttle actually opens (date not yet known); Tiers 1 and 2 are genuinely optional per round and roll forward to a later Chipalooza round if the shuttle date arrives first.

## 6. Deliverables (per block)

- Schematic + testbench (open-source: ngspice/Xyce via xschem; cross-checked against Cadence/Spectre where useful for sanity)
- Layout (Magic/KLayout), DRC/LVS/PEX clean against the SG13G2 open PDK
- Characterization plan: process corners + Monte Carlo (offset, PSRR/line regulation for the bandgaps, gain/BW for the OTAs, offset/kickback for the comparator, overshoot-vs-delay for the ZCBC)
- Public GitHub repository per block (schematics, layout, testbenches, README), consistent with the existing Efabless/Sky130 Chipalooza repository precedent
- Test/characterization hooks exposed through the shared analog mux plus the Tiny Tapeout demo board, so each block can be measured on real silicon post-tapeout

## 7. About the Proposer

25+ years of full-custom analog/mixed-signal IC design (Cadence Virtuoso/Spectre, 130-350 nm CMOS/BiCMOS), including 47 V high-voltage charge pumps with >40 dB PSRR, low-drift ring oscillators (first-time-right silicon, 6+ years in volume production), Ahuja-compensated low-noise amplifiers (13 uVrms), SAR-ADC quantizer redesign (50% power reduction), and current-steering DACs - all directly relevant to the blocks proposed here. Full ownership of tape-out flow including DRC/LVS/PEX/DFM in industrial (proprietary) tools. Three prior Tiny Tapeout submissions (Sky130 CPU, Sky130 all-digital PLL, IHP ring-oscillator array), but this is the first project using open-source **custom analog layout** tools - a deliberate, disclosed learning curve that this proposal's block sequencing is designed to absorb safely.

## 8. Longer-Term Roadmap (context, not part of this proposal)

Once these seven primitives are silicon-proven, the natural next Chipalooza submission is a cyclic/pipeline ADC assembled from them directly: Block 1 as the sub-ADC quantizer, one of Blocks 2/5/6/7 as the residue/gain amplifier (whichever performed best in silicon), Block 3 or 4 as the reference. This proposal is intentionally the risk-reducing first step toward that goal.
