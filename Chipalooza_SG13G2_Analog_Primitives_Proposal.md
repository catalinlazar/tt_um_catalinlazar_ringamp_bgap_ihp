# Chipalooza Challenge Proposal
## An SG13G2 Analog Primitives Library on a Single Tiny Tapeout Analog Tile

**Proposer:** Catalin Lazar
**Target PDK:** IHP SG13G2 (130 nm SiGe BiCMOS), via Tiny Tapeout
**Target vehicle:** One Tiny Tapeout analog tile (1×2, expandable to 2×2 if needed)
**Target supply:** ~1.0 V for all four blocks (see Section 3 for the bandgap architecture implication)
**Submission track:** Open Circuit Design / Chipalooza Challenge, Round 1 (IHP)
**Repository (in progress, behavioral models + ngspice sanity checks committed):** https://github.com/catalinlazar/tt_um_catalinlazar_ringamp_bgap_ihp

**On the July 27 deadline:** this document is intended as an early draft. Since Chipalooza is coordinated informally through the fossi-chat.org/#chipalooza channel and challenge meetings rather than a fully specified formal review process, the plan is to circulate this draft there ahead of July 27 and incorporate any feedback before final submission, using the 2-week grace period (to ~Aug 10) as a buffer if changes are needed. This isn't a documented formal step in the challenge announcement — it's a reasonable use of the community channel that's explicitly named as the way to stay engaged with the challenge. **Note: the public Chipalooza announcement does not specify an exact submission mechanism (no email address, form, or portal is given) — confirm the actual "how to send" step on fossi-chat.org/#chipalooza before the deadline.**

---

## 2. Current Architecture Status (as of this draft)

Beyond the four blocks below, the design now includes two infrastructure details worth noting for reviewers:
- **Shared output buffer:** all three mux-arbitrated blocks (ring-amp, HBT bandgap, SC bandgap) pass through a single shared output buffer before reaching the tile's analog output pin, isolating them from unpredictable off-chip test-setup loading (particularly important for the ring-amp's tuned settling behavior). A mode-select bit lets this same buffer instance be bypassed for direct characterization, or the raw block output viewed on the same pin without a second physical pad (avoiding permanent extra parasitic loading on the sensitive ring-amp node).
- **Default/reset mux state:** the block-select mux defaults to the HBT bandgap at power-on (rather than an undefined/floating bus state), chosen specifically because it is the least sensitive of the three mux-arbitrated blocks to an unknown external load.
- The comparator (Block 1 below) is not part of this mux at all — its digital output is wired directly to its own pin, independent of block selection.

A behavioral (Verilog-A) model of the full architecture has been built and sanity-checked in ngspice (temperature sweep, AC response, decision transfer curve, mux/buffer routing) — see the repository for details.

---

## 1. Motivation

Chipalooza's stated goal is a library of high-quality, reusable analog components for all three open PDKs, enabling cross-platform SoC porting. Rather than propose a single monolithic circuit, this proposal targets **four small, independently characterizable primitives that share circuit DNA**, so the deliverable reads as a genuine *library* rather than a demo chip — while also being a deliberate, honest first step: this is the proposer's first custom analog layout in an open-source flow (25+ years of full-custom analog design in Cadence/Virtuoso, first time in Magic/KLayout/xschem/ngspice). The block selection and sequencing below is designed so that layout risk is front-loaded onto the easiest blocks, and schedule slip degrades gracefully rather than endangering the whole submission.

These four blocks are also, not coincidentally, the exact building blocks of a cyclic/pipeline ADC (comparator → quantizer element, ring-amp → residue amplifier, bandgap → reference generation). This proposal is explicitly framed as **Phase 1** of that longer-term goal: get each primitive proven and open-sourced individually first, then propose the composed ADC in a later Chipalooza round with real open-source layout mileage behind it.

## 2. Platform & Tile Plan

- One **1×2 analog tile** on the IHP SG13G2 Tiny Tapeout shuttle (2×2 fallback if floorplan needs it).
- All four blocks are small enough (a handful of active devices each) to coexist in this footprint; the constraint is analog pin budget (6 `ua[]` pins max), not area.
- **Pin-sharing strategy:** a digitally-controlled analog mux (CMOS transmission gates, driven from `ui_in` select bits — free digital pins) routes whichever block is under test to a small shared analog I/O bus. Budgeted as:
  - `ua[0]` — shared analog input / stimulus
  - `ua[1]` — shared analog output
  - `ua[2]` — shared second reference/input (needed by the comparator and both bandgaps)
  - `ua[3–5]` — reserved for direct test-point access to the two "stretch" blocks if pin budget allows, so they can still be characterized standalone even while sharing the mux.
- Mode/status reporting and mux control uses `ui_in`/`uio` — no additional analog pins required.

## 3. Block List (priority order = build/verification order)

### Tier 0 — Core deliverables (must-ship)

**Block 1: Dynamic latched comparator (StrongARM / double-tail latch)**
A clocked, regenerative comparator: cross-coupled inverter latch, input differential pair, tail switch. No bias currents to trim, no compensation network, minimal matching sensitivity beyond the input pair — the layout is close to a digital standard-cell exercise, making it the ideal first custom-layout block for the open-source flow. Directly reuses the proposer's SAR-quantizer design experience. This block is also the internal clock-phase/chopping element for Block 4.
*Risk: low. Reuse: universal (SAR/ΣΔ/flash ADC quantizers, zero-crossing detectors). Target supply: ~1.0 V comfortably (dynamic, no static bias headroom to fight).*

**Block 2: Ring-amplifier / FEA-style OTA**
An open-loop or lightly-wrapped gain cell built from cascaded inverter-like stages — no Miller compensation, no heavy bias network. The only layout-sensitive element is the input differential pair (offset/matching); the rest is straightforward inverter-chain layout. This is a genuinely under-represented block in the current open-PDK ecosystem (Sky130/GF180 Chipalooza submissions to date lean on classical two-stage Miller OTAs), so it's a differentiated contribution. It also serves as the OTA core for Block 4.
*Risk: medium (one matched input pair; everything downstream is low-risk). Reuse: residue amplifiers, general-purpose gain stages, SC amplifier cores. Target supply: ~1.0 V comfortably — ring amps were introduced specifically to work well below the headroom classical two-stage/telescopic op-amps need (Hershberg et al., IEEE JSSC, Dec 2012).*

### Tier 1 — Stretch deliverables (built from Tier 0)

**Block 3: SiGe HBT bandgap reference — current-summed, resistively-scaled architecture, ~1.0 V supply**
SG13G2's defining advantage over Sky130/GF180MCU is real vertical SiGe HBTs, and this block is built specifically to exploit that rather than port a CMOS-only approach. A naive Brokaw/Kuijk-style bandgap generates its output as VBE + K·ΔVBE in series, landing near the physical silicon bandgap voltage (≈1.2 V) — a value that doesn't shrink with technology scaling, so a literal implementation needs roughly 1.3–1.5 V of supply headroom, incompatible with our 1.0 V target. Instead, this block sums the PTAT and CTAT branches as **currents** into a single output resistor (rather than stacking voltages in series), so the output level is set by a resistor ratio and can be placed anywhere below the ~1.2 V physical constant while every internal node stays within 1.0 V-friendly headroom. This is the same principle published in P. Malcovati, F. Maloberti, C. Fiocchi, U. Pruzzi, "Curvature-Compensated BiCMOS Bandgap with 1-V Supply Voltage," IEEE JSSC, July 2001 — adapted here to SG13G2's HBTs.
*Risk: low–medium. Reuse: any SoC needing a reference; a strong, distinctive IHP-only library entry. Target supply: ~1.0 V by design (the entire point of the current-summed architecture).*

**Novelty:** based on a search of the existing Tiny Tapeout / Chipalooza ecosystem, no IHP-shuttle Tiny Tapeout submission to date (TTIHP0p2, TTIHP25a, TTIHP25b, TTIHP26a) has used SG13G2's HBTs for a bandgap reference. The bandgap references that do exist in the TT ecosystem (e.g. `tt_um_bgr_agolmanesh` on TT08, and one design among TT07's analog entries) are Sky130-based and necessarily CMOS-only, since Sky130 has no dedicated BJTs. IHP's own direct-customer MPW tapeout stream separately includes a `bandgap_ref_cmos` project (IHP-GmbH/TO_Apr2025, testfield T586) — not a Tiny Tapeout submission, and notably CMOS rather than BJT-based, consistent with the headroom argument above. This block would be, as far as can be determined, the first HBT-based, sub-1.2V-headroom bandgap reference contributed to any open-PDK Tiny Tapeout/Chipalooza library — a genuine differentiator for reviewers, not just an incremental design.

**Block 4: Switched-capacitor bandgap ("end-of-life" variant)**
A resistor-free bandgap: PTAT/CTAT information is generated via capacitor ratios and correlated double sampling / chopping instead of poly resistors, using Block 2 as the core amplifier and Block 1 as the clock/chopping controller. Two things make this attractive for the library specifically: (a) it has no dependency on HBTs or precision resistors, so — unlike Block 3 — it is directly portable to Sky130 and GF180MCU, which is precisely the "cross-platform porting" goal the challenge calls out; and (b) it demonstrates the composability story end-to-end: two Tier-0 primitives assembled into a working reference.
*Risk: medium (most novel block, but built entirely from already-validated Tier 0 sub-circuits, which substantially de-risks it relative to a from-scratch design). Target supply: ~1.0 V — since the output level here is set by capacitor ratio rather than a forced series voltage stack, it is not subject to the same ~1.2 V physical-constant constraint as Block 3's naive form, and is expected to be at least as 1.0 V-friendly.*

## 4. Delivery Plan / Risk Tiering

| Tier | Block | Fallback if schedule slips |
|---|---|---|
| 0 | Comparator | Ships regardless — simplest, first learning vehicle |
| 0 | Ring-amp OTA | Ships regardless — second learning vehicle |
| 1 | HBT bandgap | Ships if Tier 0 lands with margin |
| 1 | SC bandgap | Ships if Tier 0 + HBT bandgap land with margin; otherwise documented as schematic/sim-validated only, with layout deferred to a follow-up submission |

Even in the worst case, the tile ships two fully silicon-proven, independently useful library primitives (comparator + ring-amp OTA) — never zero.

## 5. Deliverables (per block)

- Schematic + testbench (open-source: ngspice/Xyce via xschem; cross-checked against Cadence/Spectre where useful for sanity)
- Layout (Magic/KLayout), DRC/LVS/PEX clean against the SG13G2 open PDK
- Characterization plan: process corners + Monte Carlo (offset, PSRR/line regulation for the bandgaps, gain/BW for the ring-amp, offset/kickback for the comparator)
- Public GitHub repository per block (schematics, layout, testbenches, README), consistent with the existing Efabless/Sky130 Chipalooza repository precedent
- Test/characterization hooks exposed through the shared analog mux plus the Tiny Tapeout demo board, so each block can be measured on real silicon post-tapeout

## 6. About the Proposer

25+ years of full-custom analog/mixed-signal IC design (Cadence Virtuoso/Spectre, 130–350 nm CMOS/BiCMOS), including 47 V high-voltage charge pumps with >40 dB PSRR, low-drift ring oscillators (first-time-right silicon, 6+ years in volume production), Ahuja-compensated low-noise amplifiers (13 µVrms), SAR-ADC quantizer redesign (50% power reduction), and current-steering DACs — all directly relevant to the blocks proposed here. Full ownership of tape-out flow including DRC/LVS/PEX/DFM in industrial (proprietary) tools. Three prior Tiny Tapeout submissions (Sky130 CPU, Sky130 all-digital PLL, IHP ring-oscillator array), but this is the first project using open-source **custom analog layout** tools — a deliberate, disclosed learning curve that this proposal's block sequencing is designed to absorb safely.

## 7. Longer-Term Roadmap (context, not part of this proposal)

Once these four primitives are silicon-proven, the natural next Chipalooza submission is a cyclic/pipeline ADC assembled from them directly: Block 1 as the sub-ADC quantizer, Block 2 as the residue/gain amplifier, Block 3 or 4 as the reference. This proposal is intentionally the risk-reducing first step toward that goal.
