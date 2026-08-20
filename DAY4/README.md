# Day 4 — Gate-Level Simulation, Blocking vs Non-Blocking Assignments, and Synthesis-Simulation Mismatch

## Overview

This experiment introduces:

- Gate-Level Simulation (GLS)
- RTL vs GLS simulation
- Synthesis-simulation mismatch
- Incomplete sensitivity lists
- Blocking assignments
- Non-blocking assignments
- Yosys synthesis
- SDF-based timing simulation

The main objective is to understand how RTL coding practices affect simulation and synthesized hardware behavior.

---

# 1. Gate-Level Simulation

## What is Gate-Level Simulation?

**Gate-Level Simulation (GLS)** is the simulation of the synthesized gate-level netlist instead of the original RTL.

The synthesized design may contain:

- Multiplexers
- AND gates
- OR gates
- NOT gates
- Flip-flops
- Standard cells

GLS is used to check:

- Post-synthesis functionality
- RTL-to-netlist behavior
- Synthesis mismatches
- Standard-cell behavior
- Timing behavior when SDF is used

### GLS Flow

```text
RTL Design
     ↓
RTL Simulation
     ↓
Synthesis
     ↓
Gate-Level Netlist
     ↓
Gate-Level Simulation
     ↓
Waveform Comparison
```

---

# 2. RTL Simulation vs Gate-Level Simulation

## RTL Simulation

RTL simulation executes the original Verilog code.

It includes constructs such as:

- `always`
- `if`
- `case`
- Blocking assignments
- Non-blocking assignments
- Continuous assignments

## Gate-Level Simulation

Gate-Level Simulation executes the synthesized netlist containing gates and standard-cell models.

### Main Difference

RTL simulation verifies the behavior of the **RTL description**, while GLS verifies the behavior of the **synthesized hardware structure**.

---

# 3. Incomplete Sensitivity List

Consider the following RTL:

```verilog
module ternary_operator_mux (
    input i0,
    input i1,
    input sel,
    output reg y
);

always @(sel) begin
    if (sel)
        y = i1;
    else
        y = i0;
end

endmodule
```

The intended functionality is:

```text
sel = 0 → y = i0
sel = 1 → y = i1
```

However, the sensitivity list contains only:

```verilog
@(sel)
```

The inputs `i0` and `i1` are missing.

## Problem

The `always` block executes only when `sel` changes.

If `i0` changes while `sel` remains unchanged:

```text
i0 changes
     ↓
sel does not change
     ↓
always block is not triggered
     ↓
y does not update
```

This can cause incorrect RTL simulation behavior.

---

## Correct Sensitivity List

Traditional Verilog:

```verilog
always @(i0 or i1 or sel)
```

Preferred Verilog:

```verilog
always @(*)
```

SystemVerilog:

```systemverilog
always_comb
```

For combinational logic, `always @(*)` is preferred when using Verilog.

---

# 4. Practical Sensitivity-List Experiment

## Testbench

```verilog
`timescale 1ns / 1ps

module tb_ternary_operator_mux;

reg i0, i1, sel;
wire y;

ternary_operator_mux uut (
    .i0(i0),
    .i1(i1),
    .sel(sel),
    .y(y)
);

initial begin

    $dumpfile("tb_ternary_operator_mux.vcd");
    $dumpvars(0, tb_ternary_operator_mux);

    sel = 0;
    i0 = 0;
    i1 = 0;
    #10;

    i0 = 1;
    #10;

    sel = 1;
    #10;

    i1 = 1;
    #10;

    $finish;

end

endmodule
```

## Simulation

```bash
iverilog ternary_operator_mux.v tb_ternary_operator_mux.v -o rtl_mux.out
./rtl_mux.out
gtkwave tb_ternary_operator_mux.vcd
```

---

# 5. Synthesis Using Yosys

The RTL can be synthesized using Yosys.

```text
yosys

read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

read_verilog ternary_operator_mux.v

synth -top ternary_operator_mux

abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

write_verilog -noattr ternary_operator_mux_netlist.v

exit
```

The synthesized design represents the hardware structure generated from the RTL.

Conceptually:

```text
           +-------+
i0 ------->|       |
i1 ------->|  MUX  |------> y
sel ------>|       |
           +-------+
```

---

# 6. Gate-Level Simulation

The synthesized netlist can be simulated using the standard-cell Verilog models.

## Compile

```bash
iverilog \
../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
ternary_operator_mux_netlist.v \
tb_ternary_operator_mux.v \
-o gls_mux.out
```

## Run

```bash
./gls_mux.out
```

## View Waveform

```bash
gtkwave tb_ternary_operator_mux.vcd
```

The RTL and GLS waveforms can then be compared.

---

# 7. Blocking Assignment Execution Order

Blocking assignment uses:

```verilog
=
```

Consider:

```verilog
module blocking_caveat (
    input a,
    input b,
    input c,
    output reg d
);

reg x;

always @(*) begin
    d = x & c;
    x = a | b;
end

endmodule
```

The simulator executes the statements sequentially.

First:

```verilog
d = x & c;
```

Then:

```verilog
x = a | b;
```

Therefore, `d` uses the previous value of `x`.

The intended data flow is:

```text
x = a | b
d = x & c
```

which is equivalent to:

```text
d = (a | b) & c
```

---

# 8. Correct Blocking Assignment Order

A safer implementation is:

```verilog
always @(*) begin
    x = a | b;
    d = x & c;
end
```

The data flow becomes:

```text
a,b
 ↓
OR
 ↓
x
 ↓
AND ← c
 ↓
d
```

Another option is continuous assignment:

```verilog
assign x = a | b;
assign d = x & c;
```

---

# 9. Blocking vs Non-Blocking Assignments

## Blocking Assignment

Syntax:

```verilog
=
```

The assignment takes effect immediately during procedural execution.

Example:

```verilog
always @(*) begin
    x = a | b;
    y = x & c;
end
```

Blocking assignments are generally used for **combinational logic**.

---

## Non-Blocking Assignment

Syntax:

```verilog
<=
```

The update is scheduled for the end of the current simulation time step.

Example:

```verilog
always @(posedge clk) begin
    q <= d;
end
```

Non-blocking assignments are generally used for **sequential logic**.

---

# 10. Why Non-Blocking Assignments Are Used for Sequential Logic

Consider two flip-flops connected in series:

```verilog
always @(posedge clk) begin
    q1 <= d;
    q2 <= q1;
end
```

At the clock edge:

- `q1` receives the previous value of `d`
- `q2` receives the previous value of `q1`

This correctly models sequential hardware.

```text
d
|
v
+-------+       +-------+
|  FF1  |------>|  FF2  |----> q2
+-------+       +-------+
    |
    q1
```

---

# 11. Coding Rules

## Combinational Logic

Use:

```verilog
always @(*) begin
    ...
end
```

Use blocking assignment:

```verilog
=
```

Example:

```verilog
always @(*) begin
    x = a | b;
    y = x & c;
end
```

## Sequential Logic

Use:

```verilog
always @(posedge clk) begin
    ...
end
```

Use non-blocking assignment:

```verilog
<=
```

Example:

```verilog
always @(posedge clk) begin
    q <= d;
end
```

### Quick Rule

```text
Combinational → always @(*) → Blocking (=)

Sequential   → always @(posedge clk) → Non-Blocking (<=)
```

---

# 12. RTL and GLS Verification Flow

```text
RTL Verilog
     ↓
RTL Simulation
     ↓
Check Functionality
     ↓
Synthesis
     ↓
Gate-Level Netlist
     ↓
GLS Simulation
     ↓
Compare Waveforms
     ↓
Match / Mismatch
     ↓
Debug if Required
```

## Icarus Verilog Commands

### RTL Simulation

```bash
iverilog <design.v> <testbench.v> -o sim.out
./sim.out
gtkwave <dumpfile.vcd>
```

### GLS

```bash
iverilog \
../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
<netlist.v> \
<testbench.v> \
-o gls.out
```

Run:

```bash
./gls.out
```

View:

```bash
gtkwave <dumpfile.vcd>
```

---

# 13. SDF Timing Simulation

GLS can also be combined with **SDF back-annotation** to analyze timing behavior.

SDF can contain information about:

- Cell delays
- Interconnect delays
- Propagation delays
- Setup violations
- Hold violations

### Timing Flow

```text
RTL
 ↓
Synthesis
 ↓
Gate-Level Netlist
 ↓
SDF
 ↓
Timing Simulation
```

---

# 14. RTL vs GLS Coding Guidelines

| Issue | Cause | Recommended Fix |
|---|---|---|
| Incomplete Sensitivity List | Signals used inside the block are missing | Use `always @(*)` |
| Blocking Assignment Order | Signal is used before being updated | Follow proper data-flow order |
| Sequential Race | Blocking assignment used in clocked logic | Use non-blocking `<=` |
| Combinational Latch | Output is not assigned in every condition | Assign outputs in all conditions |
| RTL/GLS Mismatch | RTL behavior differs from synthesized hardware | Compare RTL and GLS waveforms |
| Timing Mismatch | RTL does not model actual gate delays | Use GLS with SDF |

---

# 15. Learning Outcome

After completing Day 4, I learned:

- What Gate-Level Simulation is
- Difference between RTL simulation and GLS
- Why GLS is performed after synthesis
- How synthesis-simulation mismatches can occur
- Problems caused by incomplete sensitivity lists
- Why `always @(*)` is used for combinational logic
- Difference between blocking and non-blocking assignments
- Why blocking assignments are used for combinational logic
- Why non-blocking assignments are used for sequential logic
- How assignment ordering affects simulation
- How Yosys generates a gate-level netlist
- How Icarus Verilog performs RTL and GLS simulation
- How GTKWave is used for waveform analysis
- The purpose of SDF timing simulation

---

# 16. Conclusion

Day 4 provided practical understanding of **Gate-Level Simulation, synthesis-simulation mismatch, sensitivity-list problems, and blocking versus non-blocking assignments**.

The complete flow is:

```text
RTL Design
    ↓
RTL Simulation
    ↓
Synthesis
    ↓
Gate-Level Netlist
    ↓
Gate-Level Simulation
    ↓
Waveform Comparison
    ↓
Timing Analysis
```

DAY4 completed
