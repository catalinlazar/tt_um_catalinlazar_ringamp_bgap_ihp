# src/

- `behavioral/` — Verilog-A behavioral models, used for early top-level
  verification before transistor-level schematics exist:
  - `comparator_beh.va` — Block 0, StrongARM/double-tail comparator
  - `ringamp_ota_beh.va` — Block 1, ring-amp OTA
  - `cmos_vref_beh.va` — Block 2, CMOS 1.2V-class voltage reference:
    current-summed (Banba-style) PTAT+CTAT model using subthreshold-biased
    CMOS devices, parameterized (N, n_slope, R1-R3, k1/k2), targets ~1.0V
    by construction. CORRECTED from an earlier HBT-based version - this
    round's actual target (ihp-sg13cmos5l) has no usable NPN and an
    uncharacterized PNP, so a bipolar bandgap wasn't viable
  - `sc_bandgap_beh.va` — Block 3, switched-capacitor bandgap
  - `zcbc_amp_beh.va` — Block 4, Zero-Crossing-Based discharge amplifier
    (Sepke/Fiorenza/Sodini/Holloway/Lee, IEEE JSSC Dec 2006). Models the
    dominant known non-ideality (comparator-delay-induced overshoot) but
    not the actual RC discharge transient shape or two-phase mitigation
  - `fia_ota_beh.va` — Block 5, Floating Inverter Amplifier. Citation not
    independently verified - search IEEE Xplore before citing a specific
    paper. Behaviorally identical to the ring-amp model at this stage;
    real differentiation (floating-rail settling) needs transistor level
  - `classic_ota_beh.va` — Block 6, classic OTA/folded-cascode backup for
    the eventual cyclic ADC. Also behaviorally a single-pole gain block
    at this stage - the point of this block is conventional, well-
    understood risk, not behavioral novelty
  - `buffer_beh.va` — shared output buffer, generic placeholder (topology
    TBD, source follower is the leading candidate). ONE instance, used both
    for the normal signal path and for buffer-only characterization via
    bypass — deliberately not duplicated, to avoid reintroducing mismatch
  - `tt_um_catalinlazar_ringamp_bgap_ihp_beh.va` — top-level wrapper: 3-bit
    mux (000/110/111=CMOS voltage reference default, 001=ring-amp,
    010=SC bandgap, 011=ZCBC, 100=FIA, 101=classic OTA - never floating),
    shared buffer with buf_bypass characterization mode, and ua_raw_sel
    mode-select on ua[1] (buffered vs bare mux output, same pad)

Still to add:
- `xschem/` — transistor-level schematics per block, plus the shared
  analog mux and top-level tile assembly

## Sanity-check results (`spice/`)

ngspice testbenches mirroring the behavioral models as native B-source
expressions (not yet the compiled Verilog-A/OSDI path - that still needs
OpenVAF set up):

- `cmos_vref_tc.cir` — temperature sweep, 0-80C, for the corrected CMOS
  subthreshold reference (with the n_slope=1.4 factor accounted for in
  the k1 solve: k1=2.3917). <1uV drift in this idealized linear model.
  Superseded the earlier HBT-based `bandgap_tc.cir` (removed).
- `ringamp_ac.cir` — AC gain/bandwidth. Matches spec exactly: 60dB DC gain,
  pole at 50kHz, unity-gain at 50MHz (=GBW parameter).
- `comparator_dc.cir` — static decision transfer curve. Clean 0/1V digital
  swing, trip point at exactly -offset as expected.
- `mux_remap_tb.cir` — original 4-code (2-bit) mapping + raw-mode test,
  superseded by `mux_8way_tb.cir` below but kept for reference.
- `mux_8way_tb.cir` — full 3-bit, 8-code mapping test. Confirms all 8
  codes route to the correct block (CMOS voltage ref at 000/110/111, ring-amp
  at 001, SC bandgap at 010, ZCBC at 011, FIA at 100, classic OTA at 101).

All passed. See `sim_summary.png`, `mux_remap_tb.png` for earlier plots.
