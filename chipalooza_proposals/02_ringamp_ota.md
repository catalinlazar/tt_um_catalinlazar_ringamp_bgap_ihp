# Chipalooza Challenge #2 (IHP SG13CMOS5L) — Proposal
## Ring-Amplifier OTA

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
| VDD, VSS | Supply | 1.2V digital rail (design target; see Section 4) |

Test ports: functional input/output pins double as test access. No
additional dedicated test pins requested.

**Power architecture note:** core supply is nominally 1.2V via an on-board (not on-chip) LDO, but all supplies route through a header and can be overridden from an external source. Each project's periphery and core supplies are independently power-gated, so a project can run at a different voltage (e.g. 1.5V, within the standard cells' 1.08-1.65V characterized range) without affecting other projects on the shared test chip.

### 3. Functional Description
An inverter-chain-based gain stage that exploits a narrow, high-gain
"dead zone" near the switching threshold of a cascaded inverter structure,
giving class-AB-like push-pull efficiency without a continuous static
bias tail current. Aimed at efficient residue amplification in switched-
capacitor circuits (e.g. pipeline/cyclic ADC MDACs). This is one of three
structurally distinct low-power OTA candidates submitted as companion
proposals (alongside the Floating Inverter Amplifier and the Zero-
Crossing-Based discharge amplifier) — an intentional bake-off to identify
the best-performing modern low-voltage residue amplifier for a future
cyclic-ADC submission, not independent/redundant work. Reference: B.
Hershberg et al., "Ring Amplifiers for Switched-Capacitor Circuits," IEEE
JSSC, Dec 2012.

### 4. Target Specification

| Parameter | Min | Typ | Max | Absolute limit | Notes |
|---|---|---|---|---|---|
| Supply voltage | ~1.0V | 1.2V | - | per process device rating (TBD) | Ring-amp topologies are specifically suited to operating well below classical op-amp headroom requirements |
| DC open-loop gain (in dead zone) | - | >55dB | - | - | Target, pre-schematic |
| Gain-bandwidth product | 30MHz | 40MHz | 50MHz | - | Load-dependent; TBD post-schematic |
| Output swing | - | rail-to-rail, within ~150mV | - | - | |
| Input-referred offset (untrimmed) | - | <20mV | - | - | Single matched input pair, no auto-zero at this design stage |
| Settling time (representative SC load) | - | <20ns to 0.1% | - | - | Exact load TBD pending slot/wrapper spec |
| Average power (typical SC operation) | - | <100uW | - | - | Target |

Numeric targets are pre-schematic design goals; to be verified and
refined by the schematic/pre-layout review (week of Aug 31).

### 5. Test / Measurement Plan
- Closed-loop (unity-buffer) DC/AC characterization: apply small-signal
  AC stimulus at INP/INN, measure OUT on scope/network-analyzer-class
  instrument (Analog Discovery 3 or equivalent) for gain/bandwidth.
- Step response on scope for settling-time characterization.
- If dev-board pin access allows, emulate the intended SC-MDAC load with
  an external sampling capacitor network to validate settling under
  realistic conditions.
- Power measured via supply-current monitoring during representative
  clocked operation.
