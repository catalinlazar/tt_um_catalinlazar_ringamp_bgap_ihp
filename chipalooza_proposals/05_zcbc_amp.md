# Chipalooza Challenge #2 (IHP SG13CMOS5L) — Proposal
## Zero-Crossing-Based (ZCBC) Discharge Amplifier

*Target process: IHP SG13CMOS5L. Submitted to chipalooza@opencircuitdesign.com.
Personal/institutional details and CVs are provided separately per the
challenge rules, not included in this document.*

### 1. IP Block Type
Operational amplifier, low power. (No exact category match exists on the
signup form for this discharge-based mechanism; "low power op-amp" is the
closest fit — see Functional Description for how it differs from a
conventional continuous-time amplifier.)

### 2. I/O List (including test ports)

| Signal | Type | Notes |
|---|---|---|
| INP, INN | Analog input | Differential input, via shared analog mux |
| OUT | Analog output | Via shared analog mux |
| CLK | Digital input | Discharge-phase clock, via shared control bus |
| VDD, VSS | Supply | 1.2V digital rail |

Test ports: functional pins double as test access.

**Power architecture note:** core supply is nominally 1.2V via an on-board (not on-chip) LDO, but all supplies route through a header and can be overridden from an external source. Each project's periphery and core supplies are independently power-gated, so a project can run at a different voltage (e.g. 1.5V, within the standard cells' 1.08-1.65V characterized range) without affecting other projects on the shared test chip.

### 3. Functional Description
A comparator-based discharge amplifier (T. Sepke, J. Fiorenza, C. Sodini,
P. Holloway, H.-S. Lee, "Comparator-Based Switched-Capacitor Circuits for
Scaled CMOS Technologies," IEEE JSSC, Dec 2006): the output node is
discharged through a controlled current source on the clock edge; a
comparator watches for the output crossing the target (zero-crossing)
value and shuts the discharge current off at that instant. No static
bias current — power is spent only during the active discharge
transient. This design reuses the comparator topology from the companion
"Dynamic Latched Comparator" proposal (as its own dedicated instance,
not shared hardware) as the zero-crossing detector. This is one of three
structurally distinct low-power OTA candidates submitted as companion
proposals (alongside the ring-amplifier and Floating Inverter Amplifier)
— an intentional bake-off, not independent/redundant work. Known
non-ideality: finite comparator delay causes a systematic overshoot/gain
error proportional to (discharge current x comparator delay); mitigated
via a two-phase coarse/fine discharge, a standard technique from the
same literature.

### 4. Target Specification

| Parameter | Min | Typ | Max | Absolute limit | Notes |
|---|---|---|---|---|---|
| Supply voltage | ~1.0V | 1.2V | - | per process device rating (TBD) | Dynamic operation, no static headroom fight |
| Target closed-loop gain | 4x | 6x | 8x | - | Application-dependent; TBD |
| Gain error (comparator-delay-induced) | - | <1% | - | - | Bounded via two-phase discharge |
| Settling time | - | <20ns to 0.1% | - | - | TBD |
| Average power | - | <50uW | - | - | Target |

### 5. Test / Measurement Plan
- Clocked step/settling response captured on scope.
- Gain error measured across discharge-current and timing corners by
  sweeping the coarse/fine discharge timing and comparing against ideal
  closed-loop gain.
- If dev-board pin access allows, direct comparison against the
  companion ring-amp and classic-OTA proposals under an identical
  switched-capacitor test load, to inform the eventual cyclic-ADC choice.
