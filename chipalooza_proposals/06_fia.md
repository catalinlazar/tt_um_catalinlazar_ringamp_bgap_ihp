# Chipalooza Challenge #2 (IHP SG13CMOS5L) — Proposal
## Floating Inverter Amplifier (FIA)

*Target process: IHP SG13CMOS5L. Submitted to chipalooza@opencircuitdesign.com.
Personal/institutional details and CVs are provided separately per the
challenge rules, not included in this document.*

### 1. IP Block Type
Operational amplifier, low power.

### 2. I/O List (including test ports)

| Signal | Type | Notes |
|---|---|---|
| INP, INN | Analog input | Differential input, via shared analog mux |
| OUT | Analog output | Via shared analog mux |
| RST | Digital input | Reset/auto-zero phase clock, via shared control bus |
| VDD, VSS | Supply | 1.2V digital rail |

Test ports: functional pins double as test access.

**Power architecture note:** core supply is nominally 1.2V via an on-board (not on-chip) LDO, but all supplies route through a header and can be overridden from an external source. Each project's periphery and core supplies are independently power-gated, so a project can run at a different voltage (e.g. 1.5V, within the standard cells' 1.08-1.65V characterized range) without affecting other projects on the shared test chip.

### 3. Functional Description
A single CMOS inverter stage used as the gain element, with its supply
"rails" implemented as floating nodes pre-charged via capacitors during a
reset/auto-zero phase. This lands the inverter's bias point at peak gain
(both devices near saturation, close to the switching threshold)
regardless of input common-mode, giving class-AB-like push-pull
efficiency without a static tail-current bias. This is one of three
structurally distinct low-power OTA candidates submitted as companion
proposals (alongside the ring-amplifier and the ZCBC discharge
amplifier) — an intentional bake-off, not independent/redundant work.
*Note on sourcing: the specific originating citation for this topology
was not independently confirmed at proposal-draft stage; will be
verified against IEEE Xplore before the schematic-review milestone.*

### 4. Target Specification

| Parameter | Min | Typ | Max | Absolute limit | Notes |
|---|---|---|---|---|---|
| Supply voltage | ~1.0V | 1.2V | - | per process device rating (TBD) | |
| DC gain | - | >50dB | - | - | Target, pre-schematic |
| Gain-bandwidth product | 20MHz | 30MHz | 40MHz | - | TBD post-schematic |
| Settling time | - | <20ns to 0.1% | - | - | TBD, includes reset-phase floating-rail droop |
| Average power | - | <100uW | - | - | Target |

### 5. Test / Measurement Plan
- Closed-loop DC/AC characterization identical in approach to the
  companion ring-amp proposal (small-signal AC sweep, step-response
  settling).
- Explicit verification of floating-rail droop over the intended
  settling window, since this is the topology's main novel failure mode
  (charge leakage on the floating bias capacitors during operation).
