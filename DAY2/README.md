# Day 2: Sequential Logic, Timing Libraries and Synthesis

## Overview

Day 2 focuses on **sequential RTL design**, flip-flop coding, PVT conditions, standard-cell libraries, and basic synthesis using **Yosys**. The session also covers simulation with **Icarus Verilog** and waveform verification using **GTKWave**.

---

## Contents

1. Flip-Flops and Reset Techniques
2. PVT and SKY130 Library
3. Liberty `.lib` File
4. RTL Simulation
5. Yosys Synthesis
6. Hierarchical vs Flattened Design
7. Learning Outcomes

---

# 1. Flip-Flops and Reset Techniques

A **flip-flop** is a sequential logic element capable of storing one bit of information. Its output changes according to the clock and control signals.

### Synchronous Reset

A synchronous reset is checked only at the active clock edge.

```verilog
always @(posedge clk)
begin
    if (reset)
        q <= 1'b0;
    else
        q <= d;
end
```

### Asynchronous Reset

An asynchronous reset can change the output independently of the clock.

```verilog
always @(posedge clk or posedge reset)
begin
    if (reset)
        q <= 1'b0;
    else
        q <= d;
end
```

### Asynchronous Set

An asynchronous set forces the output to logic `1`.

```verilog
always @(posedge clk or posedge set)
begin
    if (set)
        q <= 1'b1;
    else
        q <= d;
end
```

### Comparison

| Feature | Synchronous Reset | Asynchronous Reset | Asynchronous Set |
|:--|:--|:--|:--|
| Active state | `0` | `0` | `1` |
| Clock dependent | Yes | No | No |
| Output response | At clock edge | Immediate | Immediate |

---

# 2. PVT and SKY130 Library

**PVT** stands for **Process, Voltage and Temperature**. These conditions influence the timing and performance of semiconductor circuits.

Example library:

```text
sky130_fd_sc_hd__tt_025C_1v80.lib
```

Here:

- `tt` → Typical process
- `025C` → 25°C
- `1v80` → 1.8V supply

The SKY130 library provides standard-cell information required during synthesis and technology mapping.

---

# 3. Liberty `.lib` File

A **Liberty file** contains characterization information for standard cells.

It includes details such as:

- Cell functionality
- Pin information
- Timing
- Delay
- Power
- Area

The library can be opened using:

```bash
gedit sky130_fd_sc_hd__tt_025C_1v80.lib
```

---

# 4. RTL Simulation

The sequential designs were verified using **Icarus Verilog** and **GTKWave**.

### Simulation Flow

```text
Verilog Design
      +
Testbench
      ↓
Icarus Verilog
      ↓
Simulation
      ↓
VCD File
      ↓
GTKWave
```

### Commands

```bash
iverilog dff_asyncres.v tb_dff_asyncres.v
```

```bash
./a.out
```

```bash
gtkwave tb_dff_asyncres.vcd
```

The waveform is used to observe the clock, reset, input, and output signals.

---

# 5. Yosys Synthesis

**Yosys** is an open-source RTL synthesis tool. It converts Verilog RTL into a synthesized hardware representation.

### Basic Commands

```text
yosys
read_verilog dff_asyncres.v
synth -top dff_asyncres
read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib
dfflibmap -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

The synthesis process can be summarized as:

```text
RTL
 ↓
Synthesis
 ↓
Optimization
 ↓
Technology Mapping
 ↓
Gate-Level Netlist
```

---

# 6. Hierarchical vs Flattened Design

### Hierarchical Design

The original module structure is preserved.

```text
Top
├── Module A
├── Module B
└── Module C
```

**Advantages:**
- Easier debugging
- Maintains module organization
- Suitable for modular designs

### Flattened Design

The module hierarchy is removed and the logic is combined.

```text
Top + Submodules
       ↓
Combined Logic
```

Yosys can flatten a design using:

```text
flatten
```

### Comparison

| Hierarchical | Flattened |
|:--|:--|
| Preserves modules | Removes hierarchy |
| Easier debugging | More difficult to trace |
| Modular structure | Combined logic |
| Limited cross-module optimization | Better global optimization |

---

# 7. Learning Outcomes

Through Day 2, I learned:

- Fundamentals of sequential logic and flip-flops.
- Synchronous and asynchronous reset techniques.
- Asynchronous set operation.
- Basics of PVT conditions.
- SKY130 standard-cell library naming.
- Purpose of Liberty `.lib` files.
- RTL simulation using Icarus Verilog.
- Waveform analysis using GTKWave.
- Basic synthesis commands in Yosys.
- Difference between hierarchical and flattened designs.

---

## Conclusion

Day 2 provided practical exposure to the transition from **sequential RTL coding and simulation to synthesis and technology mapping**. It helped strengthen my understanding of flip-flop behavior, timing libraries, and the basic RTL-to-netlist flow.
