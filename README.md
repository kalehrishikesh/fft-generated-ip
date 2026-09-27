# FFT Accelerator IP (Chipyard-generated)

Parameterizable FFT accelerator RTL, generated with [Chipyard](https://chipyard.readthedocs.io/)'s
[FFT Generator](https://chipyard.readthedocs.io/en/latest/generators/fft/) using the `FFTRocketConfig`
configuration, and verified in Verilator simulation (the bundled `sw/fft.c` test passes).

## Configuration

| Parameter | Value |
|-----------|-------|
| Points | 8 |
| Data width | 16-bit real + 16-bit imaginary per point (fixed-point) |
| Decimal point | 8 fractional bits (`decPt = 8`) |
| Base address | `0x2400` (memory-mapped) |

Generated from:

```scala
class FFTRocketConfig extends Config(
  new fftgenerator.WithFFTGenerator(numPoints = 8, width = 16, decPt = 8) ++
  new freechips.rocketchip.rocket.WithNHugeCores(1) ++
  new chipyard.config.AbstractConfig)
```

## Layout

- `rtl/fft/` — the FFT accelerator modules only:
  - `FFT.sv` — FFT datapath top (8 complex inputs -> 8 complex outputs)
  - `DirectFFT.sv` — butterfly stages
  - `Deserialize.sv` — collects input points written over the bus
  - `Unscramble.sv` — reorders outputs to natural order
  - `Tail.sv`, `LazyTail.sv` — memory-mapped bus interface (TileLink)
- `rtl/soc/gen-collateral/` — complete generated SoC (Rocket RISC-V core + FFT + uncore),
  507 SystemVerilog files, usable to rebuild the full simulation
- `sw/fft.c` — bare-metal test program (writes 8 points, reads back the transform, checks results)

## Software usage

Write the 8 input points one at a time to the write lane at `0x2400`
(each 32-bit write = one point: `[31:16]` imaginary, `[15:0]` real, both Q8.8 fixed point).
After the 8th write, read output point `i` from `0x2408 + i*8`.

Every RTL line carries a `// @[...]` comment pointing at the Chisel source line
(in [ucb-bar/FFTGenerator](https://github.com/ucb-bar/FFTGenerator)) that produced it.

## Acknowledgements

FFT generator code originates from UC Berkeley ADEPT Lab's Hydra Spine project
(authors: James Dunn, Stevo Bailey), adapted for Chipyard by the Chipyard team.
