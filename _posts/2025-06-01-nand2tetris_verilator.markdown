---
layout: post
title:  "Hardware Emulation using Open Source tools: a case study using nand2Tetris"
date:   2025-07-01 06:00:00 +0000
categories: [nand2tetris, SystemVerilog]
---

Ever since I joined AMD, I developed a stronger knack for hardware and understanding better how computers work at the lowest level. Last year I started a wonderful course called [nand2Tetris: bulding a modern computer from first principles](https://www.nand2tetris.org/). I had a lot of fun going through Part I, and while the Hardware Description Language used in the class was clear and simple, I wanted to challenge my knowledge using the industry standard SystemVerilog and emulating the full computer using only Open Source Software (C++ and verilator). In this post I will walk through my learning process, from the basics of the computer to its full emulation.

## Hack Computer Architecture 

The nand2Tetris computer is called HACK and is based on a modified von Neumann Architercure. In a classical von Neumann architecture the memory is shared between the program and the data memory. The HACK computer splits the memory in two separate banks, one for read only part for the program (ROM), and a random access memory (RAM) for the data.
TThe architecture consists of three key components:

1. CPU: Features a minimalistic design with an Arithmetic Logic Unit (ALU) and two main registers: A (address or value) and D (data). The CPU executes one instruction per clock cycle, which drives the pace of computation. Each cycle processes an instruction—fetching it from memory, decoding it, executing it, and writing back results—all within a single tick of the clock.

2. Memory: Includes 32K words of RAM for data, along with memory-mapped I/O for a screen and keyboard. Programs are stored in ROM.

3. I/O: The screen and keyboard are accessed via specific memory addresses, allowing interaction without dedicated instructions.

The instruction set is deliberately minimal: just A-instructions for setting memory addresses or constants, and C-instructions for performing computations, memory access, and jumps. This simplicity makes Hack an ideal model for learning how hardware executes software at the machine level.

In the Hack computer architecture, the CPU executes one instruction per clock cycle. The system is synchronous, meaning that all state changes—such as writing to registers or memory—occur on the positive edge of the clock signal (when the clock transitions from low to high).

Importantly, the clock cycle begins at the negative edge of the clock. This design allows the combinational logic within the CPU to stabilize before the next state is latched at the positive edge. Here's a breakdown of what happens during each cycle:

1. Start of Cycle (Negative Edge)
    * No state is updated yet; this phase begins the cycle.
    * The Program Counter (PC) outputs an address to the instruction memory (ROM).
    * The ROM returns the instruction at that address.
    * The instruction is decoded and fed through combinational logic.
    * If it is an A-instruction, the 15-bit constant is prepared for loading into the A register.
    * If it is a C-instruction, the ALU computes the result based on the current values of the A or RAM[A] and D registers.
    * The destination is evaluated and enabling signals are sent to the respective AMD destinations.
    * The jump condition is also evaluated combinationally to determine the next PC value.

2. End of Cycle (Positive Edge)

   * At the rising edge of the clock, all enabled state elements are updated simultaneously:
   * The A register is loaded either with the constant from an A-instruction or with the ALU output if specified in a   C-instruction.
   * The D register is updated with the ALU result if the destination bits include D.
   * The RAM is written to if the instruction specifies memory output (via M) and uses the address in the A register.
   * The Program Counter is either incremented (by default) or updated with the value in the A register if a jump condition is met.

It helps to look at these cycles with an example. Let’s consider this Hack assembly code:

```asm
(END)
@END
0;JMP
```

These instructions create an infinite loop, and are used at the end of every Hack program. Assuming that the PC was at value 100, when hitting the @END. 

| Cycle | Phase   | PC  | A   | Instruction | Action Taken at negedge                        |
| ----- | ------- | --- | --- | ----------- | ---------------------------------------------- |
| 1     | negedge | 42  | ??  | `@END`      | Decode A-instr → Prepare to set A = 42, PC + 1 |
| 2     | negedge | 43  | 42  | `0;JMP`     | comp = 0, jump = JMP → Next PC = A = 42        |
| 3     | negedge | 42  | 42  | `@END`      | Decode A-instr → Prepare to set A = 42, PC + 1 |
| 4     | negedge | 43  | 42  | `0;JMP`     | comp = 0, jump = JMP → Next PC = A = 42        |