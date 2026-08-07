# Day 1: Introduction to RTL Design Using Verilog

## Overview

This experiment introduces the fundamentals of **Register Transfer Level (RTL) Design** using **Verilog HDL**. The primary objective is to understand how digital circuits are described, verified through simulation, and analyzed before hardware implementation. A **2:1 Multiplexer** is implemented as the first RTL design to demonstrate the complete simulation and verification workflow.

---

## Table of Contents

1. RTL Verification Fundamentals
2. Verilog Simulation Environment
3. Practical Exercise – 2:1 Multiplexer
4. RTL Code Explanation
5. Learning Outcome

---

# 1. RTL Verification Fundamentals

## Simulator

A simulator is a software application that executes Verilog code and predicts the behavior of a digital circuit under different input conditions. It helps verify the correctness of the design before it is synthesized into hardware.

## RTL Design

An RTL design is a Verilog description of digital hardware. It specifies how data flows between different components and how the output responds to various input combinations.

## Testbench

A testbench is a separate verification module that applies different input patterns to the design and checks whether the generated outputs match the expected functionality.

---

# 2. Verilog Simulation Environment

## What is Icarus Verilog?

**Icarus Verilog (iverilog)** is an open-source Verilog compiler and simulator. It compiles the RTL design and testbench, executes the simulation, and generates a **Value Change Dump (.vcd)** file for waveform analysis using **GTKWave**.

### Simulation Workflow

```text
RTL Design
      +
Testbench
      ↓
Icarus Verilog
      ↓
Simulation
      ↓
Generate VCD File
      ↓
GTKWave
```

---

# 3. Practical Exercise – 2:1 Multiplexer

## Step 1: Install Required Tools

```bash
sudo apt install iverilog
sudo apt install gtkwave
```

## Step 2: Compile the Design

```bash
iverilog good_mux.v tb_good_mux.v
```

This command compiles the Verilog design together with its testbench.

## Step 3: Execute the Simulation

```bash
./a.out
```

Running the compiled executable performs the simulation and generates the waveform file.

## Step 4: Open the Waveform

```bash
gtkwave tb_good_mux.vcd
```

The generated waveform can now be viewed and analyzed using GTKWave.

### Simulation Result

The waveform confirms that the multiplexer selects the correct input based on the value of the selection signal.

---

# 4. RTL Code Explanation
## Verilog Design

```verilog
module good_mux (
    input i0,
    input i1,
    input sel,
    output reg y
);

always @(*)
begin
    if (sel)
        y <= i1;
    else
        y <= i0;
end

endmodule
```

---
## Multiplexer Description

The implemented circuit is a **2:1 Multiplexer** consisting of two data inputs, one selection input, and one output.

### Input Signals

- `i0` – First input
- `i1` – Second input
- `sel` – Selection signal

### Output Signal

- `y` – Multiplexer output

### Working Principle

- When `sel = 0`, the output follows **i0**.
- When `sel = 1`, the output follows **i1**.
- The select signal determines which input is routed to the output.

---

# 5. Learning Outcome

Through this experiment, I learned the basic RTL design methodology using Verilog HDL. I understood the purpose of simulation, the role of a testbench, and the process of compiling and verifying a digital circuit using Icarus Verilog and GTKWave. This exercise provides a strong foundation for implementing more complex RTL designs in the upcoming sessions.
