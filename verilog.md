# Verilog Syntax:

```v
//https://www.chipverify.com/verilog/verilog-syntax
// SYNTAX:

// COMMENTS
    // This is a single line comment

    /* 
    This is a 
    multi comment
    or block comment
    */

//WHITESPACE
____ //This is indentation of code for making it easy to read

//OPERATORS

    x = ~y; // ~ unary operator
    x = y | z; // | binary operator
    x = (y > 6) ? w : z; // ? ternary operator

//NUMBER FORMAT:

    16 //16 in decimal
    0x10 //16 in hexadecimal 
    10000 //16 in binary
    20  //16 in octal

// Sized number format:
    [size]'[base_format] [number] //syntax
    3'b010 //size is 3, base_format is binary, the number 010 (binary) is 2 in decimal
    3'd2 //size is 3, base_format is decimal, the number 2 (decimal)
    8'h1FA //size is 8, base_format is hexadecimal, the number 1FA (hex) is 506 in decimal



//STRINGS:
    "Hello World!!" //string with 13 characters


//IDENTIFIERS (variable names)
    integer var_a;  //identify contains alphabets and underscore -> valid
    integer $var_a; //identifier starts with $ -> invalid
    integer v$ar_a; //identifier contains alphabets and $ -> valid
    integer 2var;   //identifier starts with a digit -> invalid
    integer var23_g;   //identifier contains alphanumeric characters and underscore -> valid
    integer 23;      //identifier contains only numbers -> invalid

//https://athena.ecs.csus.edu/~arad/csc142/intro_verilog_hdl.pdf

```

## Simulation and Synthesis

- The two major purposes of HDLs are logic 	simulation and synthesis
    – During simulation, inputs are applied to a module, and the outputs are checked to verify that the module operates correctly
    – During synthesis, the textual description of a module is transformed into logic gates
- Circuit descriptions in HDL resemble code in a programming language. But the code is intended to represent hardware 
- Not all of the Verilog commands can be synthesized into hardware
- Our primary interest is to build hardware, we will emphasize a synthesizable subset of the language
- Will divide HDL code into synthesizable modules and a test bench (simulation).
    – The synthesizable modules describe the hardware.
    – The test bench checks whether the output results are correct (only for simulation and cannot be synthesized)

### Logical values

A bit can have any of these values

- 0 representing logic low (false)  
- 1 representing logic high (true)  
- X representing either 0, 1, or Z  
- Z representing high impedance for tri-state (unconnected inputs are set to Z)  

---

## Data

### Reg(```reg```)

A reg (```reg```) stores its value from one assignment to the next (model data storage elements)

- Don’t confuse reg with register
- Default value is X
- Default range is one bit
- By default are unsigned, but can be declare signed, using keyword signed

### Nets(```wire```)
  
Nets (```wire```) correspond to physical wires that connect instances

- Nets do not store values
- Have to be continuously driven
- The default range is one bit
- By default are unsigned

### Vectors:

```html
<data_type> [left range : right range] <Variable_name>
```

Examples:

```v
reg [0:7] A, B; //Two 8-bit reg with MSB as the 0th bit
wire [3:0] Data; //4-bit wide wire MSB as the 4th bit
```

Vector part select (access) :

```v
A[5] // bit # 5 of vector A
Data[2:0] // Three LSB of vector Data
```

---

## Module declaration

```v
module <module name> #(<param list>) (<port list>);
    <Declarations>
    <Instantiations>
    <Data flow statements>
    <Behavioral blocks>
    <task and functions>
endmodule
```

## Procedural blocks

Each procedural block represent a separate activity flow in Verilog.

Procedural blocks:

- ```always``` blocks  
  - To model a block of activity that is repeated continuously  
- ```initial``` blocks simulation only  
  - To model a block of activity that is executed at the beginning  
  
Multiple behavioral statements can be grouped using keywords begin and end  

### Procedural assignments

- Procedural assignment changes the state of a reg
- Used for both combinational and sequential logic inference
- All procedural statements must be within always (or inital) block  
  
Example:

```v
    reg A;
    always @ (B or C)
    begin
        A = ~(B & C);
    end
```
## Different levels of Verilog modelling

```v
//https://www.vlsifacts.com/different-coding-styles-verilog-language/
//BEHAVORIAL
module Mux_4to1(
   input [3:0] i,
   input [1:0] s,
   output reg o
);
 
always @(s or i)
begin
   case (s)
      2'b00 : o = i[0];
      2'b01 : o = i[1];
      2'b10 : o = i[2];
      2'b11 : o = i[3];
      default : o = 1'bx;
   endcase
end
endmodule
```

```v
//DATAFLOW
module Mux_4to1_df(
    input [3:0] i,
    input [1:0] s,
    output o
    );
    
    assign o = (~s[1] & ~s[0] & i[0]) | (~s[1] & s[0] & i[1]) | (s[1] & ~s[0] & i[2]) | (s[1] & s[0] & i[3]);
endmodule
```

```v
//GATE LEVEL
module Mux_4to1_gate(
        input [3:0] i,
        input [1:0] s,
        output o
        );
    
    wire NS0, NS1;
    wire Y0, Y1, Y2, Y3;
    not N1(NS0, s[0]);
    not N2(NS1, s[1]);
    and A1(Y0, i[0], NS1, NS0);
    and A2(Y1, i[1], NS1, s[0]);
    and A3(Y2, i[2], s[1], NS0);
    and A4(Y3, i[3], s[1], s[0]);
    or O1(o, Y0, Y1, Y2, Y3);
endmodule

```

## Blocking / Non-Blocking assignment

### Blocking assignment

- (```=``` operator) acts much like in traditional programming languages
- The whole statement is done before control passes on to the next statement.

### Non-blocking assignment

- (```<=``` operator) Evaluates all the right-hand sides for the current time unit and assigns the lefthand sides at the end of the time unit.

### Conditional statements (```if``` … ```else```)

- The statement occurs if the expressions controlling the if statement evaluates to true
  - True: 1 or non-zero value
  - False: 0 or ambiguous (X)
- Explicit priority

Syntax:

```v
    if (<expression>)
    // statement1
    else if (<expression>)
    // statement2
    else
    // statement3
```

Example:

```v
always @ (WRITE or STATUS)
begin
        if (!WRITE)
        begin
            out = oldvalue;
        end
        else if (!STATUS)
        begin
            q = newstatus;
        end
end
```

## Conditional statements (case)

```case```: case statements are used for switching between multiple selections  

- If there are multiple matches only the first is evaluated  
- Breaks automatically

Syntax:

```v
case (<expression>)
    <alternative 1> : <statement 1>;
    <alternative 2> : <statement 2>;
    default : <default statement>;
endcase
```

Example:

```v
always @(s, a, b, c, d)
case (s)
    2'b00: out = a;
    2'b01: out = b;
    2'b10: out = c;
    2'b11: out = d;
endcase
```

## Loop statements (for)

- Works the same ways as C
- Unary increment/decrement is not allowed

```v
for (<loop var init>; <loop var reentry expr>; <loop var update>) <statement>;
```

Example:

```v
// General purpose loop
interger i;
always @*
for (i = 0 ; i < 7 ; i=i+1)
    memory[i] = 0;

Loop statements (while)
- Loop execute until the expression is not true

Example:
always @*
while(delay)
// multiple statement groups with begin-end
begin
    ldlang = oldldlang;
    delay = delay – 1;
end

```

## Writing Test Bench

- A test bench specifies a sequence of inputs to be
applied by the simulator to an Verilog-based
design.
- The test bench uses an initial block and delay
statements and procedural statement.
- Verilog has advanced “behavioral” commands to
facilitate this:  
  - Delay for n units of time  
  - Full high-level constructs: if, while, sequential assignment.
  - Input/output: file I/O, output to display, etc.

```v
//MODULE
module DUT (in1, in2, clk, out1);
    input in1, in2;
    input clk;
    output reg out1;
    always @(posedge clk)
        out1 = in1^in2;
endmodule
```

```v
//TEST BENCH
`timescale 10ns/1ps
module test_bench;
    // Interface to communicate with the DUT
    reg a, b, clk;
    wire c;
    // Device under test instantiation
    DUT U1 (.in1(a), .in2(b), .clk(clk), .out1(c));
    initial
    begin // Test program
            test1 ();
            $finish;
    end
    initial
    begin
            clk = 0;
            forever #5 clk = ~clk;
    end
    initial
    begin // Monitor the simulation
        $dumpvars;
            $display ("clk | in1| in2 | out1 |");
            $monitor (" %b| %b | %b | %b |",clk, a, b, c);
    end
endmodule
```
