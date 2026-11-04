# AXI4-Lite DMA & Register Controller

A synthesizable Verilog implementation of an **AXI4-Lite Peripheral** and **DMA Controller Engine**, verified with self-checking testbenches and SystemVerilog assertion checks.

---

## Key Specs
* **Bus Protocol:** AMBA AXI4-Lite (32-bit Memory-Mapped Interface)
* **DMA Features:** Auto-increment addressing, byte-enable masks, status polling, and interrupt requests (`IRQ`).
* **Error Handling:** Traps zero-length transfers and invalid address assertions.
* **Verification Scope:** 25+ self-checking test scenarios (100% pass rate).

---

## System Architecture

```text
  +------------------+         +--------------------------+
  |  AXI4-Lite Host  | <=====> | AXI4-Lite Register File  |
  +------------------+ (MMIO)  +--------------------------+
                                            ||
                                            \/
  +------------------+         +--------------------------+
  |  Target Memory   | <=====> |   DMA Control Engine     | ===> [ IRQ Pin ]
  +------------------+ (Data)  +--------------------------+


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


## License

See [LICENSE](LICENSE).
