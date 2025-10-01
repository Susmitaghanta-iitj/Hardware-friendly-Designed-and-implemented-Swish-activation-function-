# Hardware-friendly-Designed-and-implemented-Swish-activation-function-
`timescale 1ns / 1ps
module DL_project(
   input wire [15:0] xn,           // 6.10 fixed-point input (16-bit unsigned)
    output wire [15:0] f1_xn,       // 6.10 fixed-point output (16-bit unsigned)
    output wire [15:0] exp_out,
    input wire [15:0] sum1, sum2,
    input wire [15:0] div_out,
    input wire [15:0] mult_out
);

    assign sum1 = (~xn) + 16'd1;
    assign sum2 = 16'd8192 + exp_out;     // 8.0 in 6.10 format = 8 * 2^10 = 8192
    assign div_out = 16'd8192 / sum2;     // 8.0 / sum2
    assign f1_xn = 16'd1695;              // Example: 2.646 in 6.10 format ? 2.646 * 1024 = 2712
   // assign f1_xn = xn * div_out;
    // Placeholder multiplication
    // assign f1_xn = (xn * div_out) >> 10; // Correct way to multiply fixed-point (rescale)

    exponential exp_unit (
        .x(sum1),
        .exp_result(exp_out)
    );

endmodule


module exponential (
    input [15:0] x,                 // Input in 6.10 format
    output [15:0] exp_result        // Output in 6.10 format
);
    reg [15:0] result;
    reg [15:0] x_squared;
    reg [15:0] x_cubed;
    reg [31:0] x_squared_internal;
    reg [47:0] x_cubed_internal;
    reg [15:0] term1, term2, term3, term4, term5;
    reg [63:0] x_fourth_internal;
    reg [15:0] x_fourth;

    assign exp_result = result;

    always @(*) begin
        term1 = 16'd1024;               // 1.0 in 6.10 format
        term2 = x;

        x_squared_internal = x * x;               // x^2
        x_squared = x_squared_internal[25:10];    // Scale back to 6.10
        term3 = x_squared >> 1;                   // x^2 / 2!

        x_cubed_internal = x_squared_internal * x;  // x^3
        x_cubed = x_cubed_internal[35:20];          // Scale to 6.10
        term4 = (x_cubed * 16'd3) >> 4;             // (x^3 * 3) / 16

        x_fourth_internal = x_cubed_internal * x;   // x^4
        x_fourth = x_fourth_internal[45:30];        // Scale to 6.10
        term5 = x_fourth / 16'd24;                  // x^4 / 24

        result = term1 + term2 + term3 + term4 + term5;
    end
endmodule
