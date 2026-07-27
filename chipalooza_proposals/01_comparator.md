# Chipalooza Challenge #2 (IHP SG13CMOS5L) — Proposal
## Dynamic Latched Comparator

*Target process: IHP SG13CMOS5L. Submitted to chipalooza@opencircuitdesign.com.
Personal/institutional details and CVs are provided separately per the
challenge rules, not included in this document.*

### 1. IP Block Type
Comparator, clocked.

### 2. I/O List (including test ports)

| Signal | Type | Notes |
|---|---|---|
| INP, INN | Analog input | Differential input pair; expected shared via the harness's analog mux |
| CLK | Digital input | Sampling clock, via shared digital control bus |
| DOUT | Digital output | Decision bit, via shared digital status bus |
| VDD, VSS | Supply | 1.2V digital rail |

Test ports: the functional differential inputs and digital decision output
double as the test ports — no additional dedicated test pins requested.
Exact mapping onto the SPI-based control/status bus will follow the
published wrapper/slot template.

**Power architecture note:** core supply is nominally 1.2V via an on-board (not on-chip) LDO, but all supplies route through a header and can be overridden from an external source. Each project's periphery and core supplies are independently power-gated, so a project can run at a different voltage (e.g. 1.5V, within the standard cells' 1.08-1.65V characterized range) without affecting other projects on the shared test chip.

### 3. Functional Description
A dynamic, regenerative latched comparator (StrongARM / double-tail
topology): a differential input pair sets an initial imbalance on a
cross-coupled inverter latch (reset while CLK is low), which regenerates
to a full-swing digital decision on the CLK rising edge. Fully dynamic —
no static bias current, power drawn only during the clock transition and
regeneration. No compensation network or bias-current trim required,
making it a comparatively low-layout-risk block relative to continuous-
time analog circuits.

### 4. Target Specification

| Parameter | Min | Typ | Max | Absolute limit | Notes |
|---|---|---|---|---|---|
| Supply voltage | ~1.0 | 1.2 | - | per process device rating (TBD) | Designed with margin below the 1.2V digital rail |
| Input common-mode range | 0.2V | - | 1.0V | - | To be refined post-schematic |
| Input-referred offset (3-sigma, untrimmed) | - | <15mV | <30mV | - | No calibration assumed at this design stage |
| Decision (clock-to-output) time | - | <3ns | <6ns | - | At 1.2V, 27C; across-corner max is a target pending schematic simulation |
| Kickback charge | - | minimized via balanced sizing | - | - | To be characterized post-schematic |
| Energy per decision | - | <5fJ (target) | - | - | Aggressive target; to be refined |
| Max clock rate | 200MHz | - | - | - | Target, TBD |

All numeric targets above are pre-schematic design goals based on the
topology's known behavior in comparable processes, not yet verified by
transistor-level simulation in this specific PDK — expected to firm up
by the schematic/pre-layout review (week of Aug 31).

### 5. Test / Measurement Plan
- Apply differential DC stimulus to INP/INN via bench source (Analog
  Discovery 3 or equivalent precision source); sweep to map trip point
  and offset.
- Drive CLK externally (or via provided control bus in test mode); measure
  decision latency on a scope/logic analyzer referenced to the CLK edge.
- Sweep ambient temperature (controlled hotplate for the hot end, cold
  spray/fridge for the cold end) to characterize offset temperature
  coefficient.
- Measure kickback by probing INP/INN during a CLK transition with the
  input source disconnected (high impedance) to observe injected charge.
- Power measured via supply-current monitoring during a defined clocking
  sequence to extract energy/decision.
