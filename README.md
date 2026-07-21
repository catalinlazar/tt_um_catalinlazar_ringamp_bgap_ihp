# tt_um_catalinlazar_ringamp_bgap_ihp

> **PROJECT_NAME placeholder:** every occurrence of the string
> `tt_um_catalinlazar_ringamp_bgap_ihp` in this repo (info.yaml `top_module`,
> this README, `docs/info.md`, and eventually the `gds/*.gds` /
> `gds/*.lef` filenames) is the same token. If you rename the project,
> find-and-replace this exact string everywhere, rename the GitHub repo to
> match, and re-run the DEF/GDS naming to match. Do this once, before your
> first submission — Tiny Tapeout requires the name to be identical across
> repo, `info.yaml`, and final GDS/LEF.

Submission for the [Chipalooza Challenge](https://sedemos.blogspot.com/2026/07/open-circuit-design-chipalooza-challenge.html),
targeting the IHP SG13G2 (130nm BiCMOS) PDK via [Tiny Tapeout](https://tinytapeout.com).

Four SG13G2 analog primitives on one 1x2 analog tile:
- Dynamic latched comparator (StrongARM/double-tail)
- Ring-amplifier (FEA-style) OTA
- SiGe HBT bandgap reference
- Switched-capacitor ("end-of-life") bandgap

See [`docs/info.md`](docs/info.md) for the full datasheet (how it works, pinout, how to test).

## Repo layout

```
info.yaml         - Tiny Tapeout project manifest (pins, tiles, top module)
docs/info.md       - Project datasheet (required by Tiny Tapeout)
src/               - Schematics (xschem), spice netlists, testbenches
gds/               - Final GDS + LEF (added once layout is complete)
LICENSE            - Apache-2.0 (default; update copyright header with your name)
```

## Status

- [ ] Schematics + testbenches (ngspice/xschem) for all 4 blocks
- [ ] Layout (Magic/KLayout) — Block 0 (comparator) first
- [ ] DRC/LVS/PEX clean
- [ ] GDS/LEF dropped into `gds/`
- [ ] Submitted to shuttle

## Based on

[TinyTapeout/ttsky-analog-template](https://github.com/TinyTapeout/ttsky-analog-template)
(current shared template for all custom-GDS/analog Tiny Tapeout submissions,
across sky130A / ihp-sg13g2 / gfmcu180D). Pin-placement DEF downloaded from
the [Analog Specs page](https://tinytapeout.com/specs/analog/) for the
ihp-sg13g2, 1x2 tile.
