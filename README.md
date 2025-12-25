Verilog — Top CRC Design
// MAIN MODULE CRC DESIGN
module top_crc_design (
input  wire clk,
input  wire reset_rtl_0,
input  wire [7:0] data_in,
input  wire clear,
output wire [15:0] crc_out
);

wire clk_out1_0;
wire clk_out2_0;
wire locked_0;

design_1_wrapper clk_wiz_inst (
.clk_100MHz(clk),
.reset_rtl_0(reset_rtl_0),
.clk_out1_0(clk_out1_0),
.clk_out2_0(clk_out2_0),
.locked_0(locked_0)
);

// CRC Instance
crc_parallel crc_inst (
.clk1(clk_out1_0),
.clk2(clk_out2_0),
.data_in(data_in),
.clear(clear),
.crc_out(crc_out)
);

endmodule

Verilog — CRC Parallel
module crc_parallel(clk1,clk2,data_in,clear,crc_out);
input clk1,clk2;
input [7:0] data_in;
input clear;
output wire [15:0] crc_out;

parameter POLY =  16'h8005;
reg [15:0] crc_table [0:255];

integer i,j;
reg [15:0] c;

initial begin
  for (i = 0; i < 256; i = i + 1) begin
    c = i;
    for (j = 0; j < 8; j = j + 1)
      c = (c & 1) ? (POLY ^ (c >> 1)) : (c >> 1);
    crc_table[i] = c;
  end
end

reg [15:0] stage1_crc;
reg [15:0] stage2_crc;
reg [15:0] stage3_crc;
reg [15:0] stage4_crc;
reg [15:0] crc_temp;

always @(posedge clk1) begin
  if (clear)
    stage1_crc <= 16'hFFFF ^ data_in;
  else
    stage1_crc <= crc_temp ^ data_in;
end

always @(posedge clk2) begin
  if (clear)
    stage2_crc <= 16'hFFFF;
  else
    stage2_crc <= crc_table[stage1_crc[7:0]];
end

always @(posedge clk1) begin
  if (clear)
    stage3_crc <= 16'hFFFF;
  else
    stage3_crc <= (crc_temp >> 8);
end

always @(posedge clk2) begin
  if (clear)
    stage4_crc <= 16'hFFFF;
  else
    stage4_crc <= stage2_crc ^ stage3_crc;
end

always @(posedge clk1) begin
  if (clear)
    crc_temp <= 16'hFFFF;
  else
    crc_temp <= stage4_crc;
end

assign crc_out = ~crc_temp;

endmodule

Verilog — Clock Wrapper
module design_1_wrapper
(
clk_100MHz,
clk_out1_0,
clk_out2_0,
locked_0,
reset_rtl_0
);

input clk_100MHz;
output clk_out1_0;
output clk_out2_0;
output locked_0;
input reset_rtl_0;

wire clk_100MHz;
wire clk_out1_0;
wire clk_out2_0;
wire locked_0;
wire reset_rtl_0;

design_1 design_1_i
(
.clk_100MHz(clk_100MHz),
.clk_out1_0(clk_out1_0),
.clk_out2_0(clk_out2_0),
.locked_0(locked_0),
.reset_rtl_0(reset_rtl_0)
);

endmodule

Verilog — Testbench

module tb_crc_parallel;

reg clk1, clk2;
reg [7:0] data_in;
reg clear;
wire [15:0] crc_out;

crc_parallel dut (
.clk1(clk1),
.clk2(clk2),
.data_in(data_in),
.clear(clear),
.crc_out(crc_out)
);

parameter PERIOD = 20;

initial begin
  clk1 = 0;
  clk2 = 0;
end

always #(PERIOD/2) clk1 = ~clk1;
always #(PERIOD/4) clk2 = ~clk2;

integer k;
initial begin
  data_in = 8'h00;
  clear = 1;
  #40 clear = 0;

  for (k = 0; k < 10; k = k + 1) begin
    @(posedge clk1);
    data_in = k;
  end

  repeat (8) @(posedge clk1);

  $display("Final CRC out = %h", crc_out);
  $stop;
end

initial begin
  $dumpfile("crc_parallel_tb.vcd");
  $dumpvars(0, tb_crc_parallel);

  $display("time   clk1 clk2 clear data_in    crc_out");
  $monitor("%4t   %b    %b    %b    0x%02x    0x%04x",
            $time, clk1, clk2, clear, data_in, crc_out);
end

endmodule

XDC — Constraints File
set_property -dict { PACKAGE_PIN W5   IOSTANDARD LVCMOS33 } [get_ports clk]
create_clock -add -name sys_clk_pin -period 10.00 -waveform {0 5} [get_ports clk]

# Switches
set_property -dict { PACKAGE_PIN V17 IOSTANDARD LVCMOS33 } [get_ports {data_in[0]}]
set_property -dict { PACKAGE_PIN V16 IOSTANDARD LVCMOS33 } [get_ports {data_in[1]}]
set_property -dict { PACKAGE_PIN W16 IOSTANDARD LVCMOS33 } [get_ports {data_in[2]}]
set_property -dict { PACKAGE_PIN W17 IOSTANDARD LVCMOS33 } [get_ports {data_in[3]}]
set_property -dict { PACKAGE_PIN W15 IOSTANDARD LVCMOS33 } [get_ports {data_in[4]}]
set_property -dict { PACKAGE_PIN V15 IOSTANDARD LVCMOS33 } [get_ports {data_in[5]}]
set_property -dict { PACKAGE_PIN W14 IOSTANDARD LVCMOS33 } [get_ports {data_in[6]}]
set_property -dict { PACKAGE_PIN W13 IOSTANDARD LVCMOS33 } [get_ports {data_in[7]}]

# LEDs — crc_out
set_property -dict { PACKAGE_PIN U16 IOSTANDARD LVCMOS33 } [get_ports {crc_out[0]}]
set_property -dict { PACKAGE_PIN E19 IOSTANDARD LVCMOS33 } [get_ports {crc_out[1]}]
set_property -dict { PACKAGE_PIN U19 IOSTANDARD LVCMOS33 } [get_ports {crc_out[2]}]
set_property -dict { PACKAGE_PIN V19 IOSTANDARD LVCMOS33 } [get_ports {crc_out[3]}]
set_property -dict { PACKAGE_PIN W18 IOSTANDARD LVCMOS33 } [get_ports {crc_out[4]}]
set_property -dict { PACKAGE_PIN U15 IOSTANDARD LVCMOS33 } [get_ports {crc_out[5]}]
set_property -dict { PACKAGE_PIN U14 IOSTANDARD LVCMOS33 } [get_ports {crc_out[6]}]
set_property -dict { PACKAGE_PIN V14 IOSTANDARD LVCMOS33 } [get_ports {crc_out[7]}]

# Buttons
set_property -dict { PACKAGE_PIN U18 IOSTANDARD LVCMOS33 } [get_ports clear]
set_property -dict { PACKAGE_PIN T18 IOSTANDARD LVCMOS33 } [get_ports pb]


