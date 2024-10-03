---
layout: page
title: Design of SAP-1 Computer using FPGA
subtitle: Digital Electronics Club Project
---
##### (Active - Work in progress)
# Malvino's SAP (Simple-As-Possible) Computer
## Architecture
## Clock 
The clock on the Nexys A7 development board runs at 100 MHz, which is too fast for our purposes, as our primary focus is to visualize state transitions. To effectively debug the execution, we need to slow the clock down to approximately 1 Hz. Additionally, we want to be able to step through the states manually. 

We can use one of the 16 slide switches available on the development board to toggle between the automatic slow clock and manual clock modes. Additionally, we can assign one of the push buttons to manually advance the clock. However, since these are mechanical switches, sliding the switch or pressing the button may produce noisy signals, which the FPGA could misinterpret as multiple transitions. Therefore, we need to debounce the switch transitions to ensure reliable operation.

To achieve this, we need the following:

1. Slow down the system clock.
2. Advance the clock with a button press.
3. Switch between the automatic slow clock and manual clock control.
4. Halt the clock when our computer completes its program.
5. Reset the clock and the debounce logic.
6. Drive an LED to indicate the clock pulse.

### Verilog Code for our clock

```verilog
module clock (
    input wire clk_in,        // Input clock (100 MHz from Nexys A7-100)
    input wire reset,         // Reset signal
    input wire mode_switch,   // Slide switch for single-step (1) or slow clock (0)
    input wire step_button,   // Push button for single-step mode
    input wire halt,          // halt flag to end the program
    output reg clk_out       // Output clock (slowed or single-step)
);

    parameter DIVISOR = 50000000;  // Parameter to slow the clock (adjustable)
    wire debounced_step_button;         // Debounced button signal
    wire debounced_mode_switch;         // Debounced slide switch

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

### Verilog Code for debouncing logic

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

### Constraints Definition

```
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

## Program Counter
## Memory (RAM)
## Registers
### Memory Address Register (MAR)
### Instruction Register (IR)
### Accumulator - A Register
### B Register
### Output Register
## Arithmetic and Logic Unit (ALU)
## Bus
## Control Sequencer
### Micro Code

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

```
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

```
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
