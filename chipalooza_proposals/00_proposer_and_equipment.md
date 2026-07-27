# Chipalooza Challenge #2 (IHP SG13CMOS5L) — Proposer Information & Equipment

*Sent separately from the seven technical proposals per challenge rule 6,
which requires personal/institutional details and CVs to be kept out of
the main proposal documents.*

## Proposer

Catalin Lazar. CV attached separately 
(see `Catalin_Lazar_CV.md` or `Catalin_Lazar_CV.pdf`).
25+ years of full-custom analog/mixed-signal IC design experience
(Cadence Virtuoso/Spectre, 130-350nm CMOS/BiCMOS). This is the first
project using open-source custom analog layout tools (Magic/KLayout/
xschem/ngspice) — a disclosed learning curve.

## Proposals submitted (7, companion set)

1. Dynamic Latched Comparator
2. Ring-Amplifier OTA
3. CMOS 1.2V-Class Voltage Reference (current-summed)
4. Switched-Capacitor Voltage Reference
5. Zero-Crossing-Based (ZCBC) Discharge Amplifier
6. Floating Inverter Amplifier (FIA)
7. Classic Operational Amplifier (folded-cascode/telescopic)

Proposals 2, 5, and 6 are an intentional bake-off of three structurally
distinct modern low-power residue-amplifier architectures; proposal 7 is
a deliberately conventional backup for the same eventual application
(cyclic ADC). Proposals 3 and 4 are two structurally distinct alternative
voltage-reference candidates. All seven share design DNA and are intended
to compose into a future cyclic/pipeline ADC proposal once proven.

## Equipment planned for testing

- Digilent Analog Discovery 3 (2-channel scope, arbitrary waveform
  generator, logic analyzer, programmable power supply)
- Precision bench DMM (5.5+ digit)
- PID-controlled hotplate + handheld thermocouple thermometer, for
  temperature-coefficient characterization (hot end); cold spray/fridge
  for the cold end
- Anti-static handling equipment
- Precision voltage reference (for DMM cross-calibration)

## Open-source EDA environment

Standard Xschem/ngspice/Magic/KLayout flow, following the IIC-JKU
ihp-sg13g2-ams-chip-template (being ported to ihp-sg13cmos5l) and the
July IIC-OSIC-TOOLS release targeting ihp-sg13cmos5l specifically.
