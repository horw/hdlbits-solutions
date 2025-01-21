


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



### Always blocks

```verilog
module top_module(
    input a, 
    input b,
    output wire out_assign,
    output reg out_alwaysblock
);
assign out_assign = a & b ;
always @(*) out_alwaysblock = a & b ;
endmodule
```


### Always block clock

```verilog
// synthesis verilog_input_version verilog_2001
module top_module(
    input clk,
    input a,
    input b,
    output wire out_assign,
    output reg out_always_comb,
    output reg out_always_ff   );
	
    assign out_assign = a & b ;
	always @(*) out_always_comb = a & b ;
    always @(posedge clk) 
        out_always_ff = a & b;    
endmodule
```


## If statement

```verilog
module top_module(
    input a,
    input b,
    input sel_b1,
    input sel_b2,
    output wire out_assign,
    output reg out_always   
); 
    // Continuous assignment for wire
    assign out_assign = (sel_b1 & sel_b2) ? b : a;
    
    // Always block for reg
    always @(*) begin
        if (sel_b1 & sel_b2) begin
            out_always = b;
        end
        else begin
            out_always = a;
        end
    end
endmodule

```


### If statement latches

```verilog
module top_module (
    input      cpu_overheated,
    output reg shut_off_computer,
    input      arrived,
    input      gas_tank_empty,
    output reg keep_driving  
);
    always @(*) begin
        if (cpu_overheated)
            shut_off_computer = 1;
        else
            shut_off_computer = 0;
    end

    always @(*) begin
        if (~arrived)
            keep_driving = ~gas_tank_empty;
        else
            keep_driving = 0;
    end
endmodule
```


### Case statement

```
// synthesis verilog_input_version verilog_2001
module top_module ( 
    input [2:0] sel, 
    input [3:0] data0,
    input [3:0] data1,
    input [3:0] data2,
    input [3:0] data3,
    input [3:0] data4,
    input [3:0] data5,
    output reg [3:0] out   
);

    always@(*) begin 
        case(sel)
            3'b000: out = data0;
            3'b001: out = data1;
            3'b010: out = data2;
            3'b011: out = data3;
            3'b100: out = data4;
            default: out = data5; 
        endcase
    end
endmodule
```


### Priority encoder


```
module top_module (
    input [3:0] in,
    output reg [1:0] pos  );
    always @(*) begin
        if (in[0]) pos = 2'd0;
        else if (in[1]) pos = 2'd1;
        else if (in[2]) pos = 2'd2;
        else if (in[3]) pos = 2'd3;    
        else pos = 2'd0; 
    end
endmodule
```


### Priority encoder with casez

```
module top_module (
    input [7:0] in,
    output reg [2:0] pos );
    
        always @(*) begin
            casez (in[7:0])
                8'bzzzzzzz1: pos = 0; 
                8'bzzzzzz1z: pos = 1;
                8'bzzzzz1zz: pos = 2;
                8'bzzzz1zzz: pos = 3;
                8'bzzz1zzzz: pos = 4;
                8'bzz1zzzzz: pos = 5;
                8'bz1zzzzzz: pos = 6;
                8'b1zzzzzzz: pos = 7;
            default: pos = 0;
        	endcase
        end
endmodule
```

### Avoiding latches

```
module top_module (
    input [15:0] scancode,
    output reg left,
    output reg down,
    output reg right,
    output reg up  ); 
    
    always @(*) begin
        left = 1'b0;
        down = 1'b0;
        right = 1'b0;
        up = 1'b0;
        
        case (scancode)
            16'he06b: left = 1'b1; 
            16'he072: down = 1'b1; 
            16'he074: right = 1'b1; 
            16'he075: up = 1'b1; 
            default: begin
                left = 1'b0;
                down = 1'b0;
                right = 1'b0;
                up = 1'b0;
            end
        endcase
    end

        
endmodule
```