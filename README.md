# AXI4-Lite Peripheral & DMA Controller with Self-Checking Testbenches

Verilog implementations of a four-register AXI4-Lite peripheral and a DMA control model, with separate self-checking testbenches.

The DMA model exposes source, destination, length, and control/status registers. A countdown models busy-to-done timing and drives an enabled completion interrupt. It does not implement memory-to-memory data movement.

## Source map

| File | Purpose |
| --- | --- |
| [`rtl/axi_lite_slave.v`](rtl/axi_lite_slave.v) | Four-register peripheral with byte-enable writes |
| [`UVM_tb`](UVM_tb) | Procedural register-file testbench, reference model and coverage flags |
| [`DMA_Front_End_Design/axi_lite_dma`](DMA_Front_End_Design/axi_lite_dma) | DMA register interface and completion model |
| [`DMA_Front_End_Design/tb_axi_lite_dma`](DMA_Front_End_Design/tb_axi_lite_dma) | DMA register, status, error and interrupt checks |
| [`waveforms/`](waveforms/) | Saved waveforms and screenshots |
| [`Post_Implementation_screenshots/`](Post_Implementation_screenshots/) | Saved implementation views |

The historical filename `UVM_tb` is retained. The published tests are procedural HDL testbenches and do not use the UVM library.

## Run the existing tests

Requires Icarus Verilog (`iverilog` and `vvp`). From the repository root:

```sh
iverilog -g2012 -s tb_axi_lite -o slave.sim rtl/axi_lite_slave.v UVM_tb
vvp slave.sim

iverilog -g2012 -s tb_axi_lite_dma -o dma.sim DMA_Front_End_Design/axi_lite_dma DMA_Front_End_Design/tb_axi_lite_dma
vvp dma.sim
```

The DMA test writes `tb_axi_lite_dma.vcd`, which can be opened in GTKWave. Inspect the printed scoreboard and timeout messages; the current benches do not reliably encode failures in the process exit status.

## Reproduced simulation results

The published source was run with Icarus Verilog on 20 September 2026.

| Testbench | Scoreboard | Explicit coverage flags |
| --- | --- | --- |
| Register file | 14 passed, 0 failed | 15/15 hit |
| DMA control | 11 passed, 0 failed | 9/9 hit |

Register-file coverage tracks seven write-strobe patterns and read/write access to four registers. Stimulus includes directed and pseudo-random writes.

DMA coverage tracks source/destination/length programming, start, busy, done, IRQ, zero-length error, and back-to-back operation. Its stimulus is directed.

These results apply to the checks implemented in the benches. They are not comprehensive protocol coverage or a compliance certification.

## DMA register map

| Offset | Register | Meaning |
| --- | --- | --- |
| `0x00` | `SRC_ADDR` | Stored source address |
| `0x04` | `DST_ADDR` | Stored destination address |
| `0x08` | `XFER_LEN` | Countdown length in the current model |
| `0x0C` | `CTRL_STAT` | Bit 0: start; 1: busy; 2: done; 3: error; 4: IRQ enable |

## Current limits

- No DMA read/write master datapath or transferred-data checks.
- The DMA front-end does not apply `wstrb` masks to its register writes; byte-enable support in the separate register-file peripheral should not be attributed to this module.
- Backpressure, independent channel timing and reset interruption need broader tests before claiming full protocol coverage.
- Saved implementation images are available, but the repository does not include a complete scripted implementation flow and timing constraints for reproduction.

## License

See [LICENSE](LICENSE).
