# 🚀 TINY CORE

> A lightweight, modular **RISC-V processor** written in **SystemVerilog**, designed for learning, experimentation, and future scalability.

![Language](https://img.shields.io/badge/SystemVerilog-HDL-blue)
![ISA](https://img.shields.io/badge/ISA-RISC--V-green)

---

# 📖 Overview

**TINY CORE** is a custom RISC-V processor implementation built in SystemVerilog. The project is organized into reusable hardware modules such as the ALU, register file, control unit, immediate generator, and memory controller, making it easy to understand, modify, and extend.

The goal of this project is to provide a clean foundation for CPU architecture research, FPGA development, and educational purposes.

---

# ✨ Features

* ✅ Modular SystemVerilog design
* ✅ Custom RISC-V CPU Core
* ✅ Arithmetic Logic Unit (ALU)
* ✅ Register File
* ✅ Control Unit
* ✅ Immediate Generator
* ✅ Unified Memory Controller
* ✅ Top-Level Chip Integration
* ✅ Simulation Testbench
* ✅ Easy to extend with new instructions and peripherals

---

# 📂 Project Structure

```text
TINY_CORE/
│
├── alu.systemverilog
├── control_unit.systemverilog
├── custom_rv_core.systemverilog
├── imm_gen.systemverilog
├── regfile.systemverilog
├── unified_memory_controller.systemverilog
├── top_level_chip.systemverilog
├── tb_massive_soc.systemverilog
└── README.md
```

---

# 🏗 Architecture

```text
                 +----------------------+
                 |     Instruction      |
                 |        Memory        |
                 +----------+-----------+
                            |
                            v
                  +-------------------+
                  |   Control Unit    |
                  +---------+---------+
                            |
        +-------------------+-------------------+
        |                                       |
        v                                       v
+-------------------+                 +-------------------+
| Immediate Generator|                |   Register File   |
+---------+----------+                +---------+---------+
          |                                     |
          +----------------+--------------------+
                           |
                           v
                    +--------------+
                    |     ALU      |
                    +------+-------+
                           |
                           v
              +---------------------------+
              | Unified Memory Controller |
              +-------------+-------------+
                            |
                            v
                     Data Memory / I/O
```

---

# 🛠 Requirements

* SystemVerilog Simulator

  * ModelSim
  * QuestaSim
  * Verilator
  * Xcelium
  * Vivado Simulator (optional)

---

# ▶️ Running the Simulation

Compile all modules together with the testbench.

**Verilator**

```bash
verilator --cc *.systemverilog --exe tb_massive_soc.systemverilog
```

**ModelSim**

```bash
vlog *.systemverilog
vsim tb_massive_soc
run -all
```

---

# 🎯 Design Goals

* Clean and readable RTL
* Modular architecture
* Easy debugging
* Educational implementation
* FPGA-friendly design
* Future multicore expansion

---

# 🚀 Future Roadmap

* Pipeline implementation
* Branch prediction
* Instruction cache
* Data cache
* Interrupt support
* CSR registers
* Timer and UART peripherals
* AXI/AHB bus support
* RV32I compliance verification
* FPGA deployment
* Multi-core support
* Performance optimization

---

# 🤝 Contributing

Contributions, bug reports, feature requests, and pull requests are welcome. Feel free to fork the repository and help improve **TINY CORE**.
