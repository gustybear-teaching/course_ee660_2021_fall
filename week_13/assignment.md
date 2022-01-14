---
title: "Assignment 2: Single-Cycle LC4 Processor"
date: 2021-09-01
type: book

summary: "The second assignment for EE660: Computer Architecture I."

tags:
- teaching
- ee660
- assignment

# Position of this page in the menu. Remove this option to sort alphabetically.
weight: 2

---

***
## Executive Summary
In this lab, you'll complete a single-cycle [LC4](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/LC4.pdf) processor. We're giving you the data and instruction memories (including all the I/O devices) . You'll use the ALU from the previous lab, implement a register file, and design other needed components. The end result is a fully functional [LC4](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/LC4.pdf) processor complete with video output.

## Project Files
A skeleton implementation, including Verilog code for the memory and all of the devices, pin constraints, and various memory images on which your processor will be tested is included in the compressed tarball [labfiles.tar.gz](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/labfiles.tar.gz). To extract the tarball, type 'tar -xvzf labfiles.tar.gz' at the eniac command line.

## Register File Module
The register file module will be in the file [lc4_regfile.v](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/lc4_regfile.v):

```Verilog
`timescale 1ns / 1ps

module lc4_regfile(clk, gwe, rst, r1sel, r1data, r2sel, r2data, wsel, wdata, we);
   parameter n = 16;

   input clk, gwe, rst;
   input [2:0] r1sel, r2sel, wsel;

   input [n-1:0] wdata;
   input we;
   output [n-1:0] r1data, r2data;

   wire [n-1:0] r0out, r1out, r2out, r3out, r4out, r5out, r6out, r7out;

   /*** YOUR CODE HERE ***/

endmodule
```

In a given cycle, any two registers may be read and any register may be written. A register write occurs only when the we signal is high. If the same register is read and written in the same cycle, the old value will be read (not the new value being written).

### Register File Schematic
First draw a detailed schematic (by hand or computerized) of the hardware design, including signal names, sub-modules, etc. Only once you have the design on paper in schematic form, then translate it directly to Verilog. That is, your Verilog should correspond to the schematic exactly.

### Register File Implementation
You'll build the register file out of a N-bit parametrized register module provided for you, [register.v](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/register.v):

```Verilog
`timescale 1ns / 1ps

/* A parameterized-width positive-edge-trigged register, with synchronous reset.
   The value to take on after a reset is the 2nd parameter. */

module Nbit_reg(in, out, clk, we, gwe, rst);
   parameter n = 1;
   parameter r = 0;

   output [n-1:0] out;
   input [n-1:0]  in;
   input          clk;
   input          we;
   input          gwe;
   input          rst;

   reg [n-1:0] state;

   assign #(1) out = state;

   always @(posedge clk)
     begin
       if (gwe & rst)
         state = r;
       else if (gwe & we)
         state = in;
     end
endmodule
```

> The "gwe" signal is a global write-enable (one write enable signal that controls every state variable in the design). For now, just pass this along into the N-bit register. There is more information about this below.

### Register File Testing
The testbench for the register file is [test_lc4_regfile.tf](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/test_lc4_regfile.tf) and the input trace is [test_lc4_regfile.input](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/test_lc4_regfile.input).

## Single-Cycle LC4 Processor
Below is a digram of a LC4 single-cycle datapath using the register file, ALU, and branch unit.

![](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/images/lc4_datapath.png)

Click on the image for a larger version. It is also available as: [lc4_datapath.pdf](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/lc4_datapath.pdf) and [lc4_datapath.pptx](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/lc4_datapath.pptx)

### LC4 Processor Module
The processor module is in the file [lc4_single.v](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/lc4_single.v). Most of your code should go into this file, although you want to create and use auxiliary files for sub-modules like the register file, ALU, branch unit, and other modules.:

```Verilog
`timescale 1ns / 1ps


module lc4_processor(clk,
                     rst,
                     gwe,
                     imem_addr,
                     imem_out,
                     dmem_addr,
                     dmem_out,
                     dmem_we,
                     dmem_in,
                     test_stall,
                     test_pc,
                     test_insn,
                     test_regfile_we,
                     test_regfile_reg,
                     test_regfile_in,
                     test_nzp_we,
                     test_nzp_in,
                     test_dmem_we,
                     test_dmem_addr,
                     test_dmem_value,
                     switch_data,
                     seven_segment_data,
                     led_data
                     );

   input         clk;         // main clock
   input         rst;         // global reset
   input         gwe;         // global we for single-step clock

   output [15:0] imem_addr;   // Address to read from instruction memory
   input  [15:0] imem_out;    // Output of instruction memory
   output [15:0] dmem_addr;   // Address to read/write from/to data memory
   input  [15:0] dmem_out;    // Output of data memory
   output        dmem_we;     // Data memory write enable
   output [15:0] dmem_in;     // Value to write to data memory

   output [1:0]  test_stall;       // Testbench: is this is stall cycle? (don't compare the test values)
   output [15:0] test_pc;          // Testbench: program counter
   output [15:0] test_insn;        // Testbench: instruction bits
   output        test_regfile_we;  // Testbench: register file write enable
   output [2:0]  test_regfile_reg; // Testbench: which register to write in the register file
   output [15:0] test_regfile_in;  // Testbench: value to write into the register file
   output        test_nzp_we;      // Testbench: NZP condition codes write enable
   output [2:0]  test_nzp_in;      // Testbench: value to write to NZP bits
   output        test_dmem_we;     // Testbench: data memory write enable
   output [15:0] test_dmem_addr;   // Testbench: address to read/write memory
   output [15:0] test_dmem_value;  // Testbench: value read/writen from/to memory

   input [7:0]   switch_data;
   output [15:0] seven_segment_data;
   output [7:0]  led_data;


   // PC
   wire [15:0]   pc;
   wire [15:0]   next_pc;

   Nbit_reg #(16, 16'h8200) pc_reg (.in(next_pc), .out(pc), .clk(clk), .we(1'b1), .gwe(gwe), .rst(rst));

   /*** YOUR CODE HERE ***/

   // Always execute one instruction each cycle
   assign test_stall = 2'b0;

   // For in-simulator debugging, you can use code such as the code
   // below to display the value of signals at each clock cycle.

//`define DEBUG
`ifdef DEBUG
   always @(posedge gwe) begin
      $display("%d %h %b %h", $time, pc, insn, alu_out_pre_mux);
   end
`endif

   // For on-board debugging, the LEDs and segment-segment display can
   // be configured to display useful information.  The below code
   // assigns the four hex digits of the seven-segment display to either
   // the PC or instruction, based on how the switches are set.

   assign seven_segment_data = (switch_data[6:0] == 7'd0) ? pc :
                               (switch_data[6:0] == 7'd1) ? imem_out :
                               (switch_data[6:0] == 7'd2) ? dmem_addr :
                               (switch_data[6:0] == 7'd3) ? dmem_out :
                               (switch_data[6:0] == 7'd4) ? dmem_in :
                               /*else*/ 16'hDEAD;
   assign led_data = switch_data;

endmodule
```
You will notice that the `lc4_processor` module has instruction and data memory signals declared in its external interface. The top module is [lc4_system.v](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/lc4_system.v) which instantiates the processor, memory, and devices, and interconnect them to each other and to the pins. In Xilinx, you must set [lc4_system.v](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/lc4_system.v) as the top module or else you will only synthesize the datapath and will not have a full working system.

The lc4_processor module in the [lc4_single.v](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/lc4_single.v) file also has extra outputs that begin with "test" that capture the changes to the processor state after each instruction. Your processor will work even if you don't connect these outputs to anything. As described below, they are provided to help you debug your code both in simulation (so that you can compare the values you have for these to values in a trace file genrated by [PennSim.jar](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/PennSim.jar)).

> What's this "global write enable" or "gwe" all over the place? The block memories in the FPGA are design such that they are read or written on a clock edge. Thus, we don't really have a "single cycle" design. Instead, we use a "global write enable" that is set once every four cycles. This allows reading of an instruction from the instruction memory and then reading/writing from the data memory all in a "single cycle". For the most part this should be transparent to your design.

> We're not asking you to implement the privilege bits or any other aspects to the PSR except just the N/Z/P bits.

### LC4 Processor Module Implementation
You have the freedom to implement the datapath and control any which way you want. All in all, the processor module should be at most 150-200 lines of Verilog.

> One caveat about the dmem_addr bus. This bus should have the value x0000 for any instruction other than a load or a store. The reason is that if this bus accidentally has a value that matches one of the memory-mapped device addresses, you could be unintentionally reading that device.

## Debugging Your Processor
### Using Trace Simulation
A good way to debug your design is using behavioral simulation and comparison with a trace. The `code/` subdirectory in the `labfiles/` folder contains several `.hex` and `.trace` files. the `.hex` files are memory dumps of various programs. For example, [house.hex](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/house.hex) is a memory dump of a small non-interactive program that draws the missile command houses on the screen and then exits. `house.trace` is a trace of that program created by PennSim.

A trace file is a text file containing one line per instruction executed, each with the same fields:

- The PC of the instruction
- The instruction bits themselves
- regfile_we. A bit that indicates if the instruction writes a register
- regfile_reg. The register written by the instruction (if the instruction writes a register)
- regfile_in. The value written to the register file (if the instruction writes a register)
- nzp_we. A bit that indicates if the instruction writes the condition codes (NZP)
- nzp_in. The new NZP bits (if the instruction writes the condition codes)
- dmem_we. A bit that indicates if the instruction writes memory
- dmem_addr. A value of the address accessed by a load or store
- dmem_value. A value that is written to memory (for stores) or read from memory (loads)

The Verilog test fixture [test_lc4_processor.tf](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/test_lc4_processor.tf) reads the `.trace` file, simulates the processor executing the `.hex` program instruction-by-instruction. When executing, it then compares the `lc4_processor` module interface signals to the corresponding fields in the trace file, allowing you to debug your implementation instruction by instruction.

The `.hex` file used to initialize the memory is specified in `include/bram.v`. Using a different `.hex` file requires editing the value of the `MEMORY_IMAGE_FILE` macro at the top of the memory module `bram.v`. This memory module is used both for simulation and for synthesis, so if you change it in one place it will affect the other.

The `labfiles/` contain several tests, which we recommend you use to test your design in the following order:

- test_alu.trace
- test_br.trace
- test_mem.trace
- test_all.trace
- house.trace

The newest versions of [PennSim.jar](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/PennSim.jar) also allows loading of .hex files directly using the new "loadhex" command. Just type in "loadhex house.hex" in the command line, for example. This will let you single-step through the execution in PennSim during your debugging.

### Creating Your Own Tests
The [house.hex](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/house.hex) file was created on PennSim by first loading the object files (`ld mcux` and `ld house`) and then using the command: `dump -readmemh x0000 xFFFF house.hex`. The `house.trace` file was created on PennSim using the commands `trace on house.trace`, then break set `OS_START`, `continue` and then `trace off`. You can write your own tests and create your own memory images and trace files in a similar way. You'll need the most recent version of [PennSim.jar](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/PennSim.jar).

### Debugging using Board Single-Stepping
Depending on how you count the processor executes at either 100MHz or 25MHz, but either speed is too fast to debug. You can use the expansion board to put the processor into "single-stepping" mode. Switch 8 controls clock mode: up is "auto", down is "single-step". To step the clock one cycle forward, use the down button on the main board.

When single-stepping through the program, the other expansion board switches determine what value is displayed on the expansion board 7-segment display. What is display is determined by the code at the end of your processor module:

```Verilog
assign seven_segment_data = (switch_data[6:0] == 7'd0) ? pc :
                            (switch_data[6:0] == 7'd1) ? imem_out :
                            (switch_data[6:0] == 7'd2) ? dmem_addr :
                            (switch_data[6:0] == 7'd3) ? dmem_out :
                            (switch_data[6:0] == 7'd4) ? dmem_in :
                            /*else*/ 16'hDEAD;
```

For instance, when the switches are set to 0 (i.e., all switches down) the 7-segment display shows the value currently on the pc bus, i.e., the program counter. You can also expand this code to get additional debugging info.

## Verilog Restrictions
This synthesizable part of this lab should be implemented using the structural and behavioral Verilog subset. The only state element you are allowed to use for synthesis is *Nbit_reg* in [lc4_regfile.v](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/lc4_regfile.v). You are allowed to use behavioral Verilog only in test fixtures. 

## Demos
You'll demonstrate that your design works using both simulation and the hardware boards:

- Simulation: You'll demonstrate that your design works in simulation for all the test cases we provided.
- Full Processor Hardware: You'll demonstrate that your design works correctly on your board using the [invaders.hex](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/invaders.hex) and [tetris.hex](https://github.com/gustybear-teaching/course_ee660_2021_fall/raw/main/week_13/files/tetris.hex) files.

## What to Turn In
Turn in a report via GitHub:

- Verilog Code: Your writeup should include Verilog code for the main modules and any sub-modules you created. But not the Verilog code we gave, just the code you wrote. Your Verilog code should be well-formatted, easy to understand, and include comments where appropriate (for example, use comments to describe the inputs and outputs to your Verilog modules). Some part of the project grade will be dependent on the style and readability of your Verilog, including formatting, comments, good signal names, and proper use of hierarchy.
- Questions: Answer the following questions:
  - Once you had the design working in simulation, did you encounter any problems getting it to run on the FPGA boards? If so, what problems did you encounter?
  - What other problems, if any, did you encounter while doing this lab?
  - How many hours did it take you to complete this assignment?
  - On a scale of 1 (least) to 5 (most), how difficult was this assignment?
  - What was the group division of labor on this assignment, in both hours and functional and debugging tasks?
