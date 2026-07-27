# Catalin Lazar

**Analog Mixed-Signal IC Design Engineer**

Adliswil, Switzerland    
+41 77 496 1857 \
catalin.lazar.subs@gmail.com \
[linkedin.com/in/catalinlazar](https://www.linkedin.com/in/catalinlazar)

## Professional Summary

Senior analog/mixed-signal IC designer with 25+ years industrial experience delivering low-power, low-noise CMOS circuits for MEMS interfaces, RFID/NFC transponders, LCD drivers and data converters. Full ownership of schematic, layout, verification and tape-out in 130–350 nm CMOS, including first-time-right silicon in high-volume production. Expertise in comparators, operational amplifiers, voltage/current references, oscillators, SAR ADCs, current-steering DACs, charge pumps and mixed-signal verification (Verilog-AMS, Xcelium/Symphony). Recent open-source silicon submissions via Tiny Tapeout. First project using open-source custom analog layout tools (xschem/ngspice/Magic/KLayout); learning curve disclosed. Seeking to contribute high-quality, fully documented, reusable analog IP blocks for the open PDKs (IHP SG13CMOS5L, sky130, GF180MCU).

## Professional Experience

### Senior ASIC Analog Design Engineer  
**Syntiant | Knowles | ams** — Rapperswil-Jona, Switzerland  
Jul 2017 – Oct 2025  
Mixed-signal MEMS microphone interfaces in CMOS 130 nm and 300 nm

- Redesigned high-voltage charge pump (47 V, PSRR >40 dB, noise 2 µV_rms typ / 8 µV_rms worst), increasing output by 2.2 V without added stages/area using statistical SOA margin analysis from production data. Ensured stability, noise and PSRR via SpectreRF; performed SOAC and drive-margin verification.
- Designed low-power low-drift dual ring-oscillator (1.6 MHz / 14 MHz, 0.75 µA/MHz, <10 % drift over VT); first-time-right silicon in high-volume production for 6+ years.
- Reduced balancing-amplifier noise from 22 to 13 µV_rms and redesigned compensation (Ahuja) for increased load (1.4 to 5 pF).
- Reduced ΣΔADC quantizer (SAR architecture) power by 47 % (32 to 17 µW), supply scaled to 0.95 V.
- Implemented current-steering DAC fast trim-mode (10 ms) for cut-off frequency (7.5 Hz LFRO) using internal waveform generator and FFT.
- Designed low-cost 3-channel RF EMI filter with ESD (0.3 mm²); extracted IBIS models.
- Conducted top-level mixed-signal verification with Xcelium/Symphony using Verilog-AMS models.
- Introduced Trojan States Elimination and Spectre floating-node checks (dyn_floatdcpath) for robust start-up over PVT.
- Owned full custom flow: schematic, layout (DRC/LVS/PEX/DFM), tape-out, yield support, production.

### IC Analog Design Engineer  
**EM Microelectronic** — Marin, Switzerland  
Jan 2006 – Jun 2017  
Ultra-low-power HF/UHF RFID transponders and smartpen ASICs in CMOS 180 nm

- Designed first-in-company UHF RFID temperature sensor: low-power (8 µA) ΣΔADC.
- Developed low-power PLL (900 mV, 1.4 µA, 4 MHz).
- Designed 24 V / 400 kHz pulse transmitter using experimental LDMOS ahead of PDK; collaborated with process team.
- Owned power-management blocks: rectifier, charge-pump doubler, PTAT/current reference, bandgap reference, LDO.
- Implemented demodulators, TSPC prescalers.
- Full design flow: schematic/layout (LayoutXL), DRC/LVS/PEX/DFM (PVS/QRC/Calibre), job-view verification.
- Characterized prototypes, supported production yield improvement.
- Achieved 1 m HF RFID read range at 10 µA total consumption.

### IC Analog Design Engineer  
**Philips Semiconductors** — Zürich, Switzerland  
Jun 2002 – Dec 2005  
LCD drivers in 350 nm high-voltage CMOS

- Designed power-management blocks, rail-to-rail opamps, column buffers, comparators.
- Full schematic and layout flow; scripted ERC filters in Perl and created PCells with SKILL.

### IC Analog Design Engineer  
**Antrim Design Systems** — Le Vaud, Switzerland  
Oct 2000 – May 2002  
Nyquist-rate data-converter IP in 180–350 nm CMOS

- Designed R-DAC drivers for 12-bit 2 MS/s SAR ADC; 10-bit 100 MHz current-steering DAC unit-cell.
- Designed 900 MHz Gm-C filter, opamps, comparators.
- Transferred pipeline ADC sample-and-hold from 350 nm to 180 nm CMOS.
- Verified folding-interpolating ADC using Verilog-A model.

### IC Analog Design Engineer  
**Melexis** — Bevaix, Switzerland  
Aug 1999 – Sep 2000  
RFID reader and transponder ICs in 0.6 µm CMOS

- Designed Sallen-Key anti-aliasing filters, charge pumps, clock recovery, low-power TSPC prescalers.

## Earlier Research Experience

### Research Fellow — CERN (SL Division)  
Nov 1996 – Jul 1999  
Designed and prototyped fast-pulsed septum electromagnet for LHC injection: 1 T field with 500 ppm/cm homogeneity.

### Ph.D. Candidate — femto-st Optical Laboratory  
University of Franche-Comté, Besançon, France  
Nov 1991 – Oct 1996  
Dissertation: [Tunable optical frequency filter by double Fabry-Pérot cavity on LiNbO₃](https://theses.fr/1997BESA2044). Designed, micro-fabricated and characterised double-cavity electro-optically tunable Fabry-Pérot filter on Ti-diffused LiNbO₃ waveguides.

## Education

- **Ph.D. in Integrated Optics** — University of Franche-Comté, Besançon, 1997  
- **DEA Electronics (Optical Communications)** — University of Limoges, 1991  
- **B.Sc. Electronics & Telecommunications** — Polytechnic Institute Bucharest, 1990  
  Major: Electronic Devices & Components / Microelectronics — GPA 9.98/10

## Skills

Cadence Virtuoso / ADE / Layout XL / Spectre (X, RF: PSS, Pnoise, Pstab) / AFS / Verilog-AMS / Xcelium / Symphony / Ocean / DRC / LVS / PEX / DFM / Calibre / IBIS  
Open-source EDA (first use): xschem, ngspice, Magic, KLayout  
MATLAB / Python (NumPy, Pandas, SciPy, SymPy) / Perl / SKILL / Git / SVN / Perforce  
Low-power / low-noise design / charge pumps / ring oscillators / SAR ADC / current-steering DAC / PLL / voltage & current references / mixed-signal verification / DFMEA

## Open-Source Silicon

- [IHP Gate Delay Characterizer](https://tinytapeout.com/chips/ttihp26a/tt_um_catalinlazar_ihp_osc_array) (Tiny Tapeout IHP 26a) — 3-flavor ring-oscillator array
- [TinyCore8](https://tinytapeout.com/chips/ttsky26a/tt_um_catalinlazar_tinycore8) (Tiny Tapeout SKY 26a) — basic CPU
- [127-stage Coarse-Tapped ADPLL](https://tinytapeout.com/chips/ttsky26b/tt_um_catalinlazar_adpll_125m_sky130) (Tiny Tapeout SKY 26b)

## Languages

English (C1) | French (C1) | German (B2+) | Romanian (C2) | Spanish (A2) | Italian (A1)

## Citizenship

Romanian & Swiss

---

## Patents

- [US 11,979,714 B2](https://patents.justia.com/patent/11979714) (2024): Optical transducer and method for measuring displacement
- [US 11,808,654 B2](https://patents.justia.com/patent/11808654) (2023): Integrated optical transducer and method for detecting dynamic pressure changes
- [US App. 20220337034](https://patents.justia.com/patent/20220337034) (2022): Optical device, photonic detector, and method of manufacturing an optical device
- [US App. 20220337032](https://patents.justia.com/patent/20220337032) (2022): Tuning arrangement and method for tuning
- [US 10,361,593 B2](https://patents.justia.com/patent/10361593) (2019): Dual frequency HF-UHF identification device

## Selected Courses

- Getting the Best Spectre Simulator Results — Andrew Beckett, Cadence, 2025
- [Precision Low-Dropout Regulators — Prof. Yan Lu, 2025](https://www.linkedin.com/in/catalinlazar/details/featured/1770399649634/single-media-viewer?type=DOCUMENT&profileId=ACoAAABWgUwBQR0tFyQ9UTnHbgKpClf9KcONx88)
- [High-Performance SERDES Design — Prof. Sam Palermo, 2025](https://www.linkedin.com/in/catalinlazar/details/featured/1770399502852/single-media-viewer?type=DOCUMENT&profileId=ACoAAABWgUwBQR0tFyQ9UTnHbgKpClf9KcONx88)
- [The Art of CMOS RF Design & Layout — Prof. Patrick Reynaert, 2025](https://www.linkedin.com/in/catalinlazar/details/featured/1770399336322/single-media-viewer?type=DOCUMENT&profileId=ACoAAABWgUwBQR0tFyQ9UTnHbgKpClf9KcONx88)
- [Extreme SAR ADCs — Prof. Chi-Hang Chan, 2024](https://www.linkedin.com/in/catalinlazar/details/featured/1770399006787/single-media-viewer?type=DOCUMENT&profileId=ACoAAABWgUwBQR0tFyQ9UTnHbgKpClf9KcONx88)
- [Energy-Efficient Analog IC Design — Prof. Patrick Mercier, 2024](https://www.linkedin.com/in/catalinlazar/details/featured/1770398798600/single-media-viewer?type=DOCUMENT&profileId=ACoAAABWgUwBQR0tFyQ9UTnHbgKpClf9KcONx88)
- [CMOS RF Transceivers — Prof. Thomas Cho, 2023](https://www.linkedin.com/in/catalinlazar/details/featured/1770398631011/single-media-viewer?type=DOCUMENT&profileId=ACoAAABWgUwBQR0tFyQ9UTnHbgKpClf9KcONx88)
- [Practical Filter Design Techniques — Prof. Shanthi Pavan, 2023](https://www.linkedin.com/in/catalinlazar/details/featured/1770397800892/single-media-viewer?type=DOCUMENT&profileId=ACoAAABWgUwBQR0tFyQ9UTnHbgKpClf9KcONx88)
- [Modern Wireline Transceivers — Prof. Tony Chan Carusone, 2023](https://www.linkedin.com/in/catalinlazar/details/featured/1770397647267/single-media-viewer?type=DOCUMENT&profileId=ACoAAABWgUwBQR0tFyQ9UTnHbgKpClf9KcONx88)
- [Advanced ADC Design Techniques — Prof. Seung-Tak Ryu, 2022](https://www.linkedin.com/in/catalinlazar/details/featured/1770397508647/single-media-viewer?type=DOCUMENT&profileId=ACoAAABWgUwBQR0tFyQ9UTnHbgKpClf9KcONx88)
- Analog design — Prof. Maher Kayal (EPFL)
- Deep-submicron CMOS IC design — Harry Veendrick
- ESD phenomena in ICs — NXP / Philips
- Cadence trainings: QRC, Spectre, SKILL, Ultrasim, XL Layout Editor
