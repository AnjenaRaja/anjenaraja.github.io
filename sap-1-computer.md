---
layout: page
title: SAP-1 Computer
subtitle: Digital Electronics Club Project (Active - Work in progress)
---

# SAP (Simple-As-Possible) Computer

## Architecture

## Clock 
The clock on the Nexys A7 FPGA development board runs at 100 MHz, which is too fast for our purposes, as our primary focus is to visualize state transitions. To effectively debug the execution, we need to slow the clock down to approximately 1 Hz. Additionally, we want to be able to step through the states manually. 

We can use one of the 16 slide switches available on the development board to toggle between the automatic slow clock and manual clock modes. Additionally, we can assign one of the push buttons to manually advance the clock. However, since these are mechanical switches, sliding the switch or pressing the button may produce noisy signals, which the FPGA could misinterpret as multiple transitions. Therefore, we need to debounce the switch transitions to ensure reliable operation.

To achieve this, we need the following:

1. Slow down the system clock.
2. Advance the clock with a button press.
3. Switch between the automatic slow clock and manual clock control.
4. Halt the clock when our computer completes its program.
5. Reset the clock and the debounce logic.
6. Drive an LED to indicate the clock pulse.

### Verilog Code for Clock

```verilog
module clock (
    input wire clk_in,        // Input clock (100 MHz from Nexys A7-100)
    input wire reset,         // Reset signal
    input wire mode_switch,   // Slide switch for single-step (1) or slow clock (0)
    input wire step_button,   // Push button for single-step mode
    input wire halt,          // halt flag to end the program
    output reg clk_out        // Output clock (slowed or single-step)
);

    parameter DIVISOR = 50000000;  // Parameter to slow the clock (adjustable)
    wire debounced_step_button;    // Debounced button signal
    wire debounced_mode_switch;    // Debounced slide switch

    reg [31:0] counter = 0;        // 32-bit counter for clock division
    reg slow_clk = 0;              // Internal slow clock signal
    reg button_state_prev = 0;     // Previous state of the button

    // Instantiate the debounce module for the step button
    debounce #(.DEBOUNCE_TIME(100000)) debounce_step (
        .clk(clk_in),
        .reset(reset),
        .noisy_signal(step_button),
        .debounced_signal(debounced_step_button)
    );

    // Instantiate the debounce module for the mode switch
    debounce #(.DEBOUNCE_TIME(100000)) debounce_mode (
        .clk(clk_in),
        .reset(reset),
        .noisy_signal(mode_switch),
        .debounced_signal(debounced_mode_switch)
    );
    
    // Clock Divider: Generates the slow clock
    always @(posedge clk_in or posedge reset) begin
        if (reset) begin
            counter <= 0;
            slow_clk <= 0;
        end else begin
            if (counter >= (DIVISOR - 1)) begin
                counter <= 0;
                slow_clk <= ~slow_clk;  // Toggle the slow clock
            end else begin
                counter <= counter + 1;
            end
        end
    end

    // Single-Step Mode or Slow Clock Output Logic
    always @(posedge clk_in or posedge reset) begin
        if (reset) begin
            clk_out <= 0;
            button_state_prev <= 0;
        end else begin
            if (debounced_mode_switch) begin
                // Single-step mode: clock toggles only on rising edge of debounced button
                if (debounced_step_button && ~button_state_prev) begin
                    clk_out <= (halt) ? 0 : ~clk_out;
                end
            end else begin
                // Slow clock mode: use the divided slow clock
                clk_out <= (halt) ? 0 : slow_clk;
            end
            button_state_prev <= debounced_step_button; // Store the previous button state
        end
    end

endmodule
```

### Constraints Definition

```tcl
## Clock signal
set_property -dict { PACKAGE_PIN E3    IOSTANDARD LVCMOS33 } [get_ports { clk_in }]; #IO_L12P_T1_MRCC_35 Sch=clk100mhz
create_clock -add -name sys_clk_pin -period 10.00 -waveform {0 5} [get_ports {clk_in}];
##Switches
set_property -dict { PACKAGE_PIN V10   IOSTANDARD LVCMOS33 } [get_ports { mode_switch }]; #IO_L21P_T3_DQS_14 Sch=sw[15]
## LEDs
set_property -dict { PACKAGE_PIN V11   IOSTANDARD LVCMOS33 } [get_ports { clk_led }]; #IO_L21N_T3_DQS_A06_D22_14 Sch=led[15]
##Buttons
set_property -dict { PACKAGE_PIN M18   IOSTANDARD LVCMOS33 } [get_ports { step_button }]; #IO_L4N_T0_D05_14 Sch=btnu
set_property -dict { PACKAGE_PIN P17   IOSTANDARD LVCMOS33 } [get_ports { reset }]; #IO_L12P_T1_MRCC_14 Sch=btnl
```

### Verilog Code for Debouncing Switches

```verilog
module debounce (
    input wire clk,           // System clock (e.g., 100 MHz)
    input wire reset,         // Reset signal
    input wire noisy_signal,  // Noisy input signal (button press)
    output reg debounced_signal // Debounced output signal
);

    parameter DEBOUNCE_TIME = 100000; // Debounce time in clock cycles

    reg [31:0] debounce_counter = 0; // Counter to track signal stability
    reg stable_signal = 0;           // Stable signal before debouncing

    // Debounce Logic
    always @(posedge clk or posedge reset) begin
        if (reset) begin
            debounce_counter <= 0;
            debounced_signal <= 0;
            stable_signal <= 0;
        end else begin
            if (noisy_signal == stable_signal) begin
                debounce_counter <= 0; // No change in signal, reset counter
            end else begin
                // Signal has changed, start counting
                if (debounce_counter >= DEBOUNCE_TIME) begin
                    stable_signal <= noisy_signal; // Signal stable for long enough
                    debounced_signal <= noisy_signal;
                    debounce_counter <= 0;
                end else begin
                    debounce_counter <= debounce_counter + 1; // Increment counter
                end
            end
        end
    end

endmodule
```

## Program Counter

### Verilog Code for Program Counter

```verilog
module pc (
    input wire clk,
    input wire reset,
    input wire inc,
    output reg [3:0] pc_out
);
    always @(posedge clk or posedge reset) begin
        if (reset)
            pc_out <= 4'b0;
        else if (inc && pc_out < 4'b1111)
            pc_out <= pc_out + 1;
    end
endmodule
```

## Memory (RAM)

### Verilog Code for Memory

```verilog
module memory (
    input wire [3:0] address,
    output reg [7:0] data_out
);
    reg [7:0] ram [0:15]; // 16 bytes of memory

    initial begin
        // Example program
        ram[0]  = 8'b00011110; // LDA 14
        ram[1]  = 8'b00101111; // ADD 15
        ram[2]  = 8'b00111101; // SUB 13
        ram[3]  = 8'b01000000; // OUT
        ram[4]  = 8'b11110000; // HLT

        ram[5]  = 8'b00000000; // NOP
        ram[6]  = 8'b00000000; // NOP
        ram[7]  = 8'b00000000; // NOP
        ram[8]  = 8'b00000000; // NOP
        ram[9]  = 8'b00000000; // NOP
        ram[10]  = 8'b00000000; // NOP
        ram[11]  = 8'b00000000; // NOP
        ram[12]  = 8'b00000000; // NOP

        ram[13] = 8'b00000010; // Data at memory location value 2
        ram[14] = 8'b00000110; // Data at memory location value 6
        ram[15] = 8'b00000101; // Data at memory location value 5
    end

    always @(*) begin
        data_out = ram[address];
    end
endmodule
```

## Registers

## Accumulator - A Register

### Verilog code for A Register

```verilog
module a (
    input wire clk,
    input wire reset,
    input wire load,
    input wire [7:0] data_in,
    output reg [7:0] data_out
);
    always @(posedge clk or posedge reset) begin
        if (reset)
            data_out <= 8'b0;
        else if (load)
            data_out <= data_in;
    end
endmodule
```

## B Register

### Verilog code for B Register

```verilog
module b (
    input wire clk,
    input wire reset,
    input wire load,
    input wire [7:0] data_in,
    output reg [7:0] data_out
);
    always @(posedge clk or posedge reset) begin
        if (reset)
            data_out <= 8'b0;
        else if (load)
            data_out <= data_in;
    end
endmodule
```

## Memory Address Register (MAR)

### Verilog code for Memory Address Register (MAR)

```verilog
module mar(
    input wire clk,
    input wire reset,
    input wire load,
    input wire [3:0] address_in,
    output reg [3:0] address_out
);
    always @(posedge clk, posedge reset) begin
        if (reset) begin
            address_out <= 4'b0;
        end else if (load) begin
            address_out <= address_in;
        end
    end 

endmodule
```

## Instruction Register (IR)

### Verilog code for Instruction Register (IR)

```verilog
module ir (
    input wire clk,
    input wire reset,
    input wire load,
    input wire [7:0] instruction_in,
    output reg [3:0] opcode,
    output reg [3:0] operand
);

    initial begin
        opcode = 4'b0;
        operand = 4'b0;
    end
    
    always @(posedge clk, posedge reset) begin
        if (reset) begin
            opcode <= 4'b0;
            operand <= 4'b0;
        end else if (load) begin
            opcode <= instruction_in[7:4];
            operand <= instruction_in[3:0];
        end
    end
endmodule
```

## Output Register

### Verilog code for Output Register

```verilog
module out (
    input wire clk,
    input wire reset,
    input wire load,
    input wire [7:0] data_in,
    output reg [7:0] data_out
);
    always @(posedge clk or posedge reset) begin
        if (reset) begin
            data_out <= 1'b00000000;
        end else if (load) begin
            data_out <= data_in;
        end
    end
endmodule
```

## Arithmetic and Logic Unit (ALU)

### Verilog code for Arithmetic and Logic Unit (ALU)

```verilog
module alu (
    input wire [7:0] a,
    input wire [7:0] b,
    input wire sub,
    output reg [7:0] result
);
    always @(*) begin
        if (sub)
            result = a - b;
        else
            result = a + b;
    end
endmodule
```

## Bus

## Control Sequencer

## Micro Code

### Verilog code for Control Sequencer

```verilog
module controller (
    input wire clk,
    input wire reset,
    input wire [3:0] opcode,    // opcode is 4 bits wide, 16 possible instructions
    output [12:0] cs,           // we have 13 bits of control signals
    output [2:0] tstate
);

    // These are the supported instruction 
    // NOP is not part of the SAP-1
    // but added here to avoid confusion as 
    // we set the opcode to 4'b0000 when a reset is triggered 
    localparam OP_NOP = 4'b0000;        // no operation
    localparam OP_LDA = 4'b0001;        // LOAD A
    localparam OP_ADD = 4'b0010;        // ADD 
    localparam OP_SUB = 4'b0011;        // SUB
    localparam OP_OUT = 4'b0100;        // OUT
    localparam OP_HLT = 4'b1111;        // HALT

    // Control signals are parameters 
    localparam LOAD_OUT   = 12;
    localparam HALT       = 11;
    localparam INC_PC    = 10;
    localparam EN_PC     = 9;
    localparam LOAD_MAR  = 8;
    localparam EN_RAM    = 7;
    localparam LOAD_IR   = 6;
    localparam EN_IR     = 5;
    localparam LOAD_A    = 4;
    localparam EN_A      = 3;
    localparam LOAD_B    = 2;
    localparam SUB_ALU   = 1;
    localparam EN_ALU    = 0;

    // we have 6 T states that breaks down the opcode into microcodes
    // this is a machine cycle
    reg[2:0] t_state;
    
    always @(posedge clk, posedge reset) begin
        if (reset) begin
            t_state <= 0;
        end else begin
            if (t_state == 5) begin
                t_state <= 0;
            end else begin
                t_state <= t_state + 1;
            end
        end
     end
    
    // we need to assemble the control signals into cw and finally write it out
    reg[12:0] cw;
    
    // Define control signals for different instructions
    // These signals change based on the T state we are in
    always @(*) begin
        
        // we clear the control word first and 
        cw = 13'b0000000000000;
        // then build it based on the opcode and T state
    
        case (t_state)
            0: begin
                // this state is independent of the opcode
                // we need to load the the instruction pointed by PC into MAR
                cw[EN_PC] = 1;       // PC writes to the BUS
                cw[LOAD_MAR] = 1;    // MAR loads from the BUS
            end
            1: begin
               // this state is independent of the opcode
               cw[INC_PC] = 1;     // increment PC for the next instruction
            end
            2: begin
                // this state is independent of the opcode
                // RAM is hard wired to MAR
                // we need to load the RAM's content into the IR
                cw[EN_RAM] = 1;     // RAM writes to the BUS
                cw[LOAD_IR] = 1;    // IR loads from the BUS
            end
            // IR has already split the opcode part the instruction stored in RAM
            // We need to now handle the remaining T states based on the opcode
            3: begin
                case (opcode)
                    OP_NOP: begin
                        // we don't have to do anything
                        // just waste this clock cycle
                    end
                    OP_LDA, OP_ADD, OP_SUB: begin
                        // the operand part of the IR points to the memory location
                        // we need to know which memory location that is
                        cw[EN_IR] = 1;      // IR's writes to the BUS
                        cw[LOAD_MAR] = 1;   // MAR loads from the BUS
                    end
                    OP_HLT: begin
                        // this is the end of the program and so we can stop the clock
                        cw[HALT] = 1;
                    end
                    OP_OUT: begin
                        cw[EN_A] = 1;
                        cw[LOAD_OUT] = 1;
                    end
                endcase
            end
            4: begin
                case (opcode)
                    OP_NOP, OP_OUT: begin
                        // we don't have to do anything
                        // just waste this clock cycle
                    end
                    OP_LDA: begin
                        cw[EN_RAM] = 1;     // RAM's writes to the BUS
                        cw[LOAD_A] = 1;     // A Register loads from the BUS
                    end
                    OP_ADD, OP_SUB: begin
                        cw[EN_RAM] = 1;     // RAM's writes to the BUS
                        cw[LOAD_B] = 1;     // B Register loads from the BUS
                    end
                    OP_HLT: begin
                        // since the clock was stopped in the previous T state
                        // the code should not even reach here for this opcode
                        cw[HALT] = 1;
                    end
                endcase
            end
            5: begin
                case (opcode)
                    OP_NOP, OP_OUT: begin
                        // we don't have to do anything
                        // just waste this clock cycle
                    end
                    OP_ADD: begin
                        cw[EN_ALU] = 1;     // ALU's writes to the BUS
                        cw[LOAD_A] = 1;     // A Register loads from the BUS
                    end
                    OP_SUB: begin
                        cw[SUB_ALU] = 1;    // Let ALU know that we need it to subtract and not add
                        cw[EN_ALU] = 1;     // ALU's writes to the BUS
                        cw[LOAD_A] = 1;     // A Register loads from the BUS
                    end
                    OP_HLT: begin
                        // since the clock was stopped in the previous T state
                        // the code should not even reach here for this opcode
                        cw[HALT] = 1;
                    end
                endcase
            end
        endcase
    end
    
    assign cs = cw;
    assign tstate = t_state;
    
endmodule
```

### Constraints Definition

```tcl
## LEDs
set_property -dict { PACKAGE_PIN H17   IOSTANDARD LVCMOS33 } [get_ports { cs_led[0] }]; #IO_L18P_T2_A24_15 Sch=led[0]
set_property -dict { PACKAGE_PIN K15   IOSTANDARD LVCMOS33 } [get_ports { cs_led[1] }]; #IO_L24P_T3_RS1_15 Sch=led[1]
set_property -dict { PACKAGE_PIN J13   IOSTANDARD LVCMOS33 } [get_ports { cs_led[2] }]; #IO_L17N_T2_A25_15 Sch=led[2]
set_property -dict { PACKAGE_PIN N14   IOSTANDARD LVCMOS33 } [get_ports { cs_led[3] }]; #IO_L8P_T1_D11_14 Sch=led[3]
set_property -dict { PACKAGE_PIN R18   IOSTANDARD LVCMOS33 } [get_ports { cs_led[4] }]; #IO_L7P_T1_D09_14 Sch=led[4]
set_property -dict { PACKAGE_PIN V17   IOSTANDARD LVCMOS33 } [get_ports { cs_led[5] }]; #IO_L18N_T2_A11_D27_14 Sch=led[5]
set_property -dict { PACKAGE_PIN U17   IOSTANDARD LVCMOS33 } [get_ports { cs_led[6] }]; #IO_L17P_T2_A14_D30_14 Sch=led[6]
set_property -dict { PACKAGE_PIN U16   IOSTANDARD LVCMOS33 } [get_ports { cs_led[7] }]; #IO_L18P_T2_A12_D28_14 Sch=led[7]
set_property -dict { PACKAGE_PIN V16   IOSTANDARD LVCMOS33 } [get_ports { cs_led[8] }]; #IO_L16N_T2_A15_D31_14 Sch=led[8]
set_property -dict { PACKAGE_PIN T15   IOSTANDARD LVCMOS33 } [get_ports { cs_led[9] }]; #IO_L14N_T2_SRCC_14 Sch=led[9]
set_property -dict { PACKAGE_PIN U14   IOSTANDARD LVCMOS33 } [get_ports { cs_led[10] }]; #IO_L22P_T3_A05_D21_14 Sch=led[10]
set_property -dict { PACKAGE_PIN T16   IOSTANDARD LVCMOS33 } [get_ports { cs_led[11] }]; #IO_L15N_T2_DQS_DOUT_CSO_B_14 Sch=led[11]
set_property -dict { PACKAGE_PIN V15   IOSTANDARD LVCMOS33 } [get_ports { cs_led[12] }]; #IO_L16P_T2_CSI_B_14 Sch=led[12]
```

## Display
The development board has 8 seven-segment LEDs, which we would like to use to display the hexadecimal values of various signals such as the Program Counter, the Bus, the Memory Address Register (MAR), the RAM contents pointed to by the MAR, T States, and more.

We need to decode the hexadecimal values to drive the individual segments of the seven-segment LED displays. Additionally, a multiplexer is required to control each of the seven-segment displays. Since we are relying on persistence of vision, we will use the system clock to refresh the display.

### Verilog Code for 7-Segment Decoder

```verilog
module seven_seg_decoder (
    input [3:0] digit,   // 4-bit input digit (0-15 for hex)
    output reg [6:0] seg // 7-segment display output (A-G)
);

always @(*) begin
    case (digit)
        4'd0: seg = 7'b1000000;  // Display 0
        4'd1: seg = 7'b1111001;  // Display 1
        4'd2: seg = 7'b0100100;  // Display 2
        4'd3: seg = 7'b0110000;  // Display 3
        4'd4: seg = 7'b0011001;  // Display 4
        4'd5: seg = 7'b0010010;  // Display 5
        4'd6: seg = 7'b0000010;  // Display 6
        4'd7: seg = 7'b1111000;  // Display 7
        4'd8: seg = 7'b0000000;  // Display 8
        4'd9: seg = 7'b0010000;  // Display 9
        4'd10: seg = 7'b0001000; // Display A
        4'd11: seg = 7'b0000011; // Display B
        4'd12: seg = 7'b1000110; // Display C
        4'd13: seg = 7'b0100001; // Display D
        4'd14: seg = 7'b0000110; // Display E
        4'd15: seg = 7'b0001110; // Display F
        default: seg = 7'b1111111;  // Turn off all segments
    endcase
end

endmodule
```

### Verilog Code for 7-Segment Multiplexer

```verilog
module seven_seg_mux (
    input clk,                // Clock input
    input reset,              // Reset signal
    input [31:0] value,       // 32-bit value containing 8 digits
    output reg [7:0] an,      // Anodes for the 8 digits
    output reg [3:0] digit    // Current digit to display
);

reg [19:0] refresh_counter = 0;   // Refresh counter for multiplexing
wire [2:0] refresh_digit;         // Current digit being refreshed

// Split the 32-bit value into eight 4-bit digits
wire [3:0] digit0 = value[3:0];    // Right-most digit
wire [3:0] digit1 = value[7:4];    // Second digit
wire [3:0] digit2 = value[11:8];   // Third digit
wire [3:0] digit3 = value[15:12];  // Fourth digit
wire [3:0] digit4 = value[19:16];  // Fifth digit
wire [3:0] digit5 = value[23:20];  // Sixth digit
wire [3:0] digit6 = value[27:24];  // Seventh digit
wire [3:0] digit7 = value[31:28];  // Left-most digit

// Refresh rate for multiplexing the display (e.g., 1kHz)
always @(posedge clk or posedge reset) begin
    if (reset)
        refresh_counter <= 0;
    else
        refresh_counter <= refresh_counter + 1;
end

// The current digit to display (multiplexed)
assign refresh_digit = refresh_counter[19:17]; // Change every few clock cycles

// Multiplex the anodes and assign the current digit based on the refresh_digit
always @(*) begin
    if (reset) begin
        an = 8'b11111111;   // Disable all digits during reset
        digit = 4'b0000;    // Set all digits to zero
    end else begin
        case (refresh_digit)
            3'b000: begin
                an = 8'b11111110;   // Enable the right-most digit (digit 0)
                digit = digit0;
            end
            3'b001: begin
                an = 8'b11111101;   // Enable the second digit (digit 1)
                digit = digit1;
            end
            3'b010: begin
                an = 8'b11111011;   // Enable the third digit (digit 2)
                digit = digit2;
            end
            3'b011: begin
                an = 8'b11110111;   // Enable the fourth digit (digit 3)
                digit = digit3;
            end
            3'b100: begin
                an = 8'b11101111;   // Enable the fifth digit (digit 4)
                digit = digit4;
            end
            3'b101: begin
                an = 8'b11011111;   // Enable the sixth digit (digit 5)
                digit = digit5;
            end
            3'b110: begin
                an = 8'b10111111;   // Enable the seventh digit (digit 6)
                digit = digit6;
            end
            3'b111: begin
                an = 8'b01111111;   // Enable the left-most digit (digit 7)
                digit = digit7;
            end
            default: begin
                an = 8'b11111111;   // Disable all digits
                digit = 4'b0000;
            end
        endcase
    end
end

endmodule
```

### Constraints Definition

```tcl
##7 segment display
set_property -dict { PACKAGE_PIN T10   IOSTANDARD LVCMOS33 } [get_ports { seg[0] }]; #IO_L24N_T3_A00_D16_14 Sch=ca
set_property -dict { PACKAGE_PIN R10   IOSTANDARD LVCMOS33 } [get_ports { seg[1] }]; #IO_25_14 Sch=cb
set_property -dict { PACKAGE_PIN K16   IOSTANDARD LVCMOS33 } [get_ports { seg[2] }]; #IO_25_15 Sch=cc
set_property -dict { PACKAGE_PIN K13   IOSTANDARD LVCMOS33 } [get_ports { seg[3] }]; #IO_L17P_T2_A26_15 Sch=cd
set_property -dict { PACKAGE_PIN P15   IOSTANDARD LVCMOS33 } [get_ports { seg[4] }]; #IO_L13P_T2_MRCC_14 Sch=ce
set_property -dict { PACKAGE_PIN T11   IOSTANDARD LVCMOS33 } [get_ports { seg[5] }]; #IO_L19P_T3_A10_D26_14 Sch=cf
set_property -dict { PACKAGE_PIN L18   IOSTANDARD LVCMOS33 } [get_ports { seg[6] }]; #IO_L4P_T0_D04_14 Sch=cg
set_property -dict { PACKAGE_PIN H15   IOSTANDARD LVCMOS33 } [get_ports { dp }]; #IO_L19N_T3_A21_VREF_15 Sch=dp
set_property -dict { PACKAGE_PIN J17   IOSTANDARD LVCMOS33 } [get_ports { anode[0] }]; #IO_L23P_T3_FOE_B_15 Sch=an[0]
set_property -dict { PACKAGE_PIN J18   IOSTANDARD LVCMOS33 } [get_ports { anode[1] }]; #IO_L23N_T3_FWE_B_15 Sch=an[1]
set_property -dict { PACKAGE_PIN T9    IOSTANDARD LVCMOS33 } [get_ports { anode[2] }]; #IO_L24P_T3_A01_D17_14 Sch=an[2]
set_property -dict { PACKAGE_PIN J14   IOSTANDARD LVCMOS33 } [get_ports { anode[3] }]; #IO_L19P_T3_A22_15 Sch=an[3]
set_property -dict { PACKAGE_PIN P14   IOSTANDARD LVCMOS33 } [get_ports { anode[4] }]; #IO_L8N_T1_D12_14 Sch=an[4]
set_property -dict { PACKAGE_PIN T14   IOSTANDARD LVCMOS33 } [get_ports { anode[5] }]; #IO_L14P_T2_SRCC_14 Sch=an[5]
set_property -dict { PACKAGE_PIN K2    IOSTANDARD LVCMOS33 } [get_ports { anode[6] }]; #IO_L23P_T3_35 Sch=an[6]
set_property -dict { PACKAGE_PIN U13   IOSTANDARD LVCMOS33 } [get_ports { anode[7] }]; #IO_L23N_T3_A02_D18_14 Sch=an[7]
```

## SAP Computer

### Verilog Code for SAP Computer

```verilog
module top(
    input wire clk_in,          // 100 MHz input clock from Nexys A7-100T
    input wire reset,           // Reset signal
    input wire mode_switch,     // Slide switch for single-step (1) or slow clock (0)
    input wire step_button,     // Push button for single-step mode
    output wire clk_led,        // LED to display clock output
    output wire [6:0] seg,      // 7-segment display output
    output wire dp,             // decimal point output
    output wire [7:0] anode,    // Anode control for 7-segment displays
    output wire [12:0] cs_led   // LED to display the control signals
);

    wire clk;           // Output clock from the clock module
    wire halt;          // flag to halt to the CPU clock
    
    // Instantiate the clock module
    clock clock_inst (
        .clk_in(clk_in),            // Connect the input clock
        .reset(reset),              // Connect the reset signal
        .mode_switch(mode_switch),  // Mode switch to select single-step or slow clock
        .step_button(step_button),  // Push button for single-step mode
        .halt(halt),                // halt the CPU clock
        .clk_out(clk)               // Output clock from the clock module
    );
        
    // Connect the clock output to the LED for visualization
    assign clk_led = clk;
    
    reg [7:0] bus; // this is the common bus for both address and data
    
    always @(*) begin
        case (1'b1) // This will select the first true condition
            ir_en: bus = {4'b0, operand};
            alu_en: bus = alu_out;
            a_en: bus = a_out;
            mem_en: bus = mem_out;
            pc_en: bus = {4'b0, pc_out};
            default: bus = 8'b0;
        endcase
    end

    wire [3:0] pc_out;
    wire pc_inc;
    wire pc_en;
    
    // Instantiate Program Counter
    pc pc_inst (
        .clk(clk),
        .reset(reset),
        .inc(pc_inc),
        .pc_out(pc_out)
    );

    wire mar_load;
    wire [3:0] mar_out;
    
    // Instantiate Memory Address Register (MAR)
    mar mar_inst (
        .clk(clk),
        .reset(reset),
        .load(mar_load),
        .address_in(bus[3:0]),
        .address_out(mar_out)
    );
    
    wire mem_en;
    wire [7:0] mem_out;

    // Instantiate Memory (RAM)
    memory memory_instance (
        .address(mar_out),
        .data_out(mem_out)
    );
    
    wire a_load;
    wire a_en;
    wire [7:0] a_out;
    a a_inst(
        .clk(clk),
        .reset(reset),
        .load(a_load),
        .data_in(bus),
        .data_out(a_out)
    );

    wire b_load;
    wire [7:0] b_out;
    b b_inst(
        .clk(clk),
        .reset(reset),
        .load(b_load),
        .data_in(bus),
        .data_out(b_out)
    );    
    
    wire alu_sub;
    wire alu_en;
    wire [7:0] alu_out;
    alu alu_inst(
        .a(a_out),
        .b(b_out),
        .sub(alu_sub),
        .result(alu_out)
    );
        
    wire ir_load;
    wire ir_en;
    wire [3:0] opcode;
    wire [3:0] operand;
    
    ir ir_inst(
        .clk(clk),
        .reset(reset),
        .load(ir_load),
        .instruction_in(bus),
        .opcode(opcode),
        .operand(operand)
    );

    wire out_load;
    wire [7:0] out_out;
    
    out out_inst(
        .clk(clk),
        .reset(reset),
        .load(out_load),
        .data_in(bus),
        .data_out(out_out)
    );    
    
    wire [12:0] cs = {
            out_load,       // 12
            halt,           // 11
            pc_inc,         // 10
            pc_en,          // 9
            mar_load,       // 8
            mem_en,         // 7
            ir_load,        // 6
            ir_en,          // 5
            a_load,         // 4
            a_en,           // 3
            b_load,         // 2
            alu_sub,        // 1
            alu_en          // 0
        };

    assign cs_led = cs;
    wire [2:0] t_state;
    
    controller controller_inst(
        .clk(clk),
        .reset(reset),
        .opcode(opcode),
        .cs(cs),
        .tstate(t_state)
    );

    assign dp = 1; // We don't want to display the decimal point
    
    wire [31:0] disp_value;
    
    assign disp_value = {
                pc_out,             // 8th 7-segment LED
                1'b0, t_state,      // 7th 7-segment LED
                bus,                // 6th and 5th 7-segment LED
                mem_out,            // 4th and 3rd 7-segment LED
                out_out              // 2nd and 1st 7-segment LED
              };
     
    // Instantiate the seven segment display module
    seven_seg_display display_inst (
        .clk(clk_in),
        .reset(reset),
        .value(disp_value),
        .seg(seg),
        .an(anode)
    );
    
endmodule
```

## References

* Digital Computer Electronics Paperback - Albert Malvino And Jerald Brown
* Learning Digital Design - digilent.com/reference/learn/courses/digital-projects/start
* Stacey's Channel - FPGAs for Beginners - youtube.com/@FPGAsforBeginners
* HDLBits - hdlbits.01xz.net/wiki/Problem_sets
* Nexys A7 Reference Manual - digilent.com/reference/programmable-logic/nexys-a7/reference-manual
* Austin Morlan's Blog - austinmorlan.com/posts/8bit_breadboard_fpga/
* ECE 3300 - Digital Circuit Design using Verilog - Prof. Anas Salah Eddin - youtube.com/watch?v=Kt-78I-NUgY&list=PL-iIOnHwN7NXw01eBDR7wI8KzGK4mu8Sr&ab_channel=AnasSalahEddin