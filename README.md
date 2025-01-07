# Verilog Project: Arithmetic Circuit with Adders and Multiplexers

## Description

This repository contains Verilog code for an arithmetic circuit that performs various operations using half adders, full adders, and multiplexers. The main circuit module combines these components to process 3-bit inputs based on a 2-bit selector signal. A testbench is included to simulate and verify the functionality of the circuit across all possible input combinations.

## Table of Contents

- [Modules](#modules)
  - [Half Adder](#half-adder)
  - [Full Adder](#full-adder)
  - [Multiplexer](#multiplexer)
  - [Main Circuit](#main-circuit)
- [Testbench](#testbench)
- [Simulation](#simulation)
  - [Prerequisites](#prerequisites)
  - [Running the Simulation](#running-the-simulation)
- [Files](#files)
- [License](#license)

## Modules

### Half Adder

The **half adder** module performs binary addition of two single-bit inputs `a` and `b`. It outputs a `sum` and a `carry`.

```verilog
module half_adder(output sum, output carry, input a, input b);
    xor(sum, a, b);
    and(carry, a, b);
endmodule
```

- **Inputs:**
  - `a`, `b`: Single-bit inputs to be added.
- **Outputs:**
  - `sum`: Sum of the inputs.
  - `carry`: Carry-out of the addition.

### Full Adder

The **full adder** module adds three single-bit inputs: `a`, `b`, and `cin` (carry-in). It outputs a `sum` and a `carry`.

```verilog
module full_adder(output sum, output carry, input a, input b, input cin);
    wire sum1, c1, c2;
    half_adder h1(sum1, c1, a, b);
    half_adder h2(sum, c2, sum1, cin);
    or(carry, c1, c2);
endmodule
```

- **Inputs:**
  - `a`, `b`: Single-bit inputs to be added.
  - `cin`: Carry-in from the previous addition.
- **Outputs:**
  - `sum`: Sum of the inputs and carry-in.
  - `carry`: Carry-out of the addition.

### Multiplexer

The **multiplexer** module selects one of four inputs (`i0`, `i1`, `i2`, `i3`) based on a 2-bit selector `s` and outputs it as `y`.

```verilog
module mux(input i0, input i1, input i2, input i3, input [1:0] s, output y);
    wire [3:0] terms;
    and(terms[0], ~s[1], ~s[0], i0);
    and(terms[1], ~s[1],  s[0], i1);
    and(terms[2],  s[1], ~s[0], i2);
    and(terms[3],  s[1],  s[0], i3);
    or(y, terms[0], terms[1], terms[2], terms[3]);
endmodule
```

- **Inputs:**
  - `i0`, `i1`, `i2`, `i3`: Data inputs.
  - `s`: 2-bit selector signal.
- **Output:**
  - `y`: Output corresponding to the selected input.

### Main Circuit

The **main circuit** module combines the multiplexers and full adders to process 3-bit inputs `a` and `b`, based on a selector `s`, and produces a 3-bit output `y` and carries `carry`.

```verilog
module circuit(
    input [2:0] a, 
    input [2:0] b, 
    input [1:0] s, 
    output [2:0] y, 
    output [2:0] carry
);
    wire [5:0] muxOutput;

    mux m0(a[0], a[0], b[0], 1'b1, s, muxOutput[0]);
    mux m1(a[1], a[1], b[1], 1'b0, s, muxOutput[1]);
    mux m2(a[2], a[2], b[2], 1'b0, s, muxOutput[2]);
    mux m3(1'b1, b[0], a[0], b[0], s, muxOutput[3]);
    mux m4(1'b0, b[1], a[1], b[1], s, muxOutput[4]);
    mux m5(1'b0, b[2], a[2], b[2], s, muxOutput[5]);

    full_adder f1(y[0], carry[0], muxOutput[0], muxOutput[3] ^ s[1], s[1]);
    full_adder f2(y[1], carry[1], muxOutput[1], muxOutput[4] ^ s[1], carry[0]);
    full_adder f3(y[2], carry[2], muxOutput[2], muxOutput[5] ^ s[1], carry[1]);
endmodule
```

- **Inputs:**
  - `a`, `b`: 3-bit operands.
  - `s`: 2-bit selector to control the operation.
- **Outputs:**
  - `y`: 3-bit result of the operation.
  - `carry`: Carry-out bits for each full adder stage.

## Testbench

The **testbench** module `test_circuit` is used to simulate the main circuit. It systematically tests all combinations of inputs `a`, `b`, and `s`.

```verilog
module test_circuit();
    reg [2:0] a, b;
    reg [1:0] s;
    wire [2:0] y, carry;

    circuit c(a, b, s, y, carry);

    initial begin
        $dumpfile("circuit.vcd");
        $dumpvars(0, test_circuit);
        $monitor($time, " a=%b, b=%b, s=%b, y=%b, carry=%b", a, b, s, y, carry);

        for (integer i = 0; i < 8; i = i + 1) begin
            for (integer j = 0; j < 8; j = j + 1) begin
                for (integer k = 0; k < 4; k = k + 1) begin
                    a <= i;
                    b <= j;
                    s <= k;
                    #1;
                end
            end
        end
    end
endmodule
```

- **Purpose:** To verify the correctness of the circuit by simulating all possible input combinations.
- **Simulation Outputs:** Generates a Value Change Dump (VCD) file `circuit.vcd` for waveform analysis.

## Simulation

### Prerequisites

- **Verilog Simulator:** Icarus Verilog (iverilog) or any compatible Verilog simulation tool.
- **Waveform Viewer:** GTKWave (optional, for viewing simulation waveforms).

### Running the Simulation

1. **Clone the Repository:**

   ```bash
   git clone https://github.com/KareemMagdy55/Implementing-Adders-Using-Verilog.git
   cd yourrepository
   ```

2. **Compile the Verilog Code:**

   ```bash
   iverilog -o circuit_tb test_circuit.v
   ```

   > Ensure all module files (`half_adder.v`, `full_adder.v`, etc.) are included or combined into `test_circuit.v`.

3. **Run the Simulation:**

   ```bash
   vvp circuit_tb
   ```

   This command executes the compiled simulation and generates `circuit.vcd`.

4. **View Simulation Output:**

   - **Console Output:** The `$monitor` statement will print the values of `a`, `b`, `s`, `y`, and `carry` at each time step.

   - **Waveform Analysis (Optional):**

     ```bash
     gtkwave circuit.vcd
     ```

     Use GTKWave to open the VCD file and visualize signal changes over time.

## Files

- **half_adder.v:** Contains the half adder module.
- **full_adder.v:** Contains the full adder module.
- **mux.v:** Contains the multiplexer module.
- **circuit.v:** Contains the main circuit module.
- **test_circuit.v:** Contains the testbench for simulating the circuit.
- **circuit.vcd:** Value Change Dump file generated after simulation (not included in the repository).
