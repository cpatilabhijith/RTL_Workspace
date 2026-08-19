# Day 3: Combinational and Sequential RTL Optimizations

## Overview

Day 3 focuses on the optimization of RTL designs using **Yosys**. The module covers both **combinational and sequential optimization techniques** and demonstrates how synthesis tools identify redundant logic, propagate constants, simplify Boolean expressions, and remove hardware that does not contribute to the required outputs.

The experiments use the **SKY130 standard-cell library** to observe how optimized RTL can be transformed into technology-specific hardware.

---

## Contents

1. Introduction to RTL Optimization
2. Yosys Optimization Setup
3. Combinational Logic Optimization
4. Sequential Logic Optimization
5. Sequential Optimization with Unused Outputs
6. Technology Mapping
7. Optimization Command Reference
8. Overall Optimization Flow
9. Learning Outcomes
10. Conclusion

---

# 1. Introduction to RTL Optimization

RTL optimization is the process of improving a hardware description while preserving its intended functionality.

The synthesis tool analyzes the RTL and identifies opportunities to reduce unnecessary hardware.

### Main Objectives of Optimization

- Reduce hardware area
- Remove redundant logic
- Simplify Boolean expressions
- Eliminate unused registers
- Reduce circuit complexity
- Improve power efficiency
- Enable efficient technology mapping

RTL optimization can mainly be divided into two categories:

```text
                 RTL Optimization
                       │
              ┌────────┴────────┐
              │                 │
              ▼                 ▼
       Combinational       Sequential
        Optimization       Optimization
              │                 │
              ▼                 ▼
       Logic Simplification   FF/Register
       Constant Propagation   Optimization
       Redundancy Removal     State Optimization
```

---

# 2. Yosys Optimization Setup

Before performing the optimization experiments, the RTL design is loaded into **Yosys** along with the required SKY130 standard-cell library.

## Starting Yosys

```bash
yosys
```

## Loading the Standard-Cell Library

```text
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

The Liberty file provides the characterization information required for technology mapping.

## Reading the RTL

```text
read_verilog <design_name>.v
```

This command loads the Verilog design into Yosys.

## Selecting the Top Module

```text
synth -top <module_name>
```

The specified module is treated as the top-level design during synthesis.

## Optimization

```text
opt_clean -purge
```

This removes unused wires, cells, and other unreferenced logic.

## Flip-Flop Mapping

```text
dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

This maps generic flip-flops to suitable flip-flop cells available in the SKY130 library.

## Technology Mapping

```text
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

ABC performs technology mapping and maps the synthesized logic to available standard cells.

## View the Result

```text
show
```

This displays the synthesized design as a schematic.

---

# 3. D3SK1 – Introduction to Optimizations

Optimization allows synthesis tools to transform a complex RTL description into a simpler and more efficient hardware implementation.

## Combinational Optimization

Combinational optimization deals with logic whose output depends only on the current inputs.

Typical techniques include:

- Constant propagation
- Boolean simplification
- Removal of redundant gates
- Expression simplification
- Multiplexer simplification
- Logic minimization

Boolean minimization can be performed conceptually using techniques such as:

- Karnaugh Maps
- Quine-McCluskey method
- Algebraic simplification

The synthesis tool performs many of these transformations automatically.

---

## Sequential Optimization

Sequential optimization focuses on state-holding elements such as flip-flops and registers.

Common optimization techniques include:

- Constant-state optimization
- Removal of redundant registers
- State reduction
- Retiming
- Register merging
- Register cloning
- Elimination of unused outputs
- Removal of unreachable states

The objective is to retain the required behavior while reducing unnecessary sequential hardware.

---

# 4. D3SK2 – Combinational Logic Optimizations

Combinational optimization simplifies logic without changing the functional behavior of the circuit.

---

## Example 1 – Constant Propagation

### Verilog Code

```verilog
module opt_check (
    input a,
    input b,
    output y
);

assign y = a ? b : 1'b0;

endmodule
```

### Analysis

The expression represents:

```text
If a = 0 → y = 0
If a = 1 → y = b
```

This is equivalent to:

```text
y = a & b
```

Therefore, the synthesis tool can replace the conditional structure with simpler AND logic.

### Yosys Commands

```text
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog opt_check.v
synth -top opt_check
opt_clean -purge
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

### Optimization

```text
Conditional Logic
       ↓
Constant Analysis
       ↓
Boolean Simplification
       ↓
AND Logic
```

---

## Example 2 – Hardwired Constant Optimization

### Verilog Code

```verilog
module opt_check2 (
    input a,
    input b,
    output y
);

assign y = a ? 1'b1 : b;

endmodule
```

### Analysis

The output behavior is:

```text
If a = 1 → y = 1
If a = 0 → y = b
```

This can be simplified to:

```text
y = a | b
```

The constant value `1` allows the synthesis tool to simplify the original conditional logic.

### Yosys Commands

```text
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog opt_check2.v
synth -top opt_check2
opt_clean -purge
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

### Optimization

```text
MUX-like Logic
      ↓
Constant Propagation
      ↓
Boolean Simplification
      ↓
OR Logic
```

---

## Example 3 – Redundant Multiplexer Optimization

### Verilog Code

```verilog
module opt_check3 (
    input a,
    input b,
    input c,
    output y
);

assign y = a ? (c ? 1'b1 : b) : 1'b0;

endmodule
```

### Analysis

The logic can be simplified as:

```text
y = a & (c | b)
```

The nested conditional structure contains redundant logic that can be reduced by Boolean simplification.

### Yosys Commands

```text
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog opt_check3.v
synth -top opt_check3
opt_clean -purge
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

### Optimization Flow

```text
Nested Conditional Logic
          ↓
Constant Propagation
          ↓
Boolean Simplification
          ↓
Simplified AND/OR Structure
```

---

## Example 4 – Multi-bit Logic Optimization

### Verilog Code

```verilog
module opt_check4 (
    input a,
    input b,
    input c,
    output y
);

assign y = a ? (b ? (c ? 1'b1 : 1'b0) : 1'b0) : 1'b0;

endmodule
```

### Analysis

The output becomes `1` only when:

```text
a = 1
b = 1
c = 1
```

Therefore, the expression is equivalent to:

```text
y = a & b & c
```

The synthesis tool can implement the functionality using a three-input AND structure instead of the original nested conditional logic.

### Yosys Commands

```text
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog opt_check4.v
synth -top opt_check4
opt_clean -purge
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

---

# 5. D3SK3 – Sequential Logic Optimizations

Sequential optimization deals with circuits containing state elements such as flip-flops.

The synthesis tool examines how the state elements are used and determines whether some of them can be simplified or removed.

---

## Example 1 – Constant D-Input Flip-Flop

### Verilog Code

```verilog
module dff_const1 (
    input clk,
    input reset,
    output reg q
);

always @(posedge clk or posedge reset)
begin
    if (reset)
        q <= 1'b0;
    else
        q <= 1'b1;
end

endmodule
```

### Behavior

When reset is active:

```text
q = 0
```

After reset is released, the first active clock edge changes:

```text
q = 1
```

The output then remains at `1`.

Because the output has a reset-dependent transition, the flip-flop cannot simply be replaced by a constant in all situations.

### Yosys Commands

```text
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const1.v
synth -top dff_const1
dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
opt_clean -purge
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

---

## Example 2 – Flip-Flop Permanently Tied to a Constant

### Verilog Code

```verilog
module dff_const2 (
    input clk,
    input reset,
    output reg q
);

always @(posedge clk or posedge reset)
begin
    if (reset)
        q <= 1'b1;
    else
        q <= 1'b1;
end

endmodule
```

### Behavior

The output is always:

```text
q = 1
```

Both the reset condition and normal clocked operation assign `1`.

Therefore, the synthesis tool can recognize that the flip-flop is unnecessary and replace the output with a constant logic `1`.

Conceptually:

```text
Flip-Flop
   ↓
Always 1
   ↓
Remove Flip-Flop
   ↓
Tie output to Logic 1
```

### Yosys Commands

```text
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const2.v
synth -top dff_const2
dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
opt_clean -purge
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

---

## Example 3 – Multi-Flop Constant Chain

### Verilog Code

```verilog
module dff_const3 (
    input clk,
    input reset,
    output reg q
);

reg q1;

always @(posedge clk or posedge reset)
begin
    if (reset) begin
        q  <= 1'b0;
        q1 <= 1'b0;
    end
    else begin
        q1 <= 1'b1;
        q  <= q1;
    end
end

endmodule
```

### Behavior

This circuit demonstrates sequential propagation across clock cycles.

After reset:

```text
q  = 0
q1 = 0
```

After the first rising clock edge:

```text
q1 = 1
q  = 0
```

After the second rising clock edge:

```text
q1 = 1
q  = 1
```

Thus, the design demonstrates how sequential elements can introduce cycle-by-cycle latency.

### Yosys Commands

```text
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const3.v
synth -top dff_const3
dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
opt_clean -purge
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

---

# 6. D3SK4 – Sequential Optimization for Unused Outputs

Not every state element inside an RTL design necessarily contributes to the final output.

Synthesis tools perform **observability analysis** to determine which registers and logic are actually required to generate the outputs.

Unused sequential elements can then be eliminated.

---

## Example 1 – Counter with Unused Upper Bits

### Verilog Code

```verilog
module counter_opt (
    input clk,
    input reset,
    output q
);

reg [2:0] count;

assign q = count[0];

always @(posedge clk or posedge reset)
begin
    if (reset)
        count <= 3'b000;
    else
        count <= count + 1;
end

endmodule
```

### Analysis

The counter contains three state bits:

```text
count[2]
count[1]
count[0]
```

However, only:

```text
count[0]
```

is connected to the primary output.

The least significant bit toggles every clock cycle:

```text
0 → 1 → 0 → 1 → ...
```

The upper two bits do not influence the output `q`.

Therefore, synthesis can identify them as unnecessary for the observable behavior.

### Expected Optimization

```text
Original:
3 Flip-Flops
      ↓
Only count[0] affects output
      ↓
Unused state removed
      ↓
Reduced implementation
```

### Yosys Commands

```text
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog counter_opt.v
synth -top counter_opt
dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
opt_clean -purge
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

---

## Example 2 – Counter Where All Bits Are Required

### Verilog Code

```verilog
module counter_opt2 (
    input clk,
    input reset,
    output q
);

reg [2:0] count;

assign q = (count[2:0] == 3'b100);

always @(posedge clk or posedge reset)
begin
    if (reset)
        count <= 3'b000;
    else
        count <= count + 1;
end

endmodule
```

### Analysis

Here, the output depends on the complete counter value:

```text
count[2]
count[1]
count[0]
```

The output becomes `1` only when:

```text
count = 100
```

Since every bit is required to determine whether the counter equals `100`, the synthesis tool cannot remove any of the three state bits.

### Expected Result

```text
3-bit Counter
     ↓
All bits used by comparison
     ↓
All 3 flip-flops required
```

### Yosys Commands

```text
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog counter_opt2.v
synth -top counter_opt2
dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
opt_clean -purge
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

---

# 7. Comparing the Counter Optimizations

| Design | Output Dependency | Optimization |
|:--|:--|:--|
| `counter_opt` | Only `count[0]` | Upper state bits can be removed |
| `counter_opt2` | `count[2:0]` | All state bits are required |

This experiment demonstrates an important synthesis principle:

> Logic that cannot affect an observable output can potentially be removed during optimization.

---

# 8. Technology Mapping with SKY130

After RTL optimization, the design can be mapped to actual standard cells from the SKY130 library.

### Load Library

```text
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

### Map Flip-Flops

```text
dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

### Map Combinational Logic

```text
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

### View the Final Structure

```text
show
```

The resulting schematic represents the optimized design using cells available in the selected standard-cell library.

---

# 9. Optimization Command Reference

| Yosys Command | Purpose |
|:--|:--|
| `read_verilog` | Loads Verilog RTL into Yosys |
| `read_liberty -lib` | Loads the standard-cell Liberty library |
| `synth` | Performs RTL synthesis |
| `opt` | Runs a collection of optimization passes |
| `opt_expr` | Simplifies expressions and performs constant folding |
| `opt_clean -purge` | Removes unused and unreferenced logic |
| `dfflibmap` | Maps generic flip-flops to library-specific cells |
| `abc` | Performs technology mapping and logic optimization |
| `show` | Displays the synthesized schematic |

---

# 10. Complete Module 3 Flow

The complete optimization process studied in this module can be represented as:

```text
                 Verilog RTL
                      │
                      ▼
                Read Design
                      │
                      ▼
                   Synthesis
                      │
                      ▼
              RTL Optimization
                 ┌────┴────┐
                 │         │
                 ▼         ▼
          Combinational  Sequential
          Optimization   Optimization
                 │         │
                 └────┬────┘
                      ▼
               Remove Redundant
                   Hardware
                      │
                      ▼
                Flip-Flop Mapping
                      │
                      ▼
              Technology Mapping
                      │
                      ▼
              SKY130 Standard Cells
                      │
                      ▼
               Optimized Netlist
```

---

# 11. Key Observations

### Combinational Optimization

Complex conditional expressions can often be reduced to simpler Boolean logic.

For example:

```text
a ? b : 0
```

can become:

```text
a & b
```

---

### Sequential Optimization

A flip-flop whose output is permanently tied to a constant does not need to remain as a physical storage element.

```text
q = 1 always
      ↓
Constant Logic
```

---

### Unused State Optimization

If only one bit of a counter contributes to the output, the synthesis tool can remove state bits that have no observable effect.

---

### Dependency Matters

When the output depends on every state bit, the synthesis tool must retain those state elements.

```text
Output = f(count[2], count[1], count[0])
```

requires all three bits when each contributes to the result.

---

# 12. Learning Outcomes

After completing Module 3, I understood:

- The purpose of RTL optimization.
- The difference between combinational and sequential optimization.
- Constant propagation in combinational logic.
- Boolean simplification during synthesis.
- Reduction of redundant multiplexer structures.
- Optimization of constant-driven flip-flops.
- Sequential behavior across multiple clock cycles.
- Removal of unused state elements.
- The importance of output observability during synthesis.
- How Yosys performs optimization passes.
- The role of `opt_clean -purge`.
- The purpose of `dfflibmap`.
- The role of ABC technology mapping.
- How optimized RTL is mapped to SKY130 standard cells.

---

# 13. Conclusion

Module 3 provided practical insight into how synthesis tools improve RTL implementations without changing their required functionality.

The experiments demonstrated that optimization can occur at both the **combinational level**, by simplifying Boolean logic and removing redundant structures, and at the **sequential level**, by identifying constant or unobservable state elements.

The complete process can be summarized as:

```text
RTL Design
    ↓
Synthesis
    ↓
Optimization
    ↓
Redundancy Removal
    ↓
Technology Mapping
    ↓
Optimized Hardware
```

This module helped me understand how RTL coding decisions directly influence the final synthesized hardware structure.
