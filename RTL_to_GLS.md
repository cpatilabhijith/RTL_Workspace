# VSDBabySoC — RTL to Gate-Level Verification

## Overview

VSDBabySoC is a compact System-on-Chip that integrates a RISC-V processor core with clock-generation and digital-to-analog interface blocks. This project demonstrates the movement of a digital design from its RTL description through synthesis and finally to gate-level functional verification.

The work covered in this project includes RTL simulation, synthesis using Yosys, technology mapping with the Sky130 standard-cell library, generation of a synthesized netlist, inspection of the netlist, and post-synthesis simulation.

<img width="601" height="332" alt="image" src="https://github.com/user-attachments/assets/470c9651-d1d3-4994-904d-30423a529dec" />


---

## Contents

* [1. VSDBabySoC Architecture](#1-vsdbabysoc-architecture)
* [2. Project Setup](#2-project-setup)
* [3. RTL and Testbench](#3-rtl-and-testbench)
* [4. RTL Simulation](#4-rtl-simulation)
* [5. Synthesis with Yosys](#5-synthesis-with-yosys)
* [6. Sky130 Technology Libraries](#6-sky130-technology-libraries)
* [7. Netlist Generation](#7-netlist-generation)
* [8. Netlist Inspection](#8-netlist-inspection)
* [9. Gate-Level Simulation](#9-gate-level-simulation)
* [10. RTL vs GLS Comparison](#10-rtl-vs-gls-comparison)
* [11. Complete Flow](#11-complete-flow)
* [12. Conclusion](#12-conclusion)

---

# 1. VSDBabySoC Architecture

VSDBabySoC is organized around a top-level module named `vsdbabysoc`. The top module connects the main processing, clocking, and output blocks to form a single SoC.

The major blocks are:

```text
                    VSDBabySoC
                        |
          +-------------+-------------+
          |             |             |
        RVMYTH         PLL           DAC
          |             |             |
     RISC-V Core     Clocking      Analog Output
```

### RVMYTH

`RVMYTH` is the RISC-V processor core responsible for the main digital processing operations.

### PLL

PLL stands for Phase Locked Loop. It provides the clock-related functionality used by the SoC.

### DAC

DAC stands for Digital-to-Analog Converter. It receives the digital data from the processor path and produces the corresponding output.

### Top-Level Module

The `vsdbabysoc` module acts as the top-level module and establishes the connections between the RVMYTH, PLL, and DAC blocks.

---

# 2. Project Setup

A separate working directory was created for the BabySoC project, and the simulation repository was cloned into it.

```bash
mkdir baby_soc
cd baby_soc/
git clone https://github.com/Subhasis-Sahu/BabySoC_Simulation
cd BabySoC_Simulation
```

The project directory contains the RTL source files, simulation files, libraries, and other resources required for the synthesis and verification flow.

<img width="1920" height="1080" alt="Screenshot 2026-08-22 143313" src="https://github.com/user-attachments/assets/d93af42e-2649-4898-8965-82266fec88bd" />


---

# 3. RTL and Testbench

The RTL implementation contains the different modules that make up the VSDBabySoC. The `vsdbabysoc` module connects the RVMYTH processor, PLL, and DAC.

A testbench is used to provide the required clock and reset signals and to observe the output behavior of the design.

The testbench is also configured to support both pre-synthesis and post-synthesis simulation.

<img width="1536" height="787" alt="image" src="https://github.com/user-attachments/assets/8c49c456-7a3c-4cdf-ab0c-14c6603c31d1" />

---

# 4. RTL Simulation

Before synthesis, the original RTL design is simulated to verify its functional behavior.

The simulation flow is:

```text
RTL Files
    +
Testbench
    |
    v
Compilation
    |
    v
RTL Simulation
    |
    v
VCD Waveform
    |
    v
GTKWave
```

The RTL simulation is compiled using Icarus Verilog:

```bash
iverilog -o ./pre_synth_sim.out -DPRE_SYNTH_SIM src/module/testbench.v -I src/include/ -I src/module/
```

The generated simulation executable is then run:

```bash
./pre_synth_sim.out
```

The resulting waveform is opened using:

```bash
gtkwave pre_synth_sim.vcd
```
<img width="1920" height="412" alt="image" src="https://github.com/user-attachments/assets/46d7e0ff-fb4a-420e-bd71-32a83eb48510" />


The waveform can be used to inspect signals such as:

* `CLK`
* `reset`
* `OUT`
* `RV_TO_DAC[9:0]`
<img width="1920" height="553" alt="Screenshot 2026-08-26 181002" src="https://github.com/user-attachments/assets/9eb8e89a-7bae-42f6-85b9-f27403c0d189" />


The purpose of this stage is to establish the expected behavior of the original RTL before synthesis.

---

# 5. Synthesis with Yosys

After verifying the RTL, the design is synthesized using Yosys.

Synthesis converts the RTL description into a lower-level logic representation and prepares it for mapping to cells from the target technology library.

The overall synthesis process is:

```text
RTL
 |
 v
Yosys
 |
 v
Logic Synthesis
 |
 v
Optimization
 |
 v
Technology Mapping
 |
 v
Gate-Level Netlist
```

## Starting Yosys

```tcl
yosys
```

## Reading the RTL

The top-level module is first loaded:

```tcl
read_verilog src/module/vsdbabysoc.v
```

The RVMYTH module is then read:

```tcl
read_verilog -I src/include/ src/module/rvmyth.v
```

The clock-gating module is also included:

```tcl
read_verilog -I src/include/ src/module/clk_gate.v
```

## Reading the Libraries

The PLL Liberty file is loaded:

```tcl
read_liberty -lib src/lib/avsdpll.lib
```

The DAC Liberty file is loaded:

```tcl
read_liberty -lib src/lib/avsddac.lib
```

The Sky130 standard-cell library is loaded using:

```tcl
read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

## Selecting the Top Module

The synthesis process is started with:

```tcl
synth -top vsdbabysoc
```

This specifies `vsdbabysoc` as the top-level design from which synthesis should proceed.

---

# 6. Sky130 Technology Libraries

The target technology used for logic mapping is the Sky130 standard-cell library.

The main library contains cells such as:

* Logic gates
* Inverters
* Buffers
* Flip-flops

The PLL and DAC are represented using their corresponding Liberty files.

The technology mapping concept is:

```text
RTL Design
     |
     v
   Yosys
     |
     v
Sky130 Libraries
     |
     v
Technology Mapping
     |
     v
Mapped Gate-Level Design
```

## Flip-Flop Mapping

Sequential elements are mapped using:

```tcl
dfflibmap -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

This maps the generic flip-flops produced during synthesis to suitable Sky130 flip-flop cells.

```text
RTL Flip-Flop
      |
      v
  dfflibmap
      |
      v
Sky130 Flip-Flop
```

## Combinational Mapping

The combinational logic is mapped using ABC:

```tcl
abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

ABC maps the logic into cells available in the selected Sky130 library.

---

# 7. Optimization and Netlist Generation

After technology mapping, additional cleanup operations are performed.

Undefined logic values are converted to zero using:

```tcl
setundef -zero
```

Unused elements and unnecessary connections are removed with:

```tcl
clean -purge
```

The cleanup operation can be applied again when required:

```tcl
clean -purge
```

Generated internal objects are organized using:

```tcl
rename -enumerate
```

Finally, synthesis statistics are displayed using:

```tcl
stat
```
<img width="1920" height="1080" alt="Screenshot 2026-08-23 134727" src="https://github.com/user-attachments/assets/5226e96a-74ad-411e-ae44-72d75817b0ff" />


The synthesized design is written to a Verilog netlist using:

```tcl
write_verilog -noattr baby_soc_net3.v
```

The resulting `baby_soc_net3.v` represents the synthesized gate-level implementation.

---

# 8. Netlist Inspection

The synthesized netlist can be viewed at different levels to understand how the RTL has been transformed.

## Top-Level Netlist

At the block level, the major connections between the PLL, RVMYTH, and DAC can be identified.

```text
PLL
 |
 | Clock
 v
RVMYTH
 |
 | RV_TO_DAC
 v
DAC
 |
 v
OUT
```
<img width="1920" height="1080" alt="Screenshot 2026-08-23 140945" src="https://github.com/user-attachments/assets/3b20f815-86bc-47b0-815f-8e8d32844251" />


<img width="1757" height="311" alt="image" src="https://github.com/user-attachments/assets/25eb38d8-4bf5-4542-b1e5-edeeed5cefc9" />


This representation provides a simple view of the overall connectivity.

## Gate-Level Netlist

After ABC technology mapping, the internal logic is represented using a large number of Sky130 standard cells and interconnections.

<img width="1037" height="557" alt="image" src="https://github.com/user-attachments/assets/f39f298e-e8b8-47e0-8f66-51a2afc6229c" />



The mapped netlist appears much denser than the block-level representation because the internal logic has been expanded into individual technology cells.

---

# 9. Gate-Level Simulation

Once the synthesized netlist has been generated, it is simulated again.

This stage is known as Gate-Level Simulation (GLS) or post-synthesis simulation.

Unlike RTL simulation, the simulator now uses the synthesized netlist together with the Verilog models of the Sky130 standard cells.

The process is:

```text
Testbench
    +
Gate-Level Netlist
    +
Sky130 Cell Models
    |
    v
Icarus Verilog
    |
    v
Gate-Level Simulation
    |
    v
post_synth_sim.vcd
```

The simulation is compiled using:

```bash
iverilog -DPOST_SYNTH_SIM -DFUNCTIONAL \
-I src/include/ \
-I ../../sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/ \
-I src/module/ \
src/module/testbench.v
```

The generated simulation executable is executed using:

```bash
./a.out
```
<img width="1920" height="1080" alt="Screenshot 2026-08-26 180113" src="https://github.com/user-attachments/assets/4c438502-6cf1-470c-9a31-8ce2d344473a" />


The resulting waveform is opened in GTKWave:

```bash
gtkwave post_synth_sim.vcd
```

<img width="1920" height="482" alt="image" src="https://github.com/user-attachments/assets/bc67216a-b2f0-4583-a4f0-124bae124884" />


---

# 10. Understanding `POST_SYNTH_SIM` and `FUNCTIONAL`

## `POST_SYNTH_SIM`

The following option is used during gate-level compilation:

```bash
-DPOST_SYNTH_SIM
```

This defines the `POST_SYNTH_SIM` macro.

The Verilog environment can use this macro to select the post-synthesis simulation path.

For example:

```verilog
`ifdef POST_SYNTH_SIM

    // Post-synthesis simulation

`else

    // RTL simulation

`endif
```

This makes it possible to use different simulation configurations for RTL and synthesized designs.

## `FUNCTIONAL`

The second definition used during GLS is:

```bash
-DFUNCTIONAL
```

The Sky130 standard cells have Verilog models that describe their behavior during simulation.

The `FUNCTIONAL` definition selects the functional representation of these cells.

The simplified process is:

```text
Synthesized Netlist
       |
       v
Sky130 Cell Models
       |
       v
Functional Simulation
```

This allows the logical behavior of the synthesized implementation to be verified.

---
<img width="1920" height="1080" alt="Screenshot 2026-08-23 140243" src="https://github.com/user-attachments/assets/7c8bea63-bde3-4bae-bc5b-ae4c895b491b" />


# 11. RTL vs GLS Comparison

The final step is to compare the RTL simulation results with the gate-level simulation results.

The two waveform files are:

```text
pre_synth_sim.vcd
post_synth_sim.vcd
```

The main signals examined are:

* `CLK`
* `reset`
* `RV_TO_DAC[9:0]`
* `OUT`

The verification process is:

```text
        RTL Simulation
              |
              v
      pre_synth_sim.vcd
              |
              | Compare
              v
      post_synth_sim.vcd
              ^
              |
       Gate-Level Simulation
```

The corresponding signals in the RTL and GLS waveforms show the same functional behavior throughout the observed simulation interval.
<img width="1920" height="1080" alt="Screenshot 2026-08-26 181002" src="https://github.com/user-attachments/assets/aa04d9b7-1f96-43c2-851e-03d900bfc9a5" />


The `RV_TO_DAC[9:0]` bus follows the same sequence in both simulations, while the clock, reset, and output behavior remain consistent.

---

# 12. Complete Flow

The complete VSDBabySoC verification process can be summarized as:

```text
                       VSDBabySoC
                            |
                            v
                       RTL Design
                            |
                            v
                 Pre-Synthesis Simulation
                            |
                            v
                     RTL Verification
                            |
                            v
                          Yosys
                            |
              +-------------+-------------+
              |             |             |
              v             v             v
          Read RTL     Read Libraries  Select Top
              +-------------+-------------+
                            |
                            v
                        Synthesis
                            |
                            v
                       dfflibmap
                            |
                            v
                           ABC
                            |
                            v
                  Technology Mapping
                            |
                            v
                   Gate-Level Netlist
                            |
                            v
                Post-Synthesis Simulation
                            |
              +-------------+-------------+
              |             |             |
              v             v             v
       POST_SYNTH_SIM   FUNCTIONAL   Sky130 Models
              +-------------+-------------+
                            |
                            v
                  Functional Verification
```

---

# 13. Complete Yosys Script

The complete synthesis sequence used for the design is:

```tcl
read_verilog src/module/vsdbabysoc.v
read_verilog -I src/include/ src/module/rvmyth.v
read_verilog -I src/include/ src/module/clk_gate.v

read_liberty -lib src/lib/avsdpll.lib
read_liberty -lib src/lib/avsddac.lib
read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

synth -top vsdbabysoc

dfflibmap -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

opt

abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

setundef -zero
clean -purge
rename -enumerate

stat

write_verilog -noattr baby_soc_net3.v
```

---

# 14. Learning Outcomes

This project provided practical understanding of the RTL-to-gate-level portion of an ASIC design flow.

The main concepts covered were:

* Understanding the architecture of a multi-module SoC
* Identifying the top-level design module
* Working with the RVMYTH processor core
* Understanding the PLL and DAC blocks
* Performing RTL simulation
* Using Icarus Verilog
* Inspecting waveforms with GTKWave
* Managing Verilog include directories
* Loading RTL files into Yosys
* Reading Liberty technology libraries
* Performing logic synthesis
* Mapping sequential logic with `dfflibmap`
* Mapping combinational logic with ABC
* Using the Sky130 standard-cell library
* Generating a gate-level netlist
* Inspecting synthesized logic
* Performing post-synthesis functional simulation
* Understanding `POST_SYNTH_SIM`
* Understanding the `FUNCTIONAL` simulation option
* Comparing RTL and gate-level waveforms
* Checking that synthesis preserves the intended functionality

---

# 15. Conclusion

The VSDBabySoC design was successfully processed from its RTL description through pre-synthesis simulation, Yosys synthesis, Sky130 technology mapping, gate-level netlist generation, and post-synthesis functional simulation.

The comparison between the RTL and GLS waveforms shows consistent behavior for the important signals, including `RV_TO_DAC[9:0]`, `CLK`, `reset`, and `OUT`. This indicates that the synthesized gate-level implementation preserves the functional behavior observed at the RTL level.

Overall, the experiment demonstrates the practical transition from RTL design to a technology-mapped gate-level implementation and its subsequent functional verification.
