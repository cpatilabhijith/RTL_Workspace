# Day 9 — Pre-Layout Timing Analysis and the Importance of a Good Clock Tree

**Static Timing Analysis (STA)** verification, before and after **Clock Tree Synthesis (CTS)**, using an **OpenLANE + SKY130 + OpenSTA/OpenROAD** flow.

This module documents timing-model characterization, setup/hold constraints, clock uncertainty, timing-path classification, clock-tree fundamentals, CTS execution, and post-CTS timing verification.

| | |
|---|---|
| **Technology** | SKY130 |
| **Toolchain** | OpenLANE · OpenROAD · OpenSTA |
| **Analysis Type** | Static Timing Analysis (STA) |
| **Scope** | Setup, Hold, CTS, Clock Skew, Clock Latency |

---

## Table of Contents

1. [Timing Analysis Overview](#1-timing-analysis-overview)
2. [Timing Modelling and Characterization](#2-timing-modelling-and-characterization)
3. [Setup Timing Analysis with an Ideal Clock](#3-setup-timing-analysis-with-an-ideal-clock)
4. [Timing Constraints](#4-timing-constraints)
5. [Timing Paths in a Design](#5-timing-paths-in-a-design)
6. [Hold Timing Analysis](#6-hold-timing-analysis)
7. [Why the Clock Tree Matters](#7-why-the-clock-tree-matters)
8. [Clock Tree Synthesis (CTS)](#8-clock-tree-synthesis-cts)
9. [Running CTS in OpenLANE](#9-running-cts-in-openlane)
10. [Post-CTS Timing Analysis](#10-post-cts-timing-analysis)
11. [CTS Quality Metrics](#11-cts-quality-metrics)
12. [Pre-CTS vs Post-CTS](#12-pre-cts-vs-post-cts)
13. [Suggested Comparison Table](#13-suggested-comparison-table)
14. [Key Concepts Summary](#14-key-concepts-summary)
15. [Overall Physical Design Timing Flow](#15-overall-physical-design-timing-flow)
16. [Learning Outcome](#16-learning-outcome)

---

# 1. Timing Analysis Overview

Static Timing Analysis (STA) verifies whether data signals can propagate through a digital design within the required timing limits.

A typical register-to-register path is:

```text
Launch FF
   │
   Q
   │
   ▼
Combinational Logic
   │
   ▼
Capture FF
   │
   D
```

For a single-clock setup path:

```text
FF1 → Combinational Logic → FF2
```

The launch flip-flop sends data after the active clock edge. The data then passes through the combinational logic and must reach the capture flip-flop early enough before the next active clock edge.

---

# 2. Timing Modelling and Characterization

## 2.1 Liberty Files

Standard-cell timing information is normally stored in `.lib` files.

A Liberty file contains information such as:

- Cell functionality
- Input capacitance
- Output capacitance
- Timing arcs
- Propagation delay
- Output transition/slew
- Setup time
- Hold time
- Power information
- Operating conditions

OpenSTA uses this information to calculate timing for the gates in the design.

---

## 2.2 NLDM — Non-Linear Delay Model

In the traditional NLDM model, cell delay is represented using lookup tables.

Two important variables are:

### 1. Input Transition / Slew

The input transition describes how quickly the input signal changes.

A slower input transition can increase the delay of a cell.

### 2. Output Load Capacitance

The output load is the capacitance driven by the cell.

It can include:

- Input capacitance of the next cell
- Interconnect capacitance
- Other fanout capacitances

Conceptually:

```text
Input Slew  ───────────────►
             ┌───────────┐
             │ Delay LUT │
             └───────────┘
Output Load ──────────────►

                 │
                 ▼
             Cell Delay
```

If the exact slew/load combination is not present in the table, the STA tool interpolates between available table points.

---

# 3. Setup Timing Analysis with an Ideal Clock

Before CTS, the physical clock network has not yet been built.

Therefore, the clock is generally treated as an **ideal clock** for pre-CTS timing analysis.

A simplified path is:

```text
             Combinational Logic
          ┌────────────────────────┐
          │                        │
FF1/Q ────┴──────────────────────► FF2/D
 │                                  │
 │                                  │
 └────────────── CLK ───────────────┘
```

### Example Specifications

```text
Clock Frequency = 1 GHz
Clock Period    = 1 / 1 GHz
                = 1 ns

Setup Time      = 10 ps
                = 0.01 ns

Clock Uncertainty = 90 ps
                  = 0.09 ns
```

---

## 3.1 Setup Timing Equation

For an ideal-clock single-cycle path, a simplified setup constraint is:

\[
\theta \leq T - T_{setup} - T_{uncertainty}
\]

For the example:

\[
\theta \leq 1 - 0.01 - 0.09
\]

\[
\boxed{\theta \leq 0.90\ ns}
\]

Therefore, the maximum allowed data-path delay is:

```text
Maximum combinational/path delay = 0.90 ns
```

If the actual path delay is greater than 0.90 ns, a setup violation occurs.

---

## 3.2 Setup Slack

The general concept is:

\[
Slack_{setup}=Required\ Time-Arrival\ Time
\]

For the simplified ideal-clock example:

\[
Slack_{setup}
=
(T-T_{setup}-T_{uncertainty})-\theta
\]

Example:

```text
T              = 1.00 ns
Tsetup         = 0.01 ns
Tuncertainty   = 0.09 ns
Data delay θ   = 0.80 ns
```

Therefore:

\[
Slack=(1-0.01-0.09)-0.80
\]

\[
Slack=0.90-0.80
\]

\[
\boxed{Slack=+0.10\ ns}
\]

Positive setup slack means the simplified timing requirement is met.

---

# 4. Timing Constraints

Timing constraints tell the STA tool how the design is expected to operate.

## 4.1 Example SDC

```tcl
# Clock definition
create_clock [get_ports clk] \
    -name core_clock \
    -period 10.0000

# Clock uncertainty
set_clock_uncertainty 0.2000 [get_clocks core_clock]

# Clock transition
set_clock_transition 0.1000 [get_clocks core_clock]

# Input delay
set_input_delay -max 2.0000 \
    -clock core_clock [all_inputs]

# Output delay
set_output_delay -max 2.0000 \
    -clock core_clock [all_outputs]
```

> Adjust the clock name, period, input/output delays, and uncertainty to match the actual design and SDC used in your project.

---

## 4.2 OpenSTA Configuration

A basic OpenSTA Tcl flow can contain:

```tcl
set_cmd_units \
    -time ns \
    -capacitance pF \
    -current mA \
    -voltage V \
    -resistance kOhm \
    -distance um

read_liberty -min <min_liberty_file>
read_liberty -max <max_liberty_file>

read_verilog <netlist.v>

link_design <top_module>

read_sdc <constraints.sdc>

report_checks \
    -path_delay min_max \
    -fields {slew cap input nets fanout} \
    -digits 4
```

The exact Liberty files and command options depend on the installed OpenSTA/OpenLANE version.

---

# 5. Timing Paths in a Design

Timing paths describe how signals move between timing endpoints.

## 5.1 Register-to-Register Path

```text
FF1
 │
 ▼
Block 1
 │
 ▼
Block 2
 │
 ▼
FF2
```

This is the most common synchronous timing path.

It:

- Starts at the Q output of a launch flip-flop.
- Passes through combinational logic.
- Ends at the D input of a capture flip-flop.

---

## 5.2 Input-to-Output Path

```text
Primary Input
     │
     ▼
Combinational Logic
     │
     ▼
Primary Output
```

There are no registers between the input and output endpoints.

---

## 5.3 Other Common Path Categories

STA can also analyze:

- Input-to-register paths
- Register-to-output paths
- Register-to-register paths
- Clock paths
- Min-delay paths
- Max-delay paths

---

# 6. Hold Timing Analysis

Setup checks whether data arrives **early enough** for the next capture edge.

Hold checks whether data remains stable **long enough after** the active capture edge.

A simplified hold requirement is:

\[
T_{clk\rightarrow q(min)}
+
T_{data(min)}
\geq
T_{hold}
+
T_{uncertainty}
\]

Clock skew must also be considered when the clock is propagated.

The hold slack concept is:

\[
Slack_{hold}
=
Arrival\ Time
-
Required\ Time
\]

A negative hold slack indicates that data can arrive too early and a hold violation exists.

---

## Setup vs Hold

| Parameter | Setup | Hold |
|---|---|---|
| Main concern | Data arrives too late | Data arrives too early |
| Data path | Maximum delay | Minimum delay |
| Typical analysis | Max path | Min path |
| Violation | Negative setup slack | Negative hold slack |
| Common fix | Reduce delay / improve timing | Add delay to data path |

---

# 7. Why the Clock Tree Matters

The clock must reach a large number of sequential elements.

Before CTS:

```text
                 Ideal CLK
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
         FF1       FF2       FF3
```

The ideal-clock assumption does not represent the physical delay of the clock network.

After CTS:

```text
                     CLK ROOT
                        │
                      BUF
                   ┌────┴────┐
                  BUF       BUF
                ┌──┴──┐   ┌──┴──┐
               FF1   FF2 FF3   FF4
```

Each branch has physical delay.

---

## 7.1 Clock Latency

Clock latency is the time required for the clock signal to travel from its source to a clock sink.

```text
Clock Source ───────► Buffer Network ───────► FF
        <------------- clock latency ------------>
```

---

## 7.2 Clock Skew

Clock skew is the difference between clock arrival times at different sinks.

\[
T_{skew}
=
T_{arrival,max}
-
T_{arrival,min}
\]

Example:

```text
FF1 clock arrival = 1.20 ns
FF2 clock arrival = 1.25 ns

Skew = 1.25 - 1.20
     = 0.05 ns
     = 50 ps
```

Large clock skew can affect setup and hold timing.

---

# 8. Clock Tree Synthesis (CTS)

Clock Tree Synthesis builds a physical clock distribution network.

Main goals include:

- Low clock skew
- Controlled clock latency
- Acceptable transition
- Acceptable capacitance
- Sufficient clock driving strength
- Balanced distribution
- Reduced clock-related timing problems

---

## 8.1 H-Tree Concept

An H-tree uses a symmetric geometric structure to distribute the clock.

```text
                 CLK
                  │
             ─────┼─────
                  │
              ┌───┴───┐
              │       │
             ─┘       └─
            / \       / \
          FF  FF    FF  FF
```

The symmetry helps equalize clock path lengths.

---

## 8.2 Clock Buffers

Clock buffers are inserted to drive the clock network.

For SKY130, clock buffer cells can include cells from the:

```text
sky130_fd_sc_hd
```

library family, for example:

```text
sky130_fd_sc_hd__clkbuf_2
sky130_fd_sc_hd__clkbuf_4
sky130_fd_sc_hd__clkbuf_8
sky130_fd_sc_hd__clkbuf_16
```

The exact cells available depend on the installed SKY130 library version.

---

## 8.3 Crosstalk and Shielding

Clock nets switch frequently and can be sensitive to capacitive coupling.

Physical-design techniques can include:

- Clock-net spacing
- Shielding
- Careful routing
- Power/ground shielding where appropriate
- Controlled routing layers

These techniques help reduce unwanted coupling between clock and neighboring signals.

---

# 9. Running CTS in OpenLANE

The exact commands vary between OpenLANE versions. A typical interactive flow is:

```bash
cd /openlane
./flow.tcl -interactive
```

Inside the OpenLANE interactive shell:

```tcl
package require openlane 0.9

prep -design <design_name> -tag <run_tag>

run_synthesis
run_floorplan
run_placement
```

CTS is executed after placement.

---

## 9.1 Example CTS Configuration

For a flow using environment variables similar to the following:

```tcl
set ::env(CTS_ROOT_BUFFER) \
    "sky130_fd_sc_hd__clkbuf_16"

set ::env(CTS_CLK_BUFFERS_LIST) \
    "sky130_fd_sc_hd__clkbuf_2 \
     sky130_fd_sc_hd__clkbuf_4 \
     sky130_fd_sc_hd__clkbuf_8"

set ::env(CTS_MAX_CAP) 1.53e-12

set ::env(CTS_TOLERANCE) 100

run_cts
```

> These values are examples. Use the variable names and supported CTS options for the OpenLANE/OpenROAD version installed on your system.

---

# 10. Post-CTS Timing Analysis

After CTS, the clock is no longer treated as purely ideal.

The physical clock network contains:

```text
Clock Source
     │
     ▼
Clock Buffers
     │
     ├────────► FF1
     │
     ├────────► FF2
     │
     └────────► FF3
```

The clock arrival time now includes physical clock-tree delays.

---

## 10.1 Propagated Clock

In a propagated-clock analysis, the STA engine calculates clock arrival through the actual clock network.

Conceptually:

```tcl
set_propagated_clock [get_clocks core_clock]
```

The clock network is therefore included in timing calculations.

---

## 10.2 Example OpenROAD/OpenSTA Flow

A typical analysis sequence can include:

```tcl
read_lef /openlane/designs/<design_name>/runs/<run_tag>/tmp/merged.lef

read_def \
    /openlane/designs/<design_name>/runs/<run_tag>/results/cts/<design_name>.cts.def

read_liberty -min $::env(LIB_FASTEST)
read_liberty -max $::env(LIB_SLOWEST)

read_verilog \
    /openlane/designs/<design_name>/runs/<run_tag>/results/cts/<design_name>.cts.v

link_design <top_module_name>

read_sdc \
    /openlane/designs/<design_name>/src/<design_name>.sdc

set_propagated_clock [get_clocks core_clock]

report_checks \
    -path_delay min_max \
    -format full_clock_expanded \
    -digits 4

report_clock_skew -digits 4
```

The exact paths and commands should be changed according to the generated run directory.

---

# 11. CTS Quality Metrics

After CTS, several metrics should be collected.

## 11.1 Clock Skew

\[
Skew =
T_{arrival,max}
-
T_{arrival,min}
\]

A smaller skew generally means the clock arrives more uniformly at the sinks.

---

## 11.2 Clock Latency

Clock latency is the delay from the clock source to a sink.

It can be reported separately for different clock endpoints.

---

## 11.3 Clock Transition

Clock transition indicates how quickly the clock signal changes at a sink.

Excessive transition can affect timing quality and signal integrity.

---

## 11.4 Clock Capacitance

The clock network drives a large number of sequential elements.

The total capacitance must remain within the supported design/library limits.

---

## 11.5 Buffer Count

CTS inserts clock buffers to distribute the clock.

Example:

```text
Number of clock buffers = 42
```

The exact number depends on:

- Design size
- Placement
- Clock topology
- Fanout
- Buffer selection
- Timing constraints

---

## 11.6 WNS

Worst Negative Slack:

\[
WNS = \min(Slack)
\]

For setup analysis, a negative WNS indicates at least one setup timing path violates the constraint.

---

## 11.7 TNS

Total Negative Slack is the sum of negative slacks over violating paths.

Conceptually:

\[
TNS=\sum Slack_i
\quad\text{for all }Slack_i<0
\]

WNS identifies the worst violation, while TNS represents the total magnitude of negative slack across violating paths.

---

# 12. Pre-CTS vs Post-CTS

| Parameter | Pre-CTS | Post-CTS |
|---|---|---|
| Clock model | Ideal / estimated | Propagated physical clock |
| Clock buffers | Not yet inserted | Inserted |
| Clock latency | Idealized | Physical |
| Clock skew | Idealized / constrained | Measured from clock network |
| Clock transition | Constraint/estimate | Physical network result |
| Timing | Preliminary | More physically representative |
| Main purpose | Identify data-path timing problems | Verify timing with real clock network |

---

## Why Compare Them?

A useful project report should compare:

```text
                Pre-CTS          Post-CTS
                -------          --------
Setup WNS       ______           ______
Setup TNS       ______           ______
Hold WNS        ______           ______
Hold TNS        ______           ______
Clock Skew      ______           ______
Clock Latency   ______           ______
Buffer Count    ______           ______
```

The actual values should be taken directly from the OpenLANE/OpenSTA reports.

---

# 13. Suggested Comparison Table

After running the actual flow, fill the table using values from your generated reports.

| Metric | Pre-CTS | Post-CTS |
|---|---:|---:|
| Clock model | Ideal | Propagated |
| Clock latency | — | ___ ns |
| Clock skew | — / constrained | ___ ns |
| Setup WNS | ___ ns | ___ ns |
| Setup TNS | ___ ns | ___ ns |
| Hold WNS | ___ ns | ___ ns |
| Hold TNS | ___ ns | ___ ns |
| Clock buffer count | 0 | ___ |
| Max clock transition | ___ | ___ |
| Clock capacitance | ___ | ___ |

Do not manually invent the values. Extract them from the corresponding STA/CTS reports.

---

# 14. Key Concepts Summary

### Setup

```text
Data must arrive sufficiently BEFORE the capture edge.
```

### Hold

```text
Data must remain stable sufficiently AFTER the capture edge.
```

### Clock Latency

```text
Time taken by the clock to reach a sink.
```

### Clock Skew

```text
Difference between clock arrival times at sinks.
```

### CTS

```text
Builds the physical clock distribution network.
```

### Pre-CTS

```text
Ideal/estimated clock
        ↓
Preliminary STA
```

### Post-CTS

```text
Physical clock tree
        ↓
Propagated-clock STA
        ↓
More realistic timing verification
```

---

# 15. Overall Physical Design Timing Flow

```text
RTL
 │
 ▼
Synthesis
 │
 ▼
Gate-Level Netlist
 │
 ▼
Floorplanning
 │
 ▼
Placement
 │
 ▼
Pre-CTS STA
 │
 │  Ideal Clock
 │
 ▼
Clock Tree Synthesis
 │
 ▼
Post-CTS STA
 │
 │  Propagated Clock
 │
 ▼
Routing
 │
 ▼
Post-Route STA
 │
 ▼
Final Timing Verification
```

---

# 16. Learning Outcome

After completing this module, you should be able to explain:

1. How Liberty timing tables are used by STA.
2. The relationship between input slew, output load, and cell delay.
3. Setup and hold timing constraints.
4. The difference between ideal and propagated clocks.
5. Why clock latency and clock skew matter.
6. The purpose of Clock Tree Synthesis.
7. The role of clock buffers in a physical design.
8. How to run basic pre-CTS STA using OpenSTA.
9. How CTS is performed in an OpenLANE flow.
10. How to perform post-CTS timing analysis.
11. How to compare pre-CTS and post-CTS WNS, TNS, hold slack, latency, and skew.
12. How to interpret CTS quality metrics for sign-off readiness.

---

## Notes

Command syntax and environment-variable names can differ between OpenLANE/OpenROAD releases. Verify all commands against the version installed in your environment before executing them.

This document is intended as a learning and project-documentation reference. Actual timing conclusions should be based on the generated STA/CTS reports for the specific design, constraints, PDK, and operating corners used in the run.
