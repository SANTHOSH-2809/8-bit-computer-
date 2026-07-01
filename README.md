```
                                                    █████╗     ██████╗ ██╗████████╗
                                                   ██╔══██╗    ██╔══██╗██║╚══██╔══╝
                                                   ╚█████╔╝    ██████╔╝██║   ██║   
                                                   ██╔══██╗    ██╔══██╗██║   ██║   
                                                   ╚█████╔╝    ██████╔╝██║   ██║   
                                                    ╚════╝     ╚═════╝ ╚═╝   ╚═╝   
```
 
# 8-Bit Harvard Architecture CPU
### Designed in Verilog HDL — Multi-Cycle • Custom ISA • FPGA-Ready

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![Verilog](https://img.shields.io/badge/Language-Verilog%20HDL-blue)
![FPGA](https://img.shields.io/badge/FPGA-Xilinx%20Spartan--7-orange)
![Architecture](https://img.shields.io/badge/Architecture-Harvard-green)
![Status](https://img.shields.io/badge/Status-Active%20Development-brightgreen)
![Vivado](https://img.shields.io/badge/Toolchain-Vivado%202025.1-red)
![RTL](https://img.shields.io/badge/Design-RTL-lightgrey)

<!-- DEMO GIF — record Vivado sim / FPGA running, convert to gif -->
<img src="PASTE_DEMO_GIF_LINK_HERE" alt="CPU running demo" width="80%">

</div>

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Architecture Comparison](#architecturecomparison)
- [Datapath Diagram](#datapath-diagram)
- [Instruction Set](#instruction-set)
- [Timing Diagram](#timing-diagram)
- [Bus Activity per Instruction](#bus-activity-per-instruction)
- [Repository Structure](#repository-structure)
- [Simulation Results](#simulation-results)
- [FPGA Implementation](#fpga-implementation)
- [Getting Started](#getting-started)
- [Example Programs](#example-programs)
- [Feature Comparison](#feature-comparison)
- [Project Progress](#project-progress)
- [Roadmap](#roadmap)
- [Statistics](#statistics)
- [Contributing](#contributing)
- [License](#license)

---

## 🚀 Overview

This project implements a fully custom 8-bit Harvard-inspired CPU from scratch in Verilog HDL, designed for FPGA implementation. Inspired by the simplicity of SAP-1, the processor extends the architecture with separate Instruction (IRAM) and Data (DRAM) memories, a 5-stage T-state control sequencer (T0–T4), an enhanced custom ISA, and modular RTL components.

Unlike a traditional Von Neumann processor, instructions and data are stored in independent memory banks, while a shared 8-bit internal bus efficiently interconnects the CPU modules. A Memory Address Register (MAR) stores both the memory address and memory selection, allowing seamless access to either memory during execution.

The processor features a hardwired Control Unit, Program Counter, Instruction Register, ALU, A/B Registers, Flag Register, and Output Register, working together to execute 16 custom instructions including arithmetic, logic, branching, bitwise count in accumulator, threshold detection, and shift operations. The ALU automatically updates the Carry, Zero, and Threshold flags for conditional execution.

**Operating Mode**

Three operating modes are supported:

-  **Programming Mode** — Load instructions into **IRAM** and data into **DRAM** directly from the FPGA switches.

-  **Execution Mode** — Execute programs starting from address `0x0`, with manual clock control, CPU reset, and automatic termination using the `HLT` instruction while preserving memory contents.

-  **Clock Speed Mode** — Select the CPU clock frequency:
    - `0` → **Slow Mode** (0.5 Hz) for step-by-step observation and debugging.
    - `1` → **Fast Mode** (50 Hz) for faster program execution.
      
-  Hardware Controls

| Control | Function |
|----------|----------|
| **MEM_SEL Switch** | Selects **IRAM** (`0`) or **DRAM** (`1`) during programming. |
| **Address Switches** | Select the 4-bit memory address. |
| **Data Switches** | Provide the 8-bit instruction or data value. |
| **Enter Button** | Stores the selected data into the selected memory address. |
| **CPU Reset Button** | Resets the CPU state while preserving IRAM and DRAM contents. |
| **RAM Reset Button** | Clears both IRAM and DRAM memories. |
| **Run/Clock Enable Switch** | Starts or pauses CPU execution. |

> [!NOTE]
> **Harvard-Inspired Architecture**
>
> This processor follows a **Harvard-inspired architecture** rather than a classical Harvard architecture. It features **separate Instruction RAM (IRAM)** and **Data RAM (DRAM)**, while both memories communicate over a **shared 8-bit system bus**.
>
> This hybrid approach preserves the advantages of separate instruction and data storage while reducing hardware complexity, simplifying bus routing, and minimizing FPGA resource utilization.
---

## 🛠️ Features

| Feature | Description |
|---------|-------------|
| **Harvard Architecture** | Separate instruction RAM (`iram`) and data RAM (`dram`) |
|  **Custom 16-Instruction ISA** | Includes `POPCNT` and `THRESHOLD` (not found in classic SAP-1 designs) |
|  **Multi-Cycle Control Unit** | 5 T-states (T0 Fetch → T1 Fetch → T2/T3/T4 Decode/Execute) |
|  **Condition Flags** | Carry, Zero, Threshold driving conditional jumps (`JC`, `JZ`) |
|  **Dual Seven-Segment Output** | Live output register + program counter display |
|  **In-System RAM Programming** | program instructions/data directly via FPGA switches before execution |
|  **FPGA-Ready** | Targets Xilinx Spartan-7 (`xc7s50csga324-1`) |
---

## Architecture

<!-- ARCHITECTURE BLOCK DIAGRAM — create in diagrams.net -->
<p align="center">
<img src="PASTE_ARCHITECTURE_DIAGRAM_LINK_HERE" alt="CPU block architecture" width="75%">
</p>



---
## 🏗️ Architecture Comparison

| Feature | 🖥️ Classical Von Neumann | ⚡ Classical Harvard | 📥 My Harvard-Inspired CPU |
|---------|---------------------------|----------------------|----------------------------|
| **Instruction Memory** | Shared with data | Dedicated | Dedicated **IRAM** |
| **Data Memory** | Shared with instructions | Dedicated | Dedicated **DRAM** |
| **Memory Organization** | Single unified memory | Separate memories | Separate memories |
| **Instruction Bus** | Shared | Dedicated | Shared internal bus |
| **Data Bus** | Shared | Dedicated | Shared internal bus |
| **Instruction Fetch** | Shared memory | IRAM only | IRAM only |
| **Data Access** | Shared memory | DRAM only | DRAM only |
| **Parallel Instruction & Data Access** | ❌ No | ✅ Yes | ❌ No |
| **Instruction Safety** | Code can be overwritten | Code isolated | Code isolated |
| **Self-Modifying Code** | Possible | Not supported | Not supported |
| **Program Capacity** | Shared with data | Dedicated program memory | Full IRAM reserved for instructions |
| **Data Capacity** | Reduced by program size | Dedicated data memory | Full DRAM reserved for data |
| **Memory Selection Logic** | Address-based | Separate buses | `MEM_SEL` + MAR |
| **Hardware Complexity** | ⭐ Low | ⭐⭐⭐ High | ⭐⭐ Moderate |
| **FPGA Resource Usage** | Low | High | Moderate |
| **Educational Value** | Basic computer architecture | High-performance architecture | Harvard concepts with simplified hardware |

> [!NOTE]
> **Harvard-Inspired Architecture**
>
> This processor follows a **Harvard-inspired architecture** rather than a classical Harvard architecture. It features **separate Instruction RAM (IRAM)** and **Data RAM (DRAM)**, while both memories communicate over a **shared 8-bit system bus**.
>
> This hybrid approach preserves the advantages of separate instruction and data storage while reducing hardware complexity, simplifying bus routing, and minimizing FPGA resource utilization.

## Datapath Diagram

<!-- DETAILED DATAPATH SCREENSHOT FROM VIVADO/SCHEMATIC -->
<p align="center">
<img src="PASTE_DATAPATH_IMAGE_LINK_HERE" alt="Datapath diagram" width="85%">
</p>

---

## Instruction Set

| Opcode | Mnemonic    | Hex   | Description                                  |
|:------:|:------------|:-----:|:----------------------------------------------|
| 0000   | `NOP`       | 0x0   | No operation                                  |
| 0001   | `LDA`       | 0x1   | Load A from data RAM address                  |
| 0010   | `STA`       | 0x2   | Store A to data RAM address                   |
| 0011   | `ADD`       | 0x3   | A = A + dRAM[addr]                            |
| 0100   | `SUB`       | 0x4   | A = A − dRAM[addr]                            |
| 0101   | `LDI`       | 0x5   | Load immediate 4-bit value into A              |
| 0110   | `JMP`       | 0x6   | Unconditional jump                             |
| 0111   | `JC`        | 0x7   | Jump if carry flag set                         |
| 1000   | `JZ`        | 0x8   | Jump if zero flag set                          |
| 1001   | `OUT`       | 0x9   | Output A to 7-segment display                  |
| 1010   | `POPCNT`    | 0xA   | Count set bits in A                            |
| 1011   | `AND_OP`    | 0xB   | A = A & dRAM[addr]                             |
| 1100   | `OR_OP`     | 0xC   | A = A \| dRAM[addr]                            |
| 1101   | `THRESHOLD` | 0xD   | Set threshold flag if popcount(A) ≥ 5          |
| 1110   | `SHL`       | 0xE   | Shift A left by 1                              |
| 1111   | `HLT`       | 0xF   | Halt execution                                 |

<!-- COLORFUL ISA TABLE WITH ICONS — optional image version -->
<p align="center">
<img src="PASTE_ISA_TABLE_IMAGE_LINK_HERE" alt="Instruction set table" width="70%">
</p>

---

## Timing Diagram

Each instruction executes across up to 5 T-states:

```
T0 — Fetch:    PC  -----> BUS -----> MAR
T1 — Fetch:    RAM -----> BUS -----> IR        (PC increments)
T2 — Decode:   opcode-dependent micro-op begins
T3 — Execute:  operand fetch / ALU input staging
T4 — Execute:  ALU result written back, flags updated
```

<!-- WAVEFORM / TIMING SCREENSHOT FROM VIVADO -->
<p align="center">
<img src="PASTE_TIMING_DIAGRAM_IMAGE_LINK_HERE" alt="Timing diagram" width="85%">
</p>

---

## Bus Activity per Instruction

```
LDA / STA / ADD / SUB / AND / OR:
  T0: PC  --> BUS --> MAR
  T1: RAM --> BUS --> IR          (CE: PC++)
  T2: IR(operand) --> BUS --> MAR
  T3: RAM --> BUS --> B  (or A --> BUS --> RAM for STA)
  T4: ALU --> BUS --> A           (flags updated)

LDI:
  T0: PC --> BUS --> MAR
  T1: RAM --> BUS --> IR          (CE: PC++)
  T2: IR(operand) --> BUS --> A

JMP / JC / JZ:
  T0: PC --> BUS --> MAR
  T1: RAM --> BUS --> IR          (CE: PC++)
  T2: IR(operand) --> BUS --> PC  (conditional on flag)

OUT:
  T0: PC --> BUS --> MAR
  T1: RAM --> BUS --> IR          (CE: PC++)
  T2: A --> BUS --> OUT_REG
```

---

## Repository Structure

```
8bit-harvard-cpu/
│
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── CHANGELOG.md
│
├── docs/
│   ├── architecture.md
│   ├── instruction_set.md
│   ├── control_signals.md
│   ├── datapath.md
│   ├── timing_diagram.md
│   ├── memory_map.md
│   └── developer_notes.md
│
├── rtl/
│   ├── top_module.v
│   ├── cpu_top.v
│   ├── alu.v
│   ├── control_unit.v
│   ├── pc.v
│   ├── mar.v
│   ├── ir.v
│   ├── reg_a.v
│   ├── reg_b.v
│   ├── ram.v
│   ├── bus_mux.v
│   ├── flag_register.v
│   ├── output_register.v
│   ├── t_state_counter.v
│   ├── clock_divider.v
│   └── seven_seg_driver.v
│
├── sim/
│   ├── tb_top_module.v
│   ├── waveforms/
│   └── test_programs/
│
├── images/
│   ├── architecture.png
│   ├── datapath.png
│   ├── timing.png
│   ├── demo.gif
│   └── fpga.jpg
│
├── programs/
│   ├── addition.asm
│   ├── threshold.asm
│   ├── popcnt.asm
│   └── conditional_jump.asm
│
└── fpga/
    ├── top_module.xdc
    ├── vivado_project/
    └── bitstream/
```

---

## Simulation Results

<!-- Add one screenshot row per test case -->

### Addition
<img src="PASTE_ADDITION_WAVEFORM_LINK_HERE" alt="Addition waveform" width="80%">

### Subtraction
<img src="PASTE_SUBTRACTION_WAVEFORM_LINK_HERE" alt="Subtraction waveform" width="80%">

### POPCNT
<img src="PASTE_POPCNT_WAVEFORM_LINK_HERE" alt="POPCNT waveform" width="80%">

### THRESHOLD
<img src="PASTE_THRESHOLD_WAVEFORM_LINK_HERE" alt="Threshold waveform" width="80%">

### Conditional Jump (JC / JZ)
<img src="PASTE_JUMP_WAVEFORM_LINK_HERE" alt="Conditional jump waveform" width="80%">

### Store / Load
<img src="PASTE_STORE_LOAD_WAVEFORM_LINK_HERE" alt="Store/Load waveform" width="80%">

---

## FPGA Implementation

**Target Board:** Xilinx Spartan-7 — `xc7s50csga324-1`

<!-- FPGA HARDWARE PHOTOS -->
<p align="center">
<img src="PASTE_FPGA_BOARD_IMAGE_LINK_HERE" alt="FPGA board" width="45%">
<img src="PASTE_SEVENSEG_DISPLAY_IMAGE_LINK_HERE" alt="Seven segment display running" width="45%">
</p>

<p align="center">
<img src="PASTE_PROGRAMMING_SWITCHES_IMAGE_LINK_HERE" alt="Programming switches" width="45%">
<img src="PASTE_RUNNING_CPU_IMAGE_LINK_HERE" alt="CPU running on FPGA" width="45%">
</p>

### Resource Utilization

<!-- Paste Vivado utilization report screenshot -->
<p align="center">
<img src="PASTE_RESOURCE_UTILIZATION_IMAGE_LINK_HERE" alt="Resource utilization" width="70%">
</p>

---

## Getting Started

### Prerequisites
- Xilinx Vivado 2025.1 (or compatible)
- Spartan-7 FPGA board (`xc7s50csga324-1`) — or simulate without hardware

### Simulation

```bash
# Open the project in Vivado
open_project 8_bit.xpr

# Run behavioral simulation
launch_simulation
run 50ms
```

### Synthesis & Implementation

```bash
launch_runs synth_1 -jobs 10
launch_runs impl_1 -jobs 10
launch_runs impl_1 -to_step write_bitstream -jobs 10
```

### Programming the FPGA

1. Connect your board via JTAG
2. Open Hardware Manager
3. Program device with generated `.bit` file
4. Use `prog_mode` / `prog_we` switches to load a program into RAM
5. Toggle `clk_enable` to run

---

## Example Programs

**`programs/addition.asm`**
```asm
LDI  1        ; A = 1
LDI  3        ; A = 3
ADD  [1]      ; A = A + dRAM[1]
OUT           ; display result
HLT
```

**`programs/threshold.asm`**
```asm
LDI  15       ; A = 0xF
THRESHOLD     ; set threshold flag if popcount(A) >= 5
OUT
HLT
```

**`programs/popcnt.asm`**
```asm
LDI  15       ; A = 0xF (1111)
POPCNT        ; A = number of 1-bits
OUT
HLT
```

---

## Feature Comparison

| Feature                  | SAP-1 | This CPU |
|---------------------------|:-----:|:--------:|
| Harvard Architecture       | ❌    | ✅       |
| POPCNT instruction         | ❌    | ✅       |
| Threshold instruction      | ❌    | ✅       |
| Conditional Jumps (JC/JZ)  | ❌    | ✅       |
| Dual 7-Segment Output      | ❌    | ✅       |
| Separate I/D Memories      | ❌    | ✅       |
| Multi-Cycle Control        | ✅    | ✅       |
| FPGA Ready                 | ❌    | ✅       |
| LED Microinstruction Debug | ❌    | ✅       |

---

## Project Progress

```
CPU Core         ██████████ 100%
ALU              ██████████ 100%
Memory           ██████████ 100%
Control Unit     ██████████ 100%
FPGA Testing     ████████░░  80%
Assembler        ██░░░░░░░░  20%
```

---

## Roadmap

**Version 1.0**
- ✅ Harvard Architecture
- ✅ Custom ALU (8 operations)
- ✅ FPGA deployment
- ✅ Dual seven-segment output
- ✅ Conditional jumps with flags

**Version 2.0**
- ⬜ Stack Pointer
- ⬜ `CALL` / `RET` instructions
- ⬜ UART communication
- ⬜ Interrupt handling
- ⬜ Pipelined datapath
- ⬜ Custom assembler / toolchain

---

## Statistics

| Metric              | Value             |
|----------------------|-------------------|
| Lines of Verilog      | 3000+             |
| Modules               | 17                |
| Instructions          | 16                |
| ALU Operations        | 8                 |
| Clock Cycles per Instr| Up to 5 (T0–T4)   |
| Architecture           | Harvard           |
| Target FPGA            | Xilinx Spartan-7  |
| Toolchain               | Vivado 2025.1     |

---

## Contributing

Contributions, bug reports, and feature suggestions are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

---

<div align="center">

**Built with Verilog HDL — From Gates to a Working Processor**

</div>
