# Chipalooza Challenge #2 (IHP SG13CMOS5L) — Proposal
## Switched-Capacitor Voltage Reference ("End-of-Life" variant)

*Target process: IHP SG13CMOS5L. Submitted to chipalooza@opencircuitdesign.com.
Personal/institutional details and CVs are provided separately per the
challenge rules, not included in this document.*

### 1. IP Block Type
CMOS 1.2V voltage reference (equivalent to a bandgap).

### 2. I/O List (including test ports)

| Signal | Type | Notes |
|---|---|---|
| VREF | Analog output | Via shared analog mux |
| CLK | Digital input | Switching-phase clock, via shared control bus |
| VDD, VSS | Supply | 1.2V digital rail |

Test ports: VREF is the test port; CLK access also enables direct
observation of switching-phase ripple.

**Power architecture note:** core supply is nominally 1.2V via an on-board (not on-chip) LDO, but all supplies route through a header and can be overridden from an external source. Each project's periphery and core supplies are independently power-gated, so a project can run at a different voltage (e.g. 1.5V, within the standard cells' 1.08-1.65V characterized range) without affecting other projects on the shared test chip.

### 3. Functional Description
A resistor-free reference: PTAT/CTAT information is generated via
capacitor ratios and correlated double sampling / chopping instead of
poly resistors. This is a companion proposal to the CMOS voltage
reference (current-summed) submission — a second, structurally distinct
reference candidate for the library, using the charge domain rather than
the resistive domain. Two properties differentiate it: (a) no dependency
on precision resistors, which may offer better matching/area trade-offs
than a resistor-based design; (b) the underlying mechanism (capacitor-
ratio-based generation, no bipolar devices) is directly portable to
Sky130 and GF180MCU, supporting the cross-platform IP-library goal.

### 4. Target Specification

| Parameter | Min | Typ | Max | Absolute limit | Notes |
|---|---|---|---|---|---|
| Supply voltage | ~1.0V | 1.2V | - | per process device rating (TBD) | |
| Output voltage | - | ~0.5-1.2V (open design choice) | - | - | Same open design question as the companion CMOS-vref proposal |
| Clock rate | 100kHz | - | 1MHz | - | Target range, TBD |
| Output ripple (post-settling) | - | <5mV pk-pk | - | - | Target |
| Temperature coefficient | - | <50ppm/C (target) | - | 0-85C | To be validated post-schematic |
| Average power | - | <10uW | - | - | Target |

### 5. Test / Measurement Plan
- DC output measurement via precision DMM across a controlled temperature
  sweep (same setup as the companion CMOS-vref proposal), for TC
  extraction.
- Clock-synchronized ripple measurement on scope, triggered from CLK.
- Settling-time-to-target measurement following power-up or clock start.
