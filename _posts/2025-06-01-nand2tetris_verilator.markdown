---
layout: post
title:  "Hardware Emulation using Open Source tools: a case study using nand2Tetris"
date:   2125-01-04 06:00:00 +0000
categories: [nand2tetris, SystemVerilog]
---

Ever since I joined AMD, I developed a stronger knack for hardware and understanding better how computers work at the lowest level. Last year I started a wonderful course called [nand2Tetris: bulding a modern computer from first principles](https://www.nand2tetris.org/). I had a lot of fun going through Part I, and while the Hardware Description Language used in the class was clear and simple, I wanted to challenge my knowledge using the industry standard SystemVerilog and emulating the full computer using only Open Source Software (C++ and verilator). In this post I will walk through my learning process, from the basics of the computer to its full emulation.

## Hack Computer Architecture 

The nand2Tetris computer is called HACK and is based on a modified von Neumann Architercure. In a classical von Neumann architecture the memory is shared between the program and the data memory. The HACK computer splits the memory in two separate banks, one for read only part for the program (ROM), and a random access memory (RAM) for the data.
The way this computer operates is very simple at every clock cycle (from the fallig edge of a clock to the next), one instruction from the ROM (left side of the image) is transferred in the CPU togetehr with any memory data from the RAM. The CPU then decodes  
 
