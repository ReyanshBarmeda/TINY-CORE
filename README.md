# TINY CORE

A multi-core RISC-V (RV64) SoC implemented in SystemVerilog — 1,000 single-cycle-style cores sharing one unified memory, all connected through a round-robin memory arbiter.

## Architecture.

```
top_level_chip
 ├── custom_rv_core  x 1000   (genvar-generated array, core_id 0-999)
 │    ├── control_unit
 │    ├── imm_gen
 │    ├── regfile
 │    └── alu
 └── unified_memory_controller  (round-robin arbiter over all 1000 cores)
```

Cores connect to memory through a shared `mem_bus_if` interface: a 40-bit address, 512-bit read/write data, and read/write/ready handshake signals.

## Modules.

| File | Description |
|---|---|
| `top_level_chip.systemverilog` | Top-level module. Defines `mem_bus_if` and instantiates 1,000 `custom_rv_core` instances plus the `unified_memory_controller`. |
| `custom_rv_core.systemverilog` | Single RISC-V core. Wires together the ALU, register file, control unit, and immediate generator behind a 2-state (FETCH/EXECUTE) FSM. |
| `control_unit.systemverilog` | Decodes the 32-bit instruction (opcode/funct3/funct7) into control signals for R-type, I-type, loads, stores, and branches. |
| `imm_gen.systemverilog` | Extracts and sign-extends immediates for I/S/B/U/J instruction formats. |
| `regfile.systemverilog` | 32 x 64-bit register file; x0 hardwired to zero. |
| `alu.systemverilog` | 64-bit ALU: ADD, SUB, AND, OR, XOR, SLL, SRL, SRA. |
| `unified_memory_controller.systemverilog` | Round-robin (SCAN -> ACCESS -> FINISH) arbiter giving all 1,000 cores fair access to a single 512-bit memory bus, with a simulated 5-cycle access latency. |
| `tb_massive_soc.systemverilog` | Testbench for the full SoC. |
| `matrix_mult.systemverilog` | Standalone N x N matrix multiplication accelerator (single shared MAC unit, FSM-sequenced). Not yet wired into the SoC. |
| `tb_matrix_mult.systemverilog` | Self-checking testbench for `matrix_mult`. |

## Instruction Support

Currently decoded by the control unit / immediate generator:

- **R-type:** ADD, SUB, AND, OR
- **I-type:** ADDI, ANDI
- **Loads/Stores:** LD/LW-style, SD/SW-style (base + offset addressing)
- **Branches:** BEQ-style (via ALU SUB + zero flag)
- **Immediate formats:** I, S, B, U (LUI/AUIPC), J (JAL) are generated in `imm_gen`, though not all are yet wired into `control_unit`

## Core Execution Model

Each core runs a simple 2-state FSM:

1. **FETCH** - requests the instruction at `pc` over the memory bus, waits for `mem.ready`, latches the low 32 bits of the 512-bit response.
2. **EXECUTE** - runs the decoded instruction. For loads/stores, issues a second memory transaction using the ALU-computed address; for ALU/branch ops, resolves immediately and returns to FETCH.

## Memory

- Single unified memory space, addressed with 40 bits, accessed in 512-bit lines.
- Shared across all 1,000 cores via `unified_memory_controller`, which scans cores round-robin and grants one at a time, simulating a 5-cycle latency per access.

## Matrix Multiplier Accelerator

`matrix_mult.systemverilog` adds a standalone `N x N` fixed-point matrix multiplication engine, since the base ALU has no `MUL` opcode and can't do this in software on the cores as-is.

- **How it works:** a single shared multiply-accumulate (MAC) unit runs an `i/j/k` FSM (`IDLE -> COMPUTE -> ADVANCE -> DONE`), doing one MAC per cycle, so a full multiply takes on the order of `N^3` cycles. It trades speed for area - no parallel multiplier array.
- **Interface:** matrices `A` and `B` are loaded one element at a time (`load_a`/`load_b` + row/col address + data), a `start` pulse kicks off the computation, `done` pulses for one cycle when `C` is ready, and `C` is read back the same element-at-a-time way via `read_row`/`read_col`.
- **Data width:** 64-bit signed elements, matching the core register width, with extra internal accumulator headroom to reduce (but not eliminate) overflow risk on larger matrices.
- **Status:** standalone - it is not yet wired into `top_level_chip.systemverilog`. To make it usable from the cores it would need to be added as a memory-mapped peripheral on the unified memory bus (translating core reads/writes in a reserved address range into `load_*`/`read_*` pulses), rather than given its own dedicated port.
- **Testbench:** `tb_matrix_mult.systemverilog` loads a fixed 3x3 example, computes the expected result in the testbench itself, and reports a match/mismatch per output element.

## Known Issues

- `unified_memory_controller.systemverilog` has a stray `4` on its own line (line 7, right after the header comment) - this will not compile as-is and should be removed.
- Only a subset of the RV64I instruction set is decoded (see Instruction Support above); U-type and J-type immediates are generated but not yet consumed by `control_unit`.
- `matrix_mult`'s accumulator has extra headroom but is not fully overflow-checked; widen it further before using large matrices or large element magnitudes.

## Simulation

- `tb_massive_soc.systemverilog` - top-level testbench for the full 1,000-core SoC.
- `tb_matrix_mult.systemverilog` - standalone, self-checking testbench for the matrix multiplier.
