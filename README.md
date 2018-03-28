# datapath

module LEGv8_datapath (controlWord, K, Q, clock, reset, status_sig, status_z);

	input [30:0]controlWord;
	input [63:0]K;
	input clock;
	input reset;
	output[31:0]Q;
	output [3:0]status_sig;
	output status_z;
	
	wire RegWrite, BSel, En_mem, En_Alu, mem_write, En_PC, IR_L, status_load;
	wire [1:0]PS;
	wire [4:0]DA, SA, SB, FS;
	wire[3:0] status;
	wire[63:0] A, B, D, muxout, Data, mem_output, alu_output;
	
	assign {DA, SA, SB, FS, RegWrite, BSel, C0, En_mem, En_Alu, mem_write, PS, En_PC, IR_L, status_load} = controlWord;
	assign status_z = status[0];

	
	
	RegisterFile32x64 Regmain(A, B, SA, SB, Data, DA, RegWrite, reset, clock);
	Mux2to1Nbit mainmux(B, K, BSel, muxout);
	ALU_LEGv8 ALUmain (A, muxout, FS, C0, alu_output, status);
	LEGv8_RAM data_mem(alu_output[12:0], clock, B, mem_write, mem_output);
	
	assign Data = En_mem ? mem_output:64'bz;
	assign Data = En_Alu ? alu_output:64'bz;
	
	//defparam data_mem.memorywords = 6000;
	
	wire[63:0] PC, PC4, Rom_out;
	assign Data = En_PC ? PC:64'bz;
	LEGv8_programCounter PgC (alu_output, PS, PC, PC4, clock, reset);
	rom_case RC (Rom_out, PC4);
	
	
	RegisterNbit IR (Q, Rom_out, IR_L, reset, clock);
	defparam IR.n = 32;
	
	
	RegisterNbit Status_Reg (status_sig, status, status_load, reset, clock);
	defparam Status_Reg.n = 5;
	

endmodule
