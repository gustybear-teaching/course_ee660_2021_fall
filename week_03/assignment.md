---
title: "Assignment 1: ALU"
date: 2021-09-01
type: book
commentable: true

summary: "The first assignment for EE660: Computer Architecture I."

tags:
- teaching
- ee660
- assignment
---

***
## Executive Summary
In this lab, you'll create a key component of the processor: the ALU for [LC4](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_03/files/LC4.pdf). We're providing the Verilog module definitions and testing harness to help you test your designs in Vivado. The testbench code uses Verilog file I/O and other advanced features of Verilog, which is why we've provided it to you. If you are not familar with Vivado for simulation and debugging Verilog, you can refer to the [Vivado tutorial for EE260](https://youtu.be/Io-uqv-oNEM).



## Part A: DIV/MOD Module
LC4 contains the DIV (divide) and MOD (modulo) instructions. To complete the ALU that you will use in your single-cycle design, the first step is to create the logic fo a single-cycle DIV/MOD calculation. This is the same calculation as the multi-cycle DIV/MOD given below, but done in a single cycle using combinational logic only.

### DIV/MOD Specification
The DIV/MOD module will be in the file [lc4_divider.v](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_03/files/lc4_divider.v).

```Verilog
`timescale 1ns / 1ps

module lc4_divider(dividend_in, divisor_in, remainder_out, quotient_out);

   input [15:0] dividend_in, divisor_in;
   output [15:0] remainder_out, quotient_out;

   /*** YOUR CODE HERE ***/

endmodule
```

The module takes as input two 16-bit data values (dividend and divisor) and outputs two 16-bit values (remainder and quotient). It calculates using the following algorithm:

```Verilog
int divide(int dividend, int divisor)
{
  int quotient = 0;
  int remainder = 0;

  if (divisor == 0) {
    return 0;
  }

  for (int i = 0; i < 16; i++) {
    remainder = (remainder << 1) | ((dividend >> 15) & 0x1);
    if (remainder < divisor) {
      quotient = (quotient << 1) | 0x0;
    } else {
      quotient = (quotient << 1) | 0x1;
      remainder = remainder - divisor;
    }
    dividend = dividend << 1;
  }

  return quotient;
}
```

### DIV/MOD Tips
Assuming you create a module that does one "iteration" of the division operation. You can then instantiate 16 copies of this module to form the full divider.

```Verilog
`timescale 1ns / 1ps

module lc4_divider_one_iter(dividend_in, divisor_in, remainder_in, quotient_in,
                            dividend_out, remainder_out, quotient_out);

   input [15:0] dividend_in, divisor_in, remainder_in, quotient_in;
   output [15:0] dividend_out, remainder_out, quotient_out;

   /*** YOUR CODE HERE ***/

endmodule
```

### DIV/MOD Testing
The testbench for the DIV/MOD unit is [test_lc4_divider.tf](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_03/files/test_lc4_divider.tf).

## Part B: ALU Module
The LC4 ALU module performs all of the arithmetic and logical operations for the various instructions. In this lab you'll build a self-contained ALU datapath with the corresponding control signals.

### ALU Specification
The ALU module will be in the file [lc4_alu.v](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_03/files/lc4_alu.v).

```Verilog
`timescale 1ns / 1ps

module lc4_alu(insn, pc, r1data, r2data, out);

   input [15:0] insn, pc, r1data, r2data;
   output [15:0] out;

   /*** YOUR CODE HERE ***/

endmodule
```

The module takes as inputs two 16-bit data values, a 16-bit instruction, the 16-bit PC value, and it generates a single 16-bit output. The two data inputs correspond to the two register values coming directly out of the register file. The output will be:

- For basic ALU operations (ADD, MUL, SUB, AND, NOT, OR, XOR, SLL, SRA, SLA, CONST, HICONST, etc.) the output of the ALU is the value the instruction will write back to the register file.
- For memory operations (LDR, STR) the output of the ALU is the generated effective memory address that will be later used as the address input into the data memory.
- For comparison instructions (CMP, CMPU, CMPI, CMPIU), the output will be zero (00000000000000), one (00000000000001), or negative one (1111111111111111), depending on the result of the comparison. This output value will then later be used to set the NZP bits.
- For branch instructions (JSRR, JMPR, BR, JMP, TRAP, RTI), the output should be the "taken" branch target (that is, the PC of the next instruction). For conditional branches (BR), this ALU module does not know the state of the condition codes (NZP), so the ALU calculates the target of the branch if the branch is indeed taken.
- For DIV and MOD, use the module described above that computes the quotient and remainder of the inputs. Do not use the built in '/' and '%' operators; they will not synthesize.
- For all other instructions (NOP), the output should be all zeros.

If any of the cases are ambiguous, please let us know and we'll clarify.

### ALU Schematic
First draw a detailed schematic (by hand or computerized) of the hardware design, including signal names, sub-modules, etc. Only once you have the design on paper in schematic form, then translate it directly to Verilog. That is, your Verilog should correspond to the schematic exactly.

### ALU implementation
To implement the basic operations of the ALU, including + and *, use the built in Verilog operators. To implement the shifter, I suggest implementing a barrel shifter as described in lecture. For signed and unsigned comparisons, consider extending the 16-bit values into 17-bit values (with either zero extend or sign extend, as appropriate), performing the subtraction, and then setting the output accordingly.

> you do not need to write your own adder or multiplier. Just use the builtin + and * operators. Division and modulo, however, does need its own implementation.

### ALU Testing
The testbench for the ALU is [test_lc4_alu.tf](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_03/files/test_lc4_alu.tf) and the input trace is [test_lc4_alu.input](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_03/files/test_lc4_alu.input).

## Verilog Restrictions
This synthesizable part of this lab should be implemented using the structural and behavioral Verilog subset as in presented in the class notes. In this lab, you should use only combinational logic (no state elements). If you're not sure if you're allowed to use a certain Verilog construct, just ask.

## What to Turn In
Turn in a Report in Markdown Format via the github repository of the class containing all of the following:

- Schematics: Include the hand-drawn (or computerized) schematics for each module you created. Wires should be labeled appropriately. The schematic should match up with your Verilog code, including the signal names and such.
- Verilog Code: Include your Verilog code for your two modules and any sub-modules you created. Your writeup should include your Verilog code. Not the Verilog code we gave, just the stuff you wrote. Your Verilog code should be well-formatted, easy to understand, and include comments where appropriate (for example, use comments to describe all the inputs and outputs to your Verilog modules). Some part of the project grade will be dependent on the style and readability of your Verilog, including formatting, comments, good signal names, and proper use of hierarchy.
- Questions: Finally, answer the following questions:
  - What other problems, if any, did you encounter while doing this lab?
  - How many hours did it take you to complete this assignment?
  - On a scale of 1 (least) to 5 (most), how difficult was this assignment?
  - What was the group division of labor on this assignment, in both hours and functional and debugging tasks?
