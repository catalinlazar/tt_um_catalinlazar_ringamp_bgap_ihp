# Chipalooza Challenge #2 (IHP SG13CMOS5L) — Proposal
## CMOS 1.2V-Class Voltage Reference (Current-Summed, Banba-style)

*Target process: IHP SG13CMOS5L. Submitted to chipalooza@opencircuitdesign.com.
Personal/institutional details and CVs are provided separately per the
challenge rules, not included in this document.*

### 1. IP Block Type
CMOS 1.2V voltage reference (equivalent to a bandgap).

### 2. I/O List (including test ports)

| Signal | Type | Notes |
|---|---|---|
| VREF | Analog output | Via shared analog mux |
| TRIM (optional) | Digital input | Optional trim bits, via shared control bus, TBD |
| VDD, VSS | Supply | 1.2V digital rail |

Test ports: VREF itself is the test port for DC/TC characterization.

**Power architecture note:** core supply is nominally 1.2V via an on-board (not on-chip) LDO, but all supplies route through a header and can be overridden from an external source. Each project's periphery and core supplies are independently power-gated, so a project can run at a different voltage (e.g. 1.5V, within the standard cells' 1.08-1.65V characterized range) without affecting other projects on the shared test chip.

### 3. Functional Description
A current-summed reference (Banba-style: H. Banba et al., "A CMOS Bandgap
Reference Circuit with Sub-1-V Operation," IEEE JSSC, May 1999), built
entirely from subthreshold-biased CMOS devices — no bipolar devices are
used, consistent with this PDK's NPN unavailability / PNP being
uncharacterized. PTAT and CTAT currents are generated in separate
low-voltage branches and summed into a single output resistor, avoiding
the series-voltage-stacking headroom problem of a classical bandgap. This
is submitted as an **alternative** to the harness's built-in 1.2V bandgap
reference, not a replacement — intended to offer either improved
temperature stability or lower quiescent power as a differentiator, to be
established once real device characterization data is available; the
harness's own reference remains the default choice for designs not
specifically needing this block's characteristics.

### 4. Target Specification

| Parameter | Min | Typ | Max | Absolute limit | Notes |
|---|---|---|---|---|---|
| Supply voltage | ~1.0V | 1.2V | - | per process device rating (TBD) | |
| Output voltage | - | ~0.5V (open design choice, see note) | - | - | Output level is a free resistor-ratio choice; whether to target ~1.2V (directly comparable to the harness reference) or a distinct lower level is an open design decision to be resolved during schematic design |
| Temperature coefficient | - | <50ppm/C (target) | - | 0-85C | First-order compensated in the current-summing architecture; target to be validated post-schematic, since real device Vgs(T) is not perfectly linear |
| Line regulation | - | <1%/V | - | - | Target |
| Output impedance | - | <10kOhm | - | - | Buffered if needed |
| Quiescent power | - | <10uW | - | - | Target |
| Startup | guaranteed | - | - | - | Dedicated startup circuit required (current-summing loops have a degenerate zero-current stable point) |

### 5. Test / Measurement Plan
- DC output measurement via precision bench DMM (5.5+ digit) across a
  controlled temperature sweep (hotplate for the hot end, cold spray/
  fridge for the cold end) to extract TC.
- Line regulation via supply voltage sweep with DMM monitoring VREF.
- Startup behavior verified via power-up transient capture on scope.
- Output impedance verified via a known resistive load and DMM/scope
  measurement of the resulting voltage droop.
