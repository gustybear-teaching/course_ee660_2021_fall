---
title: "Report 2: Single-Cycle LC4 Processor"
date: 2021-12-19
type: book
commentable: true

summary: "The second assignment for EE660, fall, 2021."

tags:
- teaching
- ee660
- assignment
---

***
## Executive Summary
In this lab, our team implemented an LC4 Register File, and integrated it with other code to complete a single-cycle LC4 processor. We integrated an ALU verilog code that was made in another assignment. 

# Register File Module 
The Register File was implemented such that within a given cycle, any two registers may be read and any register may be written. However, if the register write occurs at the same time as a read, the old value needs to be read, and not the one being written. This was accomplished by separating the initializing of the reads and the writes. Register reads occurred with an "always @(*)" initialization (the same as the testbench given with the assignement), whereas Writes were initialized on every clock cycle. This ensured, while testing, that all read data was not updated too soon. 

## Register File Schematic
The image below shows how the Register File Module was implemented. 
![Register File (1)](https://user-images.githubusercontent.com/16892369/146723154-d1d40349-aa42-4157-b79a-5bc9e3e62edb.jpg)

## Register File Code
Written below is the verilog code for the Register File:

```Verilog
`timescale 1ns / 1ps

module lc4_regfile(clk, gwe, rst, r1sel, r1data, r2sel, r2data, wsel, wdata, we);
   parameter n = 16;
   
   input clk, gwe, rst;
   input [2:0] r1sel, r2sel, wsel;
   
   input [n-1:0] wdata;
   input we;
   output reg [n-1:0] r1data, r2data;
   
   wire [n-1:0] r0out, r1out, r2out, r3out, r4out, r5out, r6out, r7out;

   /*** YOUR CODE HERE ***/
   reg [n-1:0] r0, r1, r2, r3, r4, r5, r6, r7;  // registers in Register file
   reg flag = 1'b0;     // used to initialize register values (to 16'b0) once Write Enable is set high
   
   wire [2:0] state;        // holds value of register to be written to (in the case of a write)
   reg [n-1:0] wdata_reg;   // register used to contain Write data, and properly write data to registers at the correct time
   reg we_reg;              // Used to hold write enable value 
   
   Nbit_reg Nreg (wsel, state, clk, we, gwe, rst);
   
   // assign output register wires to the registers being written to
   assign r0out = r0;
   assign r1out = r1;
   assign r2out = r2;
   assign r3out = r3;
   assign r4out = r4;
   assign r5out = r5;
   assign r6out = r6;
   assign r7out = r7;

   always @(*) begin 
    // Write data and write enable saved in register for timing purposes
    wdata_reg = wdata;
    we_reg = we;
    
     // set read data 1
     case (r1sel)
       3'b000 : r1data = r0out;
       3'b001 : r1data = r1out;
       3'b010 : r1data = r2out;
       3'b011 : r1data = r3out;
       3'b100 : r1data = r4out;
       3'b101 : r1data = r5out;
       3'b110 : r1data = r6out;
       3'b111 : r1data = r7out;
       default: r1data = 16'h0;
     endcase
     // set read data 2
     case (r2sel)
       3'b000 : r2data = r0out;
       3'b001 : r2data = r1out;
       3'b010 : r2data = r2out;
       3'b011 : r2data = r3out;
       3'b100 : r2data = r4out;
       3'b101 : r2data = r5out;
       3'b110 : r2data = r6out;
       3'b111 : r2data = r7out;
       default: r2data = 16'h0;
     endcase
     
     if (we & !flag) begin
       r0 = 16'h0;
       r1 = 16'h0;
       r2 = 16'h0;
       r3 = 16'h0; 
       r4 = 16'h0; 
       r5 = 16'h0; 
       r6 = 16'h0; 
       r7 = 16'h0; 
       flag = 1'b1;
     end
   end

  always @(posedge clk) begin
    #2 // Set to delay write till after Read occurs
    
    if (we_reg & gwe) begin // If global write is and Write is enabled, Write the wdata to the register responding to state
       case (state) // State (corresponding to register) determined by Nbit_reg program (and write select value, wsel)
         3'b000 : r0 = wdata_reg;
         3'b001 : r1 = wdata_reg;
         3'b010 : r2 = wdata_reg;
         3'b011 : r3 = wdata_reg;
         3'b100 : r4 = wdata_reg;
         3'b101 : r5 = wdata_reg;
         3'b110 : r6 = wdata_reg;
         3'b111 : r7 = wdata_reg;
       endcase 
     end
  end
endmodule
```
# Register File Demonstration 

Below are performance results of the code written above. This implementation recieved 0 errors from the testbench and input file given at the beginning of the assignment.

### Image 1 : Register FIle Waveform, 0 ns to 216 ns
![image](https://user-images.githubusercontent.com/16892369/146721580-98d2af78-9f76-477c-b57f-b5eb7cab84d2.png)

### Image 2 : Register File Waveform, 10,000 ns to 10,217 ns
![image](https://user-images.githubusercontent.com/16892369/146721068-9232189e-1f2c-49d3-b08f-de3764a9c9f4.png)

### TCL Console Output
```TCL 
source testbench_v.tcl
# set curr_wave [current_wave_config]
# if { [string length $curr_wave] == 0 } {
#   if { [llength [get_objects]] > 0} {
#     add_wave /
#     set_property needs_save false [current_wave_config]
#   } else {
#      send_msg_id Add_Wave-1 WARNING "No top level signals found. Simulator will start without a wave window. If you want to open a wave window go to 'File->New Waveform Configuration' or type 'create_wave_config' in the TCL console."
#   }
# }
# run 1000ns
INFO: [USF-XSim-96] XSim completed. Design snapshot 'testbench_v_behav' loaded.
INFO: [USF-XSim-97] XSim simulation ran for 1000ns
launch_simulation: Time (s): cpu = 00:00:03 ; elapsed = 00:00:08 . Memory (MB): peak = 1246.070 ; gain = 0.000
run all
Simulation finished:        1011 test cases           0 errors [C:/Users/niwc/Documents/School - Yamauchi/ee660/project_2/project_2.srcs/sources_1/imports/Downloads/test_lc4_regfile.input]
$finish called at time : 10217 ns : File "C:/Users/niwc/Documents/School - Yamauchi/ee660/project_2/project_2.srcs/sources_1/imports/Downloads/test_lc4_regfile.tf" Line 118
```
# LC4 Processor Module 
```
`timescale 1ns / 1ps

module lc4_processor(clk,
                     rst,
                     gwe,
                     imem_addr,
                     A_imem_out,
                     B_imem_out,
                     dmem_addr,
                     dmem_out,
                     dmem_we,
                     dmem_in,
                     test_A_stall,
                     test_B_stall,
                     test_A_pc,
                     test_B_pc,
                     test_A_insn,
                     test_B_insn,
                     test_A_regfile_we,
                     test_B_regfile_we,
                     test_A_regfile_reg,
                     test_B_regfile_reg,
                     test_A_regfile_in,
                     test_B_regfile_in,
                     test_A_nzp_we,
                     test_B_nzp_we,
                     test_A_nzp_in,
                     test_B_nzp_in,
                     test_A_dmem_we, 
                     test_B_dmem_we,
                     test_A_dmem_addr, 
                     test_B_dmem_addr,
                     test_A_dmem_value, 
                     test_B_dmem_value,
                     switch_data,
                     seven_segment_data,
                     led_data
                     );
   
   input         clk;         // main clock
   input         rst;         // global reset
   input         gwe;         // global we for single-step clock
   
   output [15:0] imem_addr;   // Address to read from instruction memory
   input  [15:0] A_imem_out, B_imem_out;    // Output of instruction memory
   output [15:0] dmem_addr;   // Address to read/write from/to data memory
   input  [15:0] dmem_out;    // Output of data memory
   output        dmem_we;     // Data memory write enable
   output [15:0] dmem_in;     // Value to write to data memory
   
   output [1:0]  test_A_stall, test_B_stall;       // Testbench: is this is stall cycle? (don't compare the test values). In superscalar, only 2nd (B) issue datapath will stall
   output [15:0] test_A_pc, test_B_pc;          // Testbench: program counter
   output [15:0] test_A_insn, test_B_insn;        // Testbench: instruction bits
   output        test_A_regfile_we, test_B_regfile_we;  // Testbench: register file write enable
   output [2:0]  test_A_regfile_reg, test_B_regfile_reg; // Testbench: which register to write in the register file 
   output [15:0] test_A_regfile_in, test_B_regfile_in;  // Testbench: value to write into the register file
   output        test_A_nzp_we, test_B_nzp_we;      // Testbench: NZP condition codes write enable
   output [2:0]  test_A_nzp_in, test_B_nzp_in;      // Testbench: value to write to NZP bits
   output        test_A_dmem_we, test_B_dmem_we;     // Testbench: data memory write enable
   output [15:0] test_A_dmem_addr, test_B_dmem_addr;   // Testbench: address to read/write memory
   output [15:0] test_A_dmem_value, test_B_dmem_value;  // Testbench: value read/writen from/to memory
   
   input [7:0]   switch_data;
   output [15:0] seven_segment_data;
   output [7:0]  led_data;
 
 
   // PC
   wire [15:0]   pc;
   wire [15:0]   next_pc;

   Nbit_reg #(16, 16'h8200) pc_reg (.in(next_pc), .out(pc), .clk(clk), .we(1'b1), .gwe(gwe), .rst(rst));

   /*** YOUR CODE HERE ***/
   wire [1:0] r1mux;
   wire branch_ctrl;
   wire [15:0] wire1, wire2;
   wire [3:0] r1sel_w, r2sel_w, wsel_w;
   reg [3:0] r1sel, r2sel, wsel;
   wire [15:0] alu_out;
   wire [15:0] mux9_reg01_in;

   assign r1sel_w = r1sel;
   assign r2sel_w = r2sel;
   assign wsel_w = wsel;

   // RegFile instantiation 
   mux_Nbit_8to1 mux1(.out(r1sel), .a(imem_addr[8:6]), .b(imem_addr[11:9]), .c(imem_addr[8:6]), .d(imem_addr[8:6]), .e(3'b111), .f(imem_addr[8:6]), .g(imem_addr[8:6]), .h(3'b111), .sel(imem_addr[15:13])); // determine r1sel (need to turn Select from 3'b to 4'b
   mux_Nbit_2to1 mux2(.out(r2sel), .a(imem_addr[11:9]), .b(imem_addr[2:0]), .sel(branch_ctrl)); // determine r2sel 
   mux_Nbit_2to1 mux3(.out(wdata_sel), .a(pc+1), .b(alu_out), .sel(ctrl1)); // determines the next PC
   mux_Nbit_2to1 mux4(.out(wire1), .a(pc+1), .b(alu_out), .sel(branch_ctrl)); // determines if branch taken or not
   mux_Nbit_2to1 mux5(.out(wd), .a(imem_addr[11:9]), .b(3'b111), .sel); //wsel on register bank
   mux_Nbit_2to1 mux6(.out(mux9_reg01_in), .a(wdata_sel), .b(dmem_out), .sel(branch_ctrl)); // mux after Memory and before regfile
   branch_logic ctrl1(.insn(imem_addr[15:0]), .branch_ctrl(branch_ctrl)); //determine wdata

   lc4_regfile reg_01 (.clk(clk), .gwe(gwe), .rst(rst), .r1sel(r1sel), .r1data(A_imem_out), .r2sel(r2sel), .r2data(B_imem_out), .wsel(wsel), .wdata(mux9_reg01_in), .we(dmem_we));

   // Always execute one instruction each cycle
   assign test_A_stall = 2'b0; 
   assign test_B_stall = 2'b1; 

   // Add ALU module
   lc4_alu alu(imem_addr, pc, A_imem_out, B_imem_out, alu_out);

   
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
                               (switch_data[6:0] == 7'd1) ? A_imem_out :
                               (switch_data[6:0] == 7'd2) ? dmem_addr :
                               (switch_data[6:0] == 7'd3) ? dmem_out :
                               (switch_data[6:0] == 7'd4) ? dmem_in :
                               /*else*/ 16'hDEAD;
   assign led_data = switch_data;
   
endmodule
```

# LC4 Processor Module Demonstration
We developed the code, but unforunately, we ran out of time so we did not test our code. 

# Questions
1. Once you had the design working in simulation, did you encounter any problems getting it to run on the FPGA boards? If so, what problems did you encounter?

   We did not get to work on the FPGA boards at all because we were unable to simulate and debug our code.

2. What other problems, if any, did you encounter while doing this lab?

   Our largest problem was figuring out what some of the named wires/registers were for since they were not clear in the lab code and diagram.

3. How many hours did it take you to complete this assignment?
  
    Breakdown given below.
  
4. On a scale of 1 (least) to 5 (most), how difficult was this assignment?
   5. There's a lot of moving parts along with the limited knowledge in the language that we need to complete it.

5. What was the group division of labor on this assignment, in both hours and functional and debugging tasks?
    
    Christian: 8 hours, debug layout of processor
  
    Devin: 13 hours on the lc4_processor, specifically on finding out how to connect the muxes and blocks of the processor together.
  
    Keola: I did 10 hours on trying to create the muxes and connections between the blocks. Mainly understanding what the blocks and implementing those in the modules
  
    Tyler: 3 hours to create Register File Schematic (time was mostly spent on research and understanding), 12 hours to write and debug Register File module. Debugging was mostly spent on fixing timing issues. For example, the testbench always updated the register value too early, and if too much of a delay was placed, the Write would occur on a new clock cycle, which of course had different values for wdata and Write Enable. The solution was to perform the Write at the very start of a new cycle.
