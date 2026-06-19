# 8-Bit Harvard Architecture CPU in Verilog

## Project Overview

This project implements an 8-bit Harvard Architecture inspired CPU using Verilog HDL. Unlike traditional SAP-1 style processors that use a single memory and a limited instruction set, this design utilizes separate instruction and data memories, enabling a cleaner memory architecture and improved scalability.

The processor executes instructions using a multi-cycle control unit based on T-state sequencing. Each instruction progresses through fetch, decode, and execute phases using a dedicated timing counter and control logic.

## Key Features

* 8-bit datapath
* Harvard Architecture

  * 16×8 Instruction Memory
  * 16×8 Data Memory
* 4-bit Program Counter
* 4-bit Memory Address Register (MAR)
* Instruction Register (IR)
* Accumulator Register (A)
* Temporary Register (B)
* Multi-cycle Control Unit
* Flag Register

  * Carry Flag
  * Zero Flag
  * Threshold Flag
* External 8-bit Input Port
* 7-Segment Display Output
* Custom ALU Operations

## Supported Instructions

| Opcode | Instruction | Description                              |
| ------ | ----------- | ---------------------------------------- |
| 0000   | NOP         | No Operation                             |
| 0001   | LDA         | Load accumulator from Data RAM           |
| 0010   | STA         | Store accumulator to Data RAM            |
| 0011   | ADD         | Add memory operand to accumulator        |
| 0100   | SUB         | Subtract memory operand from accumulator |
| 0101   | LDI         | Load immediate value                     |
| 0110   | JMP         | Unconditional jump                       |
| 0111   | JC          | Jump if carry flag is set                |
| 1000   | JZ          | Jump if zero flag is set                 |
| 1001   | OUT         | Send accumulator to output register      |
| 1010   | POPCNT      | Count number of set bits in A            |
| 1011   | AND         | Bitwise AND                              |
| 1100   | OR          | Bitwise OR                               |
| 1101   | THRESHOLD   | Generate threshold-based result          |
| 1110   | SHL         | Logical left shift                       |
| 1111   | HLT         | Halt processor                           |

## ALU Operations

| ALU_SEL | Operation |
| ------- | --------- |
| 000     | ADD       |
| 001     | SUB       |
| 010     | AND       |
| 011     | OR        |
| 100     | SHL       |
| 101     | POPCNT    |
| 110     | THRESHOLD |
| 111     | PASS      |

### POPCNT

Counts the number of logic '1' bits present in the accumulator.

Example:

A = 11110010

POPCNT Result = 5

### THRESHOLD

Checks whether the number of set bits in the accumulator is greater than or equal to 5.

Example:

A = 11110010

Number of Ones = 5

THRESHOLD Result = 1

## Processor Architecture

### Core Components

* Program Counter (PC)
* Memory Address Register (MAR)
* Instruction Register (IR)
* Instruction Memory
* Data Memory
* Accumulator Register (A)
* Register B
* Arithmetic Logic Unit (ALU)
* Flag Register
* Bus Multiplexer
* Control Unit
* T-State Counter

### Memory Organization

Instruction Memory:

* 16 Locations
* 8-bit Width

Data Memory:

* 16 Locations
* 8-bit Width

The architecture follows Harvard principles where instruction and data memories are physically separated.

## Instruction Execution

### Fetch Cycle

T0:

* PC → MAR

Control Signals:

* CO
* MI

T1:

* Instruction Memory → IR
* PC Increment

Control Signals:

* RO
* II
* CE

### Example: ADD addr

T2:

* Operand → MAR

Control Signals:

* IO
* MI

T3:

* Data Memory → Register B

Control Signals:

* RO
* BI

T4:

* ALU Result → Accumulator
* Flags Updated

Control Signals:

* EO
* AI
* FI

## Timing System

The CPU uses a dedicated T-State Counter.

T0 → T1 → T2 → T3 → T4

When an instruction completes, the control unit asserts:

clr_t = 1

This resets the timing sequence back to T0, starting the fetch cycle of the next instruction.

The processor is therefore a:

* Multi-Cycle Processor
* Non-Pipelined Architecture

## Input and Output

### Input

External 8-bit switch interface.

Instruction:

IN

Operation:

Input Port → Bus → Accumulator

### Output

7-Segment Display Interface.

Instruction:

OUT

Operation:

Accumulator → Output Register → Display

## Flag Register

The processor maintains three status flags:

### Carry Flag

Set during arithmetic operations that generate carry.

### Zero Flag

Set when ALU output equals zero.

### Threshold Flag

Set according to THRESHOLD operation result.

## Future Enhancements

* CALL and RET Instructions
* Stack Pointer
* Interrupt Support
* Additional Branch Instructions
* UART Interface
* Pipelined Datapath
* RISC-V Inspired Extensions

## Conclusion

This project demonstrates the complete implementation of an educational 8-bit Harvard Architecture CPU using Verilog HDL. The design extends conventional SAP-1 style processors by introducing separate instruction and data memories, advanced ALU operations such as POPCNT and THRESHOLD, flag handling, input/output peripherals, and a scalable multi-cycle control architecture.

## Resources

* [Ben Eater's video series](https://eater.net/8bit/)
