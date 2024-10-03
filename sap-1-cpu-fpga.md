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

### Verilog Code for our clock

```
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

```
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

## Display
### 7 Segment LED Display
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

