


### Replication operator

https://hdlbits.01xz.net/wiki/Vector4

```verilog
module top_module (
    input [7:0] in,
    output [31:0] out );
    
    assign out = {{24{in[7]}}, in};

endmodule

```

### More replication

https://hdlbits.01xz.net/wiki/Vector5

```verilog
module top_module (
    input a, b, c, d, e,
    output [24:0] out
);
    wire [24:0] replicated_inputs = { {5{a}}, {5{b}}, {5{c}}, {5{d}}, {5{e}} };
    wire [24:0] repeated_inputs   = {5{a, b, c, d, e}};

    assign out = ~(replicated_inputs) ^ repeated_inputs;

endmodule

```


### Module

https://hdlbits.01xz.net/wiki/Module

```verilog
module top_module ( input a, input b, output out );
    mod_a instance1 ( a, b, out );
endmodule
```


### Three modules

https://hdlbits.01xz.net/wiki/module_shift


```verilog
module top_module ( input clk, input d, output q );
	
    wire q1, q2;

    my_dff i1(.clk(clk), .d(d), .q(q1)); 
    my_dff i2(.clk(clk), .d(q1), .q(q2)); 
    my_dff i3(.clk(clk), .d(q2), .q(q)); 

    
endmodule
```


### Modules and vectors

https://hdlbits.01xz.net/wiki/Module_shift8

```verilog
module top_module ( 
    input clk, 
    input [7:0] d, 
    input [1:0] sel, 
    output [7:0] q 
);
    wire[7:0] f1, f2, f3;
    my_dff8 b1(clk, d, f1);
    my_dff8 b2(clk, f1, f2);
    my_dff8 b3(clk, f2, f3);
    
    always @(*) begin
        case(sel)
            2'b00: q = d;  
            2'b01: q = f1;
            2'b10: q = f2;
            2'b11: q = f3;
            default: q = 8'b0;
        endcase
    end
    
    
endmodule
```

### Adder 1

https://hdlbits.01xz.net/wiki/Module_add

```verilog
module top_module(
    input [31:0] a,
    input [31:0] b,
    output [31:0] sum
);
    wire [15:0] r1, r2;
    wire c;
    
    // First 16-bit adder
    add16 a1 (
        .a(a[15:0]), 
        .b(b[15:0]), 
        .cin(0),
        .sum(r1), 
        .cout(c)
    );
    
    add16 a2 (
        .a(a[31:16]), 
        .b(b[31:16]), 
        .cin(c),
        .sum(r2), 
        .cout()
    );
 
    assign sum = {r2, r1}; 

endmodule
```


### Adder 2

https://hdlbits.01xz.net/wiki/Module_fadd

```verilog
module top_module (
    input [31:0] a,
    input [31:0] b,
    output [31:0] sum
);
    wire c;
    add16 a1(a[15:0], b[15:0], 0, sum[15:0], c);
    add16 a2(a[31:16], b[31:16], c, sum[31:16], 0);

endmodule

module add1 ( input a, input b, input cin,   output sum, output cout );
    assign cout = (a & b) | (cin & (a ^ b));
    assign sum = a ^ b ^ cin;
endmodule


module add2 ( input[1:0] a, input[1:0] b, input cin,   output[1:0] sum, output cout );
    wire c;
    add1 a1(a[0], b[0], 0, sum[0], c);
    add1 a2(a[1], b[1], c, sum[1], 0);
endmodule


module add4 ( input[3:0] a, input[3:0] b, input cin,   output[3:0] sum, output cout );
    wire c;
    add2 a1(a[1:0], b[1:0], 0, sum[1:0], c);
    add2 a2(a[3:2], b[3:2], c, sum[3:2], 0);
endmodule


module add8 ( input[7:0] a, input[7:0] b, input cin,   output[7:0] sum, output cout );
    wire c;
    add4 a1(a[3:0], b[3:0], 0, sum[3:0], c);
    add4 a2(a[7:4], b[7:4], c, sum[7:4], 0);
endmodule
```

### 9-to-1 multiplexer

https://hdlbits.01xz.net/wiki/mux9to1v

```verilog
module top_module( 
    input [15:0] a, b, c, d, e, f, g, h, i,
    input [3:0] sel,
    output reg [15:0] out );

    always @(*) begin
        case(sel)
            4'b0000: out = a;  
            4'b0001: out = b;  
            4'b0010: out = c;  
            4'b0011: out = d;  
            4'b0100: out = e;  
            4'b0101: out = f;  
            4'b0110: out = g;  
            4'b0111: out = h;
            4'b1000: out = i;

            default: out = '1;
        endcase
    end

endmodule


```

### Carry select adder

https://hdlbits.01xz.net/wiki/Module_cseladd

```verilog
module top_module(
    input [31:0] a,
    input [31:0] b,
    output [31:0] sum
);
	
    wire c;
    add16 a1(a[15:0], b[15:0], 0, sum[15:0], c);
    wire[15:0] answer1, answer2;
    add16 a21(a[31:16], b[31:16], 0, answer1, 0);
    add16 a22(a[31:16], b[31:16], 1, answer2, 0);
    always @(*) begin
        case(c)
            1'b0: sum[31:16] = answer1;
            1'b1: sum[31:16] = answer2;
        endcase
    end
    
endmodule
```

### Adder-subtractor

https://hdlbits.01xz.net/wiki/Module_addsub

```verilog
module top_module(
    input [31:0] a,
    input [31:0] b,
    input sub,
    output [31:0] sum
);
    wire[31:0] b_mod;
    wire c;
    assign b_mod = (sub) ? (~b + 1) : b;

    add16 a1(a[15:0], b_mod[15:0], 0, sum[15:0], c);
    add16 a2(a[31:16], b_mod[31:16], c, sum[31:16], 0);
    
endmodule
```
