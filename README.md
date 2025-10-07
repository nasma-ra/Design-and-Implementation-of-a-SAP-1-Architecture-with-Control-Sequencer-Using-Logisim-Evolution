# SAP-1 Architecture in Logisim

## Table of Contents
Click on the Table of Contents below to directly go to the sections:
- [Video Tutorial](#video-tutorial)
- [Project Overview](#project-overview)
- [Objectives](#objectives)
- [Key Features](#key-features)
- [Architecture and Functional Block Analysis](#architecture-and-functional-block-analysis)
  - [System Architecture Overview](#system-architecture-overview)
  - [Register Implementation (A, B)](#register-implementation-a-b)
  - [Program Counter (PC) Implementation](#program-counter-pc-implementation)
  - [Memory System and Address Register](#memory-system-and-address-register)
  - [Instruction Register and Opcode Decoder](#instruction-register-and-opcode-decoder)
  - [Arithmetic Logic Unit (ALU) Implementation](#arithmetic-logic-unit-alu-implementation)
  - [Boot/Loader Counter and Phase Generation](#bootloader-counter-and-phase-generation)
- [Control System Design](#control-system-design)
  - [Timing Control Generator](#timing-control-generator)
  - [Automatic Operation Control Logic](#automatic-operation-control-logic)
  - [Manual/Loader Operation Control](#manualloader-operation-control)
- [Instruction Set Architecture](#instruction-set-architecture)
  - [Instruction Encoding Scheme](#instruction-encoding-scheme)
  - [Assembler](#assembler)
- [Operation](#operation)
  - [Fetch–Decode–Execute Cycle](#fetchdecodeexecute-cycle)
  - [Running the CPU in Automatic Mode (COMPARE + ADD Program)](#running-the-cpu-in-automatic-mode-compare--add-program)
- [Future Improvement](#future-improvement)
- [Conclusion](#conclusion)

## Video Tutorial
Watch the complete demonstration of the *SAP-1 CPU with Control Sequencer (Manual and Automatic Modes)* implemented in *Logisim Evolution*.  
Click the image below to watch the video on YouTube:

[![SAP-1 CPU Tutorial]([(https://youtu.be/UR6E0FEg06Q?feature=shared))

## Project Overview

This project implements an **enhanced 8-bit SAP-1 computer** in **Logisim Evolution** with a **hardwired control sequencer** and extended instructions like **LDA, LDB, ADD, COMPARE, HLT**.  
It supports **Automatic Mode** (fetch–decode–execute cycle) and **Manual/Loader Mode** for safe program transfer into memory.  
A **control sequencer** manages the bus and timing, while a **web-based assembler** translates assembly code into Logisim-compatible machine code.  
The project includes **instruction verification** through test programs to ensure correct execution of instructions and memory operations.

---

## Objectives

- Develop an **8-bit SAP-1 computer** in **Logisim Evolution** for educational purposes and architectural analysis.  
- Implement a **single-bus architecture** with **8-bit data paths**, **4-bit address space (16 bytes)**, and a **hardwired control sequencer** managing the **fetch–decode–execute cycle**.  
- Support both **Automatic Mode** and **Manual/Loader Mode** for safe program loading and execution.  
- Design the **datapath** with **dual 8-bit registers** (A and B), **ripple-carry ALU** (ADD/SUB), **4-bit program counter**, **16×8 SRAM**, and an **Instruction Register** with separate opcode and operand handling.  
- **Strict bus management** to ensure no contention and safe data transfers across components.

---

## Key Features

- **8-bit CPU Architecture:** Implements a classical **SAP-1 design** with **8-bit data** and **4-bit address buses**.  
- **Manual & Automatic Modes:** Step-by-step execution in Manual Mode and continuous execution in Automatic Mode.  
- **Dual Register Set:** **A and B registers** for ALU operations and temporary data storage.  
- **Program Counter (PC):** Automatically increments for sequential instruction fetching.  
- **Memory System:** A combination of **ROM for program storage** and **SRAM for data storage**, with **MAR** for memory address management.  
- **Instruction Register & Opcode Decoder:** Splits the instruction into **opcode** and **operand** for proper ALU operations.  
- **Arithmetic Logic Unit (ALU):** Performs **addition** and **subtraction** on the accumulator register.  
- **Control Logic:** Features **Timing Control Generator** and **phase-based operation sequencing** for reliable instruction execution.  
- **Expandable Instruction Set:** Includes **LDA, LDB, ADD, COMPARE, HLT** with potential for expansion.  
- **Educational Tool:** Ideal for teaching **CPU architecture**, **control sequencing**, and **instruction cycles**.

---

## Architecture and Functional Block Analysis

### System Architecture Overview
The **system architecture** follows a **single-bus design** with **8-bit data paths**. The bus arbitration mechanism ensures that only **one driver** is active during each T-state, with possible drivers including `pc_out`, `sram_rd`, `ins_reg_out_en`, `a_out`, `b_out`, `alu_out`, and `sh_out`. Bus listener components such as `mar_in_en`, `ins_reg_in_en`, `a_in`, `b_in`, and `sram_wr` allow selective data capture when required.

![Automatic Mode Control Sequencer](images/add.png)  
*Figure 1:* Automatic mode operation of the control sequencer showing fetch–decode–execute sequencing.  

![Manual/Loader Mode Control Sequencer](images/manual.pngg)  
*Figure 2:* Manual/Loader mode operation of the control sequencer showing secure program loading with debug and handshake signals.  

---

### Register Implementation (A, B)
The **A and B registers** use **D flip-flops** to store **8-bit data**. These registers have three main interfaces:
1. **Input Interface:** Controlled by signals `a_in` and `b_in`, connected to the bus for data entry.  
2. **Output Interface:** Uses tri-state logic to output data to the bus when `a_out` or `b_out` is enabled.  
3. **Internal Interface:** Direct access to the **ALU** through `reg_int_out`.

![A/B Register Subsystem](images/ins_reg.jpg)  
*Figure 3:* A/B register subsystem with input, output, and internal interfaces.  

---

### Program Counter (PC) Implementation
The **Program Counter (PC)** handles instruction address sequencing:
- **Increment Mode:** The PC increments automatically on every clock cycle (T3) to fetch the next instruction.  
- **Jump Mode:** The `JMP` instruction allows the PC to directly load an address from the Instruction Register (IR) during T4.

**PC bus interfacing:** At **T1**, the PC value is transferred to the **MAR** to start the instruction fetch cycle.

![Program Counter Direct Load](images/lda_ldb_sum.jpg)  
*Figure 5:* Program Counter direct load and sequential functionality.

---

### Memory System and Address Register
The **Memory Address Register (MAR)** captures addresses from the bus controlled by `mar_in_en`.  
- During **instruction fetch (T1)**, the **Program Counter** value is loaded into the MAR.  
- During **operand addressing**, the operand address (IR[3:0]) is loaded into the MAR for memory access.

The **SRAM** handles both read and write operations based on control signals:  
- **Read Mode:** During **T1** (instruction fetch) and **T2** (LDA/LDB).  

![Memory Element](images/data_in_sram.jpg)  
*Figure 6:* Register-based memory element showing data_in, wr_en, rd_en, clock, and chip select (cs) signals. Output to the bus is via data_out.  

---

### Instruction Register and Opcode Decoder
The **Instruction Register (IR)** is used to store the fetched instruction and decode it:
- **Opcode Handling:** The opcode (IR[7:4]) is sent to the **opcode decoder** (`ins_tab`), which generates one-hot control signals for the corresponding instruction.  
- **Operand Handling:** The lower nibble (IR[3:0]) is placed on the bus and transferred to the **MAR** when required (e.g., LDA, LDB, STA, JMP).

The **opcode decoder** uses a **4-to-16** decoder scheme, activating specific instruction lines based on the opcode.

---

### Arithmetic Logic Unit (ALU) Implementation
The **ALU** performs **8-bit arithmetic operations** such as **addition** and **subtraction**:
- Operands come directly from the **A** and **B registers**.  
- **Subtraction (A - B)** is controlled by the `alu_sub` signal.
- The **ALU output** drives the bus when `alu_out` is enabled.

![ALU Implementation](images/alu.jpg)  
*Figure 9:* ALU implementation with ripple-carry architecture and tri-state bus interface.

---

### Boot/Loader Counter and Phase Generation
The **boot/loader counter** transfers program data from ROM to RAM in **Manual/Loader Mode**:
- **Counter:** A 4-bit counter generates addresses for sequential memory writes.  
- **Phase Generation:** The counter uses two clock phases (Φ and ¬Φ) to synchronize program loading.

![Boot/Loader Subsystem](images/add_load.jpg)  
*Figure 10:* Boot/loader subsystem with sequential address generation and dual-phase clocking.  

---

## Control System Design

The **control unit** translates instructions into precisely timed control pulses to:
1. Ensure exclusive bus driver activation.  
2. Enable appropriate latch operations during each T-state.  

Key components:
- **Ring Counter (RC):** Generates timing states T1–T6.  
- **Opcode Decoder:** Produces activation lines for instruction-specific micro-operations.  
- **Mode Control Inputs:** `debug` (manual/loader selection), `i1/i2` (loader handshake) defining CPU mode (~debug with loader masking via ~i2).  

---

### Timing Control Generator
The **Timing Control Generator** manages **six-phase sequencing** for fetch–decode–execute operations:
- **Fetch Phase:** PC drives MAR, IR loads instruction.  
- **Execute Phase:** Operands from MAR and IR are used for ALU or memory operations.

---

### Automatic Operation Control Logic
- **Automatic Mode:** Uses control signals generated from the ring counter and opcode decoder to enable the fetch–decode–execute cycle for each instruction.  
- **Manual Mode:** The `debug` pin disables automatic execution, allowing for stepwise program loading and debugging.

---

## Instruction Set Architecture

The **Instruction Set Architecture (ISA)** defines the set of operations the SAP-1 processor can perform, including both the opcodes (instructions) and the addressing methods for operands. In this project, the SAP-1 processor supports a set of 5 basic instructions, with a focus on educational simplicity.

### 1. Instruction Encoding Scheme

Each instruction consists of two main parts:

- **Opcode (4 bits):** The first 4 bits in the instruction represent the operation to be executed (e.g., Load, Add, Compare).
- **Operand/Address (4 bits):** The last 4 bits represent the address or operand associated with the instruction. In this simple design, this can be a memory address or a direct operand (depending on the instruction type).

For example:
- **LDA** (Load Accumulator) loads data from a specific memory location into the accumulator register.
- **ADD** adds the contents of memory or a register to the accumulator.

The instruction format in binary would look like:

|  Opcode  | Operand/Address |
|  4 bits  |   4 bits        |

Here’s an example:


### 2. Instruction Set

The **SAP-1 processor** in this project supports the following instructions, each of which has a specific binary opcode:

| **Opcode (Binary)** | **Instruction** | **Description**                        |
|---------------------|-----------------|----------------------------------------|
| `0001`              | LDA (Load Accumulator) | Loads data from memory to register A.  |
| `0010`              | LDB (Load Register B) | Loads data from memory to register B.  |
| `0011`              | ADD (Add)       | Adds contents of register B to A.      |
| `1101`              | COMPARE         | Compares contents of registers A and B.|
| `1111`              | HLT (Halt)      | Stops the execution.                   |

These instructions are used in a simple **fetch–decode–execute cycle** to perform basic arithmetic and control tasks.

---

### 3. Instruction Details

#### **LDA (Load Accumulator)**

- **Description**: This instruction loads the value from the memory address specified by the operand into the **Accumulator (A)**.
- **Opcode**: `0001`
- **Example**: `LDA 8` – Loads the value at memory location 8 into the accumulator.

#### **LDB (Load Register B)**

- **Description**: This instruction loads the value from the memory address specified by the operand into **Register B**.
- **Opcode**: `0010`
- **Example**: `LDB 9` – Loads the value at memory location 9 into register B.

#### **ADD (Addition)**

- **Description**: Adds the contents of **Register B** to the **Accumulator (A)** and stores the result back in **Accumulator A**.
- **Opcode**: `0011`
- **Example**: `ADD` – Adds the contents of B to A (A = A + B).

#### **COMPARE**

- **Description**: Compares the values in **Register A** and **Register B** and sets flags (e.g., less than, equal, greater than) accordingly.
- **Opcode**: `1101`
- **Example**: `COMPARE` – Compares the values of A and B and sets the respective comparison flags.

#### **HLT (Halt)**

- **Description**: Stops the processor execution.
- **Opcode**: `1111`
- **Example**: `HLT` – Halts the execution of the processor.

---

### 4. Instruction Format
Each instruction is **8 bits** in length:
- **Opcode** (4 bits): Specifies the operation to be performed.
- **Operand/Address** (4 bits): Represents the address in memory or the data for the operation.

#### Example Instruction:
Instruction: `LDA 8`


---

### 5. Assembler
A **custom Python-based assembler** was created for converting **assembly language** instructions into machine-readable **hexadecimal format**. The assembler performs the following tasks:

- **Tokenization**: The assembler reads the human-readable assembly source code, extracting mnemonics (instructions), operands, and comments.
- **Opcode Translation**: Each mnemonic is mapped to its corresponding 4-bit opcode (e.g., `LDA → 0001`).
- **Memory Address Parsing**: The operand is parsed and translated into a 4-bit address, forming the complete 8-bit machine instruction.
- **Output**: The assembler generates a list of **hexadecimal machine codes**, ready for use in the Logisim simulation.

#### Example Assembly Code:
```assembly
LDA 8
LDB 9
ADD
COMPARE
HLT
---
#### Assembler
The **Assembler** converts SAP-1 assembly language instructions into machine code:
- **Tokens**: Translates mnemonics like `LDA`, `ADD`, `STA` into **4-bit opcodes** and **4-bit operands**.
- **Output**: Generates machine code in **hexadecimal** format suitable for Logisim.

---

## Operation

### Fetch–Decode–Execute Cycle

The processor operates by continuously fetching, decoding, and executing instructions in a cyclic manner:
- **Fetch**: The program counter value is loaded into the **MAR**, and the instruction is fetched into the **IR**.  
- **Decode**: The opcode is decoded, and control signals are generated for the ALU or memory operation.  
- **Execute**: The instruction is carried out, with data transferred between registers and memory as needed.

---

### Running the CPU in Automatic Mode (COMPARE + ADD Program)

The **Automatic Mode** allows continuous execution of a program using a clock pulse:
1. **Program Loading**: The program is first loaded into **RAM** using **Manual Mode**.  
2. **Execution**: The CPU automatically fetches and executes each instruction in sequence, including operations like **ADD**, **COMPARE**, etc.

---

## Conclusion

This project successfully implements a **functional SAP-1 processor** with **automatic and manual modes**. It serves as an educational tool for learning **CPU architecture**, **control logic**, and **instruction execution**. The design can be further expanded to include more complex instructions and features, making it a great foundation for further CPU projects.
