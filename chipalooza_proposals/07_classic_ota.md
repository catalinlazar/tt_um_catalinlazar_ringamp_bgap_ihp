# Chipalooza Challenge #2 (IHP SG13CMOS5L) — Proposal
## Classic Operational Amplifier (Folded-Cascode / Telescopic)

*Target process: IHP SG13CMOS5L. Submitted to chipalooza@opencircuitdesign.com.
Personal/institutional details and CVs are provided separately per the
challenge rules, not included in this document.*

### 1. IP Block Type
Operational amplifier, high gain-bandwidth.

### 2. I/O List (including test ports)

| Signal | Type | Notes |
|---|---|---|
| INP, INN | Analog input | Differential input, via shared analog mux |
| OUT | Analog output | Via shared analog mux |
| VDD, VSS | Supply | Boosted core rail, target ~1.5V (see Section 3) |

Test ports: functional pins double as test access.

**Power architecture note:** core supply is nominally 1.2V via an
on-board (not on-chip) LDO, but all supplies route through a header and
can be overridden from an external source. Each project's periphery and
core supplies are independently power-gated, so a project can run at a
different voltage (e.g. 1.5V, within the standard cells' 1.08-1.65V
characterized range) without affecting other projects on the shared
test chip.

### 3. Functional Description
A conventional continuous-time OTA (topology — telescopic, folded-
cascode, or two-stage — to be finalized once real gain/swing
requirements are checked against available headroom). Deliberately the
most conventional design in this set of submissions: included
specifically as a proven, well-understood alternative residue amplifier
for an eventual cyclic-ADC proposal, in case the three modern low-power
candidates (ring-amp, FIA, ZCBC — companion proposals) do not perform as
hoped in silicon.

**Supply choice, revised:** targets a **boosted core rail (~1.5V)**
rather than the separate 3.3V analog rail originally considered. Staying
within the core/thin-oxide device class (same as the other six companion
proposals) avoids the interface complexity of straddling two different
oxide/voltage domains (level-shifting, thick-oxide-specific layout rules
we have no experience with yet), while still providing ~300mV of extra
cascode headroom over the shared 1.2V baseline — expected to be
sufficient for a comfortable folded-cascode design given standard cells
are characterized up to 1.65V. **Open item to confirm with organizers:**
whether a project's "periphery" supply domain (as distinct from "core")
handles interfacing with the shared digital control bus automatically at
a boosted core voltage, or whether internal level-shifting would need to
be designed by us — not yet clear from available information. Falling
back to the 3.3V analog rail remains available as a more conservative
option if 1.5V proves insufficient once real schematic-level sizing is
done.

### 4. Target Specification

| Parameter | Min | Typ | Max | Absolute limit | Notes |
|---|---|---|---|---|---|
| Supply voltage | 1.2V | 1.5V | 1.65V | per standard-cell characterization (1.08-1.65V) | Boosted core rail, project-specific via power gating |
| DC gain | 60dB | >70dB | - | - | Target, pre-schematic |
| Gain-bandwidth product | 15MHz | 25MHz | 30MHz | - | At representative SC load, TBD |
| Phase margin | 60deg | - | - | - | Target |
| Output swing | - | within ~300mV of each rail | - | - | Target |
| Input-referred offset (untrimmed) | - | <10mV | - | - | Target |
| Power | - | <300uW | - | - | Revised down from the earlier 3.3V-rail estimate, reflecting the lower boosted-core voltage |

### 5. Test / Measurement Plan
- Standard closed-loop AC/DC characterization: gain and GBW via small-
  signal AC sweep or loop-gain measurement, phase margin from the same
  data.
- Offset via DC sweep in a unity-buffer configuration.
- Step-response settling on scope for direct comparison against the
  three modern-OTA companion proposals under an identical test load.
- Core-supply sweep (1.2V-1.65V, via the external header override) to
  characterize gain/swing/headroom margin as a function of the boosted
  rail voltage, and to confirm the 1.5V target is well-chosen before
  committing to layout.
