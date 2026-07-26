<!--
PROJECT_NAME token: tt_um_catalinlazar_ringamp_bgap_ihp
(keep identical to info.yaml top_module / repo name / GDS filenames)
-->

## How it works

![Block diagram](block_diagram.png)

This project (`tt_um_catalinlazar_ringamp_bgap_ihp`) is a small library of
seven SG13G2 analog primitives sharing one 1x2 analog tile:

1. **Block 0 — Dynamic latched comparator** (StrongARM / double-tail latch)
2. **Block 1 — Ring-amplifier (FEA-style) OTA**
3. **Block 2 — SiGe HBT bandgap reference** (Brokaw/Banba-style, current-summed)
4. **Block 3 — Switched-capacitor bandgap** ("end-of-life" variant, built
   from Blocks 0 and 1)
5. **Block 4 — ZCBC amp** (Zero-Crossing-Based discharge amplifier, reuses
   Block 0 as its zero-crossing detector)
6. **Block 5 — FIA** (Floating Inverter Amplifier)
7. **Block 6 — Classic OTA/op-amp** (folded-cascode/telescopic as needed;
   conventional backup residue amplifier for the eventual cyclic ADC, in
   case the ring-amp underperforms in silicon)

Blocks 1 and 3-6 (everything except the comparator) share the analog bus
through a **3-bit** digitally controlled mux (`sel[2:0] = {ui[5], ui[1],
ui[0]}`) and a single shared output buffer. Only one is connected to
`ua[0..2]` at a time.

**Block-select mapping** (deliberately no floating/idle state — every
unused code safely defaults to the HBT bandgap):

| sel[2:0] | Block |
|---|---|
| 000 (default at reset) | HBT bandgap |
| 001 | Ring-amp OTA |
| 010 | SC bandgap |
| 011 | ZCBC amp |
| 100 | FIA |
| 101 | Classic OTA |
| 110 | HBT bandgap (spare, safe default) |
| 111 | HBT bandgap (spare, safe default) |

The default (000) is deliberately the HBT bandgap: a simple resistor-set
DC output that stays well-behaved under an unknown/undefined external
load, unlike the ring-amp/FIA/ZCBC (all tuned around a specific expected
load or timing) or the SC bandgap (charge-domain, actively corrupted by
unexpected external capacitance).

**Why three residue-amplifier candidates plus a classic backup:** this
round doubles as an informal bake-off for the eventual cyclic-ADC residue
amplifier. The ring-amp, FIA, and ZCBC are three structurally different
"modern, low-voltage-efficient" approaches to the same job (continuous
dead-zone amplifier, floating self-biased inverter, and comparator-based
discharge amplifier, respectively); the classic folded-cascode/telescopic
OTA is deliberately the least novel entry on the tile, included
specifically as proven insurance for the ADC roadmap if none of the three
modern candidates perform as hoped in silicon.

The mux output always passes through a single **shared output buffer**
before reaching `ua[1]` — isolating the sensitive block output nodes from
unpredictable off-chip loading (pad + ESD + cable + scope capacitance can
easily be 10-20x the on-chip load these circuits are tuned for, which
matters especially for the ring-amp/FIA/ZCBC's timing- or load-dependent
behavior). `buf_bypass` (`ui[3]`) forces this *same* buffer instance's
input to connect directly to `ua_in`, for buffer-only characterization —
deliberately the same instance, not a duplicate copy, so its measured
non-idealities can be de-embedded from the block measurements without
adding new instance-to-instance mismatch. Buffer topology TBD (source
follower is the leading candidate).

`ua_raw_sel` (`ui[4]`) is a **mode select on the same `ua[1]` pad**, not a
second physical pin: 0 = buffer output (normal), 1 = bare mux output,
bypassing the buffer entirely — useful if the buffer itself is ever in
doubt. A dedicated second pin for this was considered and rejected: it
would permanently load the sensitive mux_out node with pad/ESD
parasitics (worst case for the ring-amp/FIA/ZCBC) even when never used.
The trade-off is that raw and buffered outputs can only be viewed
sequentially, not simultaneously on two scope channels.

The comparator (Block 0) is not part of the mux at all: its input pair
taps `ua[0]`/`ua[2]` directly and permanently (same pins the other blocks'
inputs use), independent of `sel[2:0]`, and its digital decision goes
straight to `uo[0]` — no buffering needed since it's already a digital
level.

All blocks target a **~1.0V supply**. This is straightforward for the
comparator and the three residue-amp candidates (dynamic/inverter-chain/
discharge-based structures with no static headroom fight) but is the
interesting constraint for Block 2: a naive BJT bandgap generates
VBE + K*ΔVBE in series, landing near the physical silicon bandgap voltage
(~1.2V) which doesn't shrink with process scaling, so it would need
~1.3-1.5V of headroom. Block 2 instead sums PTAT/CTAT as *currents* into
a single output resistor (Malcovati-style), so the output level is a
resistor ratio rather than a forced series stack, keeping every internal
node within 1.0V-friendly headroom. Block 3 inherits the same benefit
naturally, since its output level is set by capacitor ratio. Block 6
(classic OTA) is the one place 1.0V is a genuine design challenge if
folded-cascode ends up being necessary rather than a simpler topology.

To our knowledge Block 2 is the first HBT-based bandgap reference on any
open-PDK Tiny Tapeout shuttle; existing TT bandgap projects (e.g. TT07/TT08)
are Sky130-based and CMOS-only, since Sky130 lacks dedicated BJTs.

## How to test

_(To be completed once testbenches exist. Outline per block:)_
- Block 0 (comparator): apply differential signal to `ua[0]`/`ua[2]`, clock
  on `ui[2]`, read digital result on `uo[0]`. Not affected by `sel[2:0]`.
- Block 1 (ring-amp OTA): set `sel[2:0]=001`, apply input on `ua[0]`, read
  amplified output on `ua[1]` (buffered by default).
- Block 2 (HBT bandgap): `sel[2:0]=000` (also the power-on default), `110`,
  or `111`, read DC reference voltage on `ua[1]`, no input needed.
- Block 3 (SC bandgap): set `sel[2:0]=010`, read DC reference voltage on
  `ua[1]`, requires `ui[2]` clock for the switching phases.
- Block 4 (ZCBC amp): set `sel[2:0]=011`, apply input on `ua[0]`, clock on
  `ui[2]`, read the discharge-settled output on `ua[1]`.
- Block 5 (FIA): set `sel[2:0]=100`, apply input on `ua[0]`, read output
  on `ua[1]`.
- Block 6 (classic OTA): set `sel[2:0]=101`, apply input on `ua[0]`, read
  output on `ua[1]`.
- Buffer characterization: `buf_bypass=1` routes `ua_in` straight into the
  buffer (any `sel[2:0]`), read `ua[1]`.
- Raw/unbuffered debug mode: `ua_raw_sel=1` makes `ua[1]` show the bare
  mux output instead of the buffer output, for the currently selected
  block. Switch back to `ua_raw_sel=0` to compare — not simultaneous.

## External hardware

None required beyond the Tiny Tapeout demo/breakout board and a bench
supply/scope or DMM for the analog pins (or the TT analog measurement
add-on, if used).
