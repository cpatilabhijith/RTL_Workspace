# Day 2: Sequential RTL Design, Libraries and Synthesis

## Overview

Day 2 extends the RTL design concepts from combinational logic to **sequential circuits and synthesis**. The session focuses on understanding flip-flop behavior, different reset mechanisms, PVT conditions, standard-cell timing libraries, and the role of Yosys in converting RTL into a technology-mapped design.

The practical work also connects **Icarus Verilog simulation** with **GTKWave waveform analysis** and **Yosys synthesis**, providing an overview of the RTL-to-netlist flow.

---

## Contents

1. Sequential Logic and Flip-Flops
2. Reset and Set Techniques
3. PVT Conditions and Standard-Cell Libraries
4. Understanding Liberty Files
5. RTL Simulation
6. Synthesis using Yosys
7. Hierarchical and Flattened Designs
8. Flip-Flop RTL Implementations
9. Complete Design Flow
10. Learning Outcomes

---

# 1. Sequential Logic and Flip-Flops

Unlike combinational circuits, sequential circuits depend on both the current inputs and previously stored information. **Flip-flops** are basic memory elements used to retain a single binary value.

They form the foundation of:

- Registers
- Counters
- State machines
- Data storage elements
- Pipeline stages

A clock signal is generally used to control when the stored value is updated.

### Basic Concept

```text
        D
        │
        ▼
   ┌─────────┐
CLK│   D FF  │───► Q
──►│         │
   └─────────┘
```

---

# 2. Reset and Set Techniques

Reset and set signals are used to place a flip-flop into a known state.

## Synchronous Reset

A synchronous reset is evaluated only at the active clock edge.

```verilog
always @(posedge clk)
begin
    if (reset)
        q <= 1'b0;
    else
        q <= d;
end
```

The reset does not change the output immediately. The flip-flop responds to the reset when the clock edge occurs.

---

## Asynchronous Reset

An asynchronous reset operates independently of the normal clocked data transfer.

```verilog
always @(posedge clk or posedge reset)
begin
    if (reset)
        q <= 1'b0;
    else
        q <= d;
end
```

When `reset` becomes active, the output is driven low without waiting for the next clock edge.

---

## Asynchronous Set

An asynchronous set forces the flip-flop output to logic `1`.

```verilog
always @(posedge clk or posedge set)
begin
    if (set)
        q <= 1'b1;
    else
        q <= d;
end
```

The set signal has priority over normal data capture.

---

## Reset Comparison

| Property | Synchronous Reset | Asynchronous Reset | Asynchronous Set |
|:--|:--|:--|:--|
| Active state | `0` | `0` | `1` |
| Depends on clock | Yes | No | No |
| Output action | At clock edge | Immediately | Immediately |
| Main purpose | Controlled reset | Immediate reset | Immediate preset |

---

# 3. PVT Conditions and Standard-Cell Libraries

## What is PVT?

PVT represents the three major conditions that can influence the behavior of semiconductor circuits:

- **Process** – Manufacturing variations in the fabrication process.
- **Voltage** – Changes in the supply voltage.
- **Temperature** – Changes in the operating temperature.

These conditions can affect parameters such as timing, delay, power consumption, and overall circuit performance.

---

## SKY130 Example

One of the standard-cell libraries used during the workshop is:

```text
sky130_fd_sc_hd__tt_025C_1v80.lib
```

The suffix provides information about the conditions represented by the library:

```text
tt_025C_1v80
│  │     │
│  │     └── 1.8 V supply
│  └──────── 25°C
└─────────── Typical process
```

Therefore:

- `tt` → Typical process corner
- `025C` → 25°C
- `1v80` → 1.8 V

---

# 4. Understanding Liberty Files

A **Liberty file (`.lib`)** describes the characteristics of standard cells available in a technology library.

It can contain information related to:

- Cell functionality
- Input and output pins
- Timing characteristics
- Propagation delay
- Power information
- Area
- Timing arcs

During synthesis, this information helps the synthesis tool select appropriate standard cells for implementing the RTL logic.

## Opening the Library

The library can be opened using a text editor such as `gedit`.

```bash
sudo apt install gedit
```

```bash
gedit sky130_fd_sc_hd__tt_025C_1v80.lib
```

### Library File

The `.lib` file can be explored to understand how standard cells are characterized for a particular process, voltage, and temperature condition.

---

# 5. RTL Simulation

Before synthesizing a design, its functionality should be verified using simulation.

The RTL simulation process uses three main components:

```text
RTL Module
     +
Testbench
     │
     ▼
Icarus Verilog
     │
     ▼
Simulation
     │
     ▼
VCD File
     │
     ▼
GTKWave
```

This allows the expected behavior of the design to be checked before moving toward synthesis.

---

## Icarus Verilog

**Icarus Verilog** is an open-source Verilog compiler and simulator used to execute RTL designs.

### Compile

```bash
iverilog dff_asyncres.v tb_dff_asyncres.v
```

### Execute

```bash
./a.out
```

### Open Waveform

```bash
gtkwave tb_dff_asyncres.vcd
```

The resulting waveform can be used to observe the relationship between the clock, reset, input, and output signals.

---

# 6. Synthesis Using Yosys

Simulation confirms functional behavior, while **synthesis** converts the RTL description into a hardware-oriented representation.

**Yosys** is an open-source synthesis framework that can read Verilog, optimize the logic, and generate a synthesized netlist.

### Basic Synthesis Flow

```text
Verilog RTL
     ↓
Read Design
     ↓
RTL Synthesis
     ↓
Logic Optimization
     ↓
Technology Mapping
     ↓
Gate-Level Netlist
```

---

## Starting Yosys

```bash
yosys
```

## Loading the RTL

```text
read_verilog dff_asyncres.v
```

## Selecting the Top Module

```text
hierarchy -top dff_asyncres
```

## Performing Synthesis

```text
synth -top dff_asyncres
```

---

# 7. Technology Mapping

After synthesis, the generated logic can be mapped to cells available in the selected standard-cell library.

### Load the Liberty Library

```text
read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib
```

### Flip-Flop Mapping

```text
dfflibmap -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
```

### Logic Mapping

```text
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
```

The mapping process replaces generic synthesized elements with actual cells from the selected technology library.

---

# 8. Flip-Flop RTL Implementations

## Asynchronous Reset Flip-Flop

```verilog
module dff_asyncres (
    input clk,
    input async_reset,
    input d,
    output reg q
);

always @(posedge clk or posedge async_reset)
begin
    if (async_reset)
        q <= 1'b0;
    else
        q <= d;
end

endmodule
```

### Behavior

The output is immediately cleared when `async_reset` becomes active. Otherwise, the input `d` is captured on the rising edge of the clock.

---

## Asynchronous Set Flip-Flop

```verilog
module dff_async_set (
    input clk,
    input async_set,
    input d,
    output reg q
);

always @(posedge clk or posedge async_set)
begin
    if (async_set)
        q <= 1'b1;
    else
        q <= d;
end

endmodule
```

### Behavior

When `async_set` is asserted, the output becomes `1` immediately. When it is inactive, the input is captured on the positive clock edge.

---

## Synchronous Reset Flip-Flop

```verilog
module dff_syncres (
    input clk,
    input sync_reset,
    input d,
    output reg q
);

always @(posedge clk)
begin
    if (sync_reset)
        q <= 1'b0;
    else
        q <= d;
end

endmodule
```

### Behavior

The reset is checked only during the rising edge of the clock. If `sync_reset` is active at that instant, the output is cleared.

---

# 9. Hierarchical and Flattened Designs

## Hierarchical Synthesis

A hierarchical design preserves the relationships between the top-level module and its submodules.

```text
Top Module
    │
    ├── Module A
    │
    ├── Module B
    │
    └── Module C
```

### Benefits

- Preserves the original module organization.
- Makes large designs easier to understand.
- Helps with debugging and design analysis.
- Supports modular development.

---

## Flattened Synthesis

Flattening removes the module boundaries and combines the logic into a single representation.

```text
Before Flattening

       TOP
      /   \
   MOD_A  MOD_B

          ↓

After Flattening

    Combined Logic
```

Yosys can flatten a design using:

```text
flatten
```

A flattened design can allow optimization across module boundaries, although tracing the original hierarchy can become more difficult.

---

## Comparison

| Feature | Hierarchical | Flattened |
|:--|:--|:--|
| Module boundaries | Preserved | Removed |
| Design organization | Modular | Combined |
| Debugging | Easier | More difficult |
| Cross-module optimization | Limited | Greater |
| Large design handling | More manageable | Can become complex |
| Main advantage | Structure | Global optimization |

---

# 10. Complete RTL-to-Netlist Flow

The concepts covered during Day 2 can be connected into the following workflow:

```text
        RTL Design
            │
            ▼
        Testbench
            │
            ▼
     Icarus Verilog
            │
            ▼
       RTL Simulation
            │
            ▼
          VCD
            │
            ▼
        GTKWave
            │
            ▼
    Functional Verification
            │
            ▼
          Yosys
            │
            ▼
      RTL Synthesis
            │
            ▼
    Liberty / SKY130 Library
            │
            ▼
   Technology Mapping
            │
            ▼
     Gate-Level Netlist
```

---

# Learning Outcomes

By the end of Day 2, I was able to:

- Understand the role of flip-flops in sequential circuits.
- Differentiate synchronous and asynchronous controls.
- Implement D flip-flops with reset and set functionality.
- Understand PVT variations in semiconductor technology.
- Interpret the naming convention of a SKY130 timing library.
- Explore the contents and purpose of a Liberty `.lib` file.
- Simulate sequential RTL designs using Icarus Verilog.
- Inspect timing behavior using GTKWave.
- Understand the purpose of RTL synthesis.
- Use basic Yosys synthesis commands.
- Understand the role of standard-cell libraries during technology mapping.
- Compare hierarchical and flattened synthesis approaches.

---

# Key Takeaways

```text
Sequential Logic
       ↓
Flip-Flop Coding
       ↓
RTL Simulation
       ↓
Waveform Verification
       ↓
PVT / Standard-Cell Library
       ↓
Yosys Synthesis
       ↓
Technology Mapping
       ↓
Gate-Level Netlist
```

Day 2 helped me understand how a simple RTL description progresses from **behavioral verification to a technology-dependent hardware representation**. The combination of Verilog, Icarus Verilog, GTKWave, Yosys, and the SKY130 library provided practical exposure to an important part of the digital IC design flow.
