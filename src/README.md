# src/

- `behavioral/` — Verilog-A behavioral models, used for early top-level
  verification before transistor-level schematics exist:
  - `comparator_beh.va` — Block 0, StrongARM/double-tail comparator
  - `ringamp_ota_beh.va` — Block 1, ring-amp/FEA-style OTA
  - `hbt_bandgap_beh.va` — Block 2, SiGe HBT bandgap: current-summed
    (Malcovati-style) PTAT+CTAT model, parameterized (N, R1-R3, k1/k2),
    targets ~1.0V by construction rather than a series VBE stack
  - `sc_bandgap_beh.va` — Block 3, switched-capacitor bandgap
  - `buffer_beh.va` — shared output buffer, generic placeholder (topology
    TBD, source follower is the leading candidate). ONE instance, used both
    for the normal signal path and for buffer-only characterization via
    bypass — deliberately not duplicated, to avoid reintroducing mismatch
  - `tt_um_catalinlazar_ringamp_bgap_ihp_beh.va` — top-level wrapper: mux
    (00/11=HBT bandgap default, 01=ring-amp, 10=SC bandgap - never
    floating), shared buffer with buf_bypass characterization mode, and
    ua_raw_sel mode-select on ua[1] (buffered vs bare mux output, same pad)

  These are functional stand-ins (ideal gain/threshold behavior), not
  transistor-accurate — refine or replace per-block as real topologies are
  chosen and simulated.

Still to add:
- `xschem/` — transistor-level schematics per block, plus the shared
  analog mux and top-level tile assembly

## Sanity-check results (`spice/`)

Four quick ngspice testbenches, mirroring the behavioral models as native
B-source expressions (not yet the compiled Verilog-A/OSDI path - that
still needs OpenVAF set up):

- `bandgap_tc.cir` — temperature sweep, 0-80C. First run used illustrative
  (wrong) coefficients and showed ~33mV drift; solved analytically for the
  correct k1 (3.3484) and re-ran: <1uV drift in this idealized linear-Vbe(T)
  model. A real implementation will show a few mV of residual curvature
  since real Vbe(T) isn't perfectly linear - that's what the optional
  curvature-correction term is for.
- `ringamp_ac.cir` — AC gain/bandwidth. Matches spec exactly: 60dB DC gain,
  pole at 50kHz, unity-gain at 50MHz (=GBW parameter).
- `comparator_dc.cir` — static decision transfer curve (not the edge-
  triggered sampling behavior - that needs the real Verilog-A/OSDI model
  or a transistor-level transient sim). Clean 0/1V digital swing, trip
  point at exactly -offset as expected.
- `mux_remap_tb.cir` — steps sel[1:0] through all four codes with the new
  mapping (00/11=HBT bandgap default, 01=ring-amp, 10=SC bandgap) with the
  shared buffer inline, then flips `ua_raw_sel` while still on sel=00 and
  confirms `ua_out` jumps from the buffered value (0.470V) to the raw
  mux_out value (0.480V) on the same pin — confirms the mode-select
  mechanism works and never requires a second physical pad.

All passed. See `sim_summary.png` and `mux_remap_tb.png` for plots.

`info.yaml`'s `source_files` list should be updated to include the real
file paths as they're added (currently points at the behavioral/ files).
