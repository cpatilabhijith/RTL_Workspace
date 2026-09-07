# Day 6 — Physical Design with OpenLane

## Overview

Day 6 introduces the **Physical Design stage of ASIC development**. The objective is to understand how a digital design moves from RTL description to a physical chip layout.

This project uses the **PicoRV32A RISC-V processor**, **OpenLane**, and the **SkyWater 130 nm (SKY130) PDK** to explore the beginning of the RTL-to-GDSII flow.

The major stages covered are:

```text
RTL Design
    ↓
Synthesis
    ↓
Floorplanning
    ↓
Placement
    ↓
Clock Tree Synthesis
    ↓
Routing
    ↓
Static Timing Analysis
    ↓
Physical Verification
    ↓
GDSII
```

---

## 1. Understanding Chip Structure

Before studying the physical-design flow, it is important to understand how a chip is organized physically.

A fabricated chip can be viewed in three major regions:

* **Pads** — provide connections between the chip and external signals, power, and interfaces.
* **Die** — the complete piece of silicon containing the designed circuit.
* **Core** — the central region where the main digital logic, macros, and IP blocks are placed.

Different interfaces such as GPIO, UART, SPI, I2C, QSPI, JTAG, ADC, and power/ground connections can be arranged around the chip boundary and connected to the internal core.

<img width="1033" height="543" alt="image" src="https://github.com/user-attachments/assets/c2234ef0-db8f-496c-92f5-ce308bf623f6" />


---

## 2. IP Blocks and Macros

The core area can contain several types of pre-designed blocks.

### Foundry IP

Foundry-provided IP blocks are specialized components supplied for a particular manufacturing technology.

Examples include:

* PLL
* ADC
* DAC

### Macros

Macros are relatively large reusable blocks that are integrated into the design as predefined units.

Examples include:

* RISC-V processor core
* SRAM

These blocks can be combined with other components such as GPIO and SPI interfaces to form a complete SoC.

<img width="1541" height="802" alt="image" src="https://github.com/user-attachments/assets/3dbd9878-959f-41b2-b378-98b6728901b9" />


---

## 3. RISC-V: From Instruction Set to Silicon

**RISC-V** is an Instruction Set Architecture (ISA). It defines the instructions, registers, and behavior that a processor implementation must support.

The ISA itself is not the physical processor. It can be implemented using different RTL architectures.

A simplified transformation is:

```text
RISC-V ISA
    ↓
RTL Implementation
    ↓
Gate-Level Netlist
    ↓
Physical Layout
```

For example, a C program can be compiled into RISC-V instructions. A processor such as **PicoRV32** implements those instructions using Verilog RTL. The RTL can then pass through synthesis and physical-design tools to eventually produce a physical layout.



---

## 4. Software and Hardware Abstraction

Software does not directly interact with the physical transistors of a processor. Several abstraction layers exist between software and hardware.

A simplified view is:

```text
Application
    ↓
Operating System / System Software
    ↓
Compiler / Assembler
    ↓
Instruction Set Architecture
    ↓
Processor Hardware
```

### Compilation Path

The transformation of a program can be represented as:

```text
Source Code
    ↓
Compiler
    ↓
RISC-V Instructions
    ↓
Assembler
    ↓
Machine Code
    ↓
Hardware
```

On the hardware-design side, another transformation takes place:

```text
RTL
    ↓
Synthesized Netlist
    ↓
Physical Implementation
    ↓
Chip Layout
```

Thus, software and hardware have different abstraction layers that eventually meet at the processor hardware.

---

## 5. Digital ASIC Design Flow

A Digital ASIC design requires three major elements:

### RTL

**Register Transfer Level (RTL)** describes the functionality and structure of the digital circuit.

For this project, the processor is represented using Verilog RTL.

### EDA Tools

**Electronic Design Automation (EDA)** tools automate different stages of chip development.

Examples include:

* **Yosys** — logic synthesis
* **OpenROAD** — physical implementation
* **OpenSTA** — static timing analysis
* Physical verification tools — DRC/LVS checking

### PDK

**Process Design Kit (PDK)** contains technology-specific information supplied for a semiconductor manufacturing process.

It can include:

* Standard-cell libraries
* Timing models
* Technology layers
* Design rules
* Physical layout information
* Electrical characteristics

This project uses the **SKY130 PDK**.

The overall relationship can be represented as:

```text
             RTL
              │
              ▼
        ┌─────────────┐
        │   EDA Tools │
        └─────────────┘
              ▲
              │
             PDK
              │
              ▼
        Physical Layout
              │
              ▼
            GDSII
```

<img width="768" height="670" alt="image" src="https://github.com/user-attachments/assets/d5a3cdfe-8b81-4c71-b4e4-f033718b9380" />

---

# 6. RTL-to-GDSII Process

The RTL-to-GDSII flow converts the logical description of a circuit into a physical representation suitable for fabrication.

### Main Stages

#### 1. RTL Design

The functionality of the processor is described using RTL and verified before physical implementation.

#### 2. Synthesis

The RTL is transformed into a gate-level netlist.

The synthesis process maps the design to standard cells available in the selected PDK.

#### 3. Floorplanning

The physical boundaries of the chip and core are established.

Important decisions include:

* Core dimensions
* Die dimensions
* I/O locations
* Macro locations

#### 4. Placement

Standard cells from the synthesized netlist are assigned physical positions inside the core.

Placement attempts to optimize:

* Cell density
* Wire length
* Congestion
* Timing

#### 5. Clock Tree Synthesis

CTS creates the clock-distribution network required by sequential elements.

Buffers are inserted to distribute the clock signal and control clock skew.

#### 6. Routing

The connections between cells are implemented using the available metal layers.

Routing generally consists of:

* Global routing
* Detailed routing

The resulting connections must satisfy the technology's physical design rules.

#### 7. Static Timing Analysis

STA verifies whether timing requirements are satisfied.

It evaluates parameters such as:

* Setup time
* Hold time
* Cell delay
* Interconnect delay
* Clock delay
* Slew
* Slack

#### 8. Physical Verification

The physical layout is checked using verification techniques such as:

* **DRC — Design Rule Check**
* **LVS — Layout Versus Schematic**

#### 9. GDSII Generation

Once the design passes the required checks, the final physical layout can be exported as a **GDSII** file.

```text
RTL
 ↓
Synthesis
 ↓
Floorplan
 ↓
Placement
 ↓
CTS
 ↓
Routing
 ↓
STA
 ↓
DRC / LVS
 ↓
GDSII
```

<img width="1518" height="670" alt="image" src="https://github.com/user-attachments/assets/0c9b4599-8c11-4113-8e48-a2271bfd9ccb" />


---

# 7. OpenLane and Open-Source ASIC Tools

**OpenLane** provides an automated RTL-to-GDSII environment by integrating several open-source EDA tools.

A simplified flow is:

```text
RTL + PDK
    ↓
   Yosys
    ↓
Synthesis
    ↓
 OpenROAD
    ↓
Floorplan
    ↓
Placement
    ↓
  CTS
    ↓
Routing
    ↓
 OpenSTA
    ↓
Timing Analysis
    ↓
Physical Verification
    ↓
  GDSII
```

Some important tools involved in the flow are:

| Tool        | Main Function           |
| ----------- | ----------------------- |
| Yosys       | RTL synthesis           |
| ABC         | Logic optimization      |
| OpenROAD    | Physical implementation |
| OpenSTA     | Static timing analysis  |
| TritonRoute | Detailed routing        |
| Magic       | Physical verification   |
| Netgen      | LVS                     |
| OpenLane    | Flow automation         |

<img width="1078" height="667" alt="image" src="https://github.com/user-attachments/assets/ebc9e409-0e69-4aea-9bc1-642659fd1ed8" />

---

# 8. SKY130 PDK Structure

The SKY130 PDK contains the technology files and libraries required by the ASIC tools.

Within the PDK, `libs.ref` contains reference libraries, including standard-cell libraries and SRAM-related libraries.

`libs.tech` contains technology files used by different tools.

For example, the standard-cell library can contain files such as:

```text
.lib
.lef
.tlef
```

These files provide different types of information.

* `.lib` → timing and cell characterization
* `.lef` → physical abstracts of cells
* `.tlef` → technology-level LEF information


---

# 9. PicoRV32A OpenLane Design

The OpenLane installation contains a `designs` directory where individual designs are maintained.

The PicoRV32A project contains:

```text
picorv32a/
├── config.tcl
├── src/
│   ├── picorv32a.v
│   └── picorv32a.sdc
└── ...
```

The important files include:

* `picorv32a.v` → processor RTL
* `picorv32a.sdc` → timing constraints
* `config.tcl` → design configuration


---

# 10. OpenLane Configuration

The main `config.tcl` file defines the parameters required to run the design.

Typical settings include:

* Design name
* RTL source files
* Clock port
* Clock period
* Library selection
* Technology configuration

A library-specific configuration file can also modify parameters related to the selected standard-cell library.

For this project, the SKY130 high-density standard-cell library is used.



---

# 11. Running OpenLane Using Docker

Docker is used to provide a controlled environment containing the OpenLane tools.

First, the available Docker images can be checked with:

```bash
docker images
```

This confirms whether the required OpenLane image is already available locally.


The OpenLane container can then be started using:

```bash
docker run -it \
  -v $PWD:/openLANE_flow \
  -v $PDK_ROOT:$PDK_ROOT \
  -e PDK_ROOT=$PDK_ROOT \
  -u $(id -u $USER):$(id -g $USER) \
  efabless/openlane:v0.21
```

The command performs several tasks:

* Mounts the current OpenLane directory into the container.
* Makes the PDK available inside the container.
* Passes the `PDK_ROOT` environment variable.
* Runs the container using the current user's UID/GID.

---

# 12. Launching the OpenLane Interactive Shell

After entering the container, the OpenLane flow can be started with:

```bash
./flow.tcl -interactive
```

This opens the interactive Tcl environment.

The OpenLane package is then loaded:

```tcl
package require openlane 0.9
```

After loading the package, OpenLane commands such as `prep` and `run_synthesis` become available.


---

# 13. Preparing the PicoRV32A Design

The design is prepared using:

```tcl
prep -design picorv32a
```

This command:

1. Loads the design configuration.
2. Identifies the selected PDK.
3. Selects the standard-cell library.
4. Creates a new run directory.
5. Prepares the files required for the selected flow.


---

# 14. OpenLane Run Directory

After the `prep` command finishes, OpenLane creates a timestamped run directory.

A typical structure contains:

```text
runs/
└── <timestamp>/
    ├── tmp/
    ├── results/
    ├── reports/
    ├── logs/
    ├── config.tcl
    └── ...
```

Different stages of the flow have their own directories.

For example:

```text
tmp/
├── synthesis/
├── placement/
├── cts/
├── routing/
└── ...
```

These directories are populated as the corresponding stages of the flow are executed.



---

# 15. Merged LEF

During preparation, OpenLane generates a merged LEF file.

```text
merged.lef
```

LEF provides an abstract physical representation of cells.

It contains information such as:

* Cell dimensions
* Pins
* Pin locations
* Metal layers
* Physical block boundaries

For example, a flip-flop standard cell can have pins such as:

```text
D
Q
SET_B
```


---

# 16. Generated Run Configuration

The run-specific `config.tcl` contains the configuration actually used for the current OpenLane execution.

It includes parameters related to:

* PDK paths
* Standard-cell libraries
* Cell padding
* Clock buffers
* Diode insertion
* Routing configuration
* Placement settings
* Other environment and flow variables

This generated configuration provides a record of the settings used during the run.


---

# 17. Running Synthesis

Synthesis is started using:

```tcl
run_synthesis
```

OpenLane invokes the synthesis tools to transform the PicoRV32A RTL into a gate-level implementation.

The synthesis process uses:

```text
PicoRV32A RTL
      ↓
    Yosys
      ↓
     ABC
      ↓
Standard-Cell Netlist
```

An OpenSTA timing check is also performed on the synthesized design.

The synthesis stage reports information such as:

* Cell count
* Area
* Netlist generation
* Timing summary
* TNS
* WNS


---

# 18. Synthesis Statistics

The synthesis statistics provide information about the generated circuit.

The recorded PicoRV32A results are:

| Parameter        |  Count |
| ---------------- | -----: |
| Total Wires      | 14,596 |
| Wire Bits        | 14,978 |
| Public Wires     |  1,565 |
| Public Wire Bits |  1,947 |
| Memories         |      0 |
| Processes        |      0 |
| Total Cells      | 14,876 |
| Flip-Flops       |  1,613 |

The standard-cell breakdown also shows how many instances of each type of cell were generated.


---

# 19. Flip-Flop Percentage

The proportion of flip-flops in the synthesized design can be calculated using:

```text
Flip-Flop Ratio =
(Flip-Flops / Total Cells) × 100
```

For PicoRV32A:

```text
= (1613 / 14876) × 100

≈ 10.84%
```

Therefore:

```text
Flip-Flop Ratio ≈ 10.84%
```

This means that approximately **10.84% of the synthesized cells are flip-flops**.

---

# 20. Synthesized Netlist

After synthesis, Yosys produces a gate-level Verilog netlist.

For this design, the synthesized file is:

```text
picorv32a.synthesis.v
```

Compared with the original RTL, the synthesized netlist is much larger because it contains the actual standard-cell instances and internal connections generated during synthesis.

It also contains automatically generated names for many internal signals.

<img width="626" height="312" alt="image" src="https://github.com/user-attachments/assets/a7cb03fb-986e-43a9-a8fd-7302ee508682" />


---

# 21. Synthesis Report

The synthesis report contains detailed information about the resulting implementation.

The report includes:

* Standard-cell types
* Number of instances
* Total cells
* Wire information
* Design area

The `stat` report provides a complete summary of the synthesized design.



---

# 22. Synthesis and Timing Reports

OpenLane stores reports generated during synthesis inside:

```text
reports/synthesis/
```

This directory can contain:

```text
1-yosys_*.rpt
2-opensta_*.rpt
```

The Yosys reports provide synthesis and cell statistics, while OpenSTA reports contain timing-related information.

Timing reports can include:

* Minimum delay
* Maximum delay
* Slew
* Timing paths
* Cell delay
* Net delay


---

# 23. Timing Path Analysis

An OpenSTA timing report can be used to inspect individual timing paths.

A timing path report shows how delay accumulates as a signal travels through the circuit.

It can provide information such as:

* Start point
* End point
* Cell delay
* Net delay
* Fanout
* Capacitance
* Slew
* Incremental delay

This helps identify critical paths and understand where timing delay is being introduced.


---

# 24. Complete Physical Design Flow

The complete ASIC implementation process studied in this project can be summarized as:

```text
RTL Design
    ↓
Synthesis
    ↓
Floorplanning
    ↓
Power Planning
    ↓
Placement
    ↓
Clock Tree Synthesis
    ↓
Routing
    ↓
Static Timing Analysis
    ↓
DRC / LVS
    ↓
Signoff
    ↓
GDSII
```

Each stage converts the design into a more physically detailed representation while checking constraints from the previous stages.

---

# 25. Tools and Technologies

| Component | Role                           |
| --------- | ------------------------------ |
| PicoRV32A | RISC-V processor RTL           |
| OpenLane  | Automated RTL-to-GDSII flow    |
| Yosys     | Logic synthesis                |
| ABC       | Logic optimization             |
| OpenROAD  | Physical implementation        |
| OpenSTA   | Static timing analysis         |
| SKY130    | Semiconductor technology / PDK |
| Docker    | Execution environment          |
| GDSII     | Final physical layout format   |

---

# 26. Key Concepts Learned

This project provided practical exposure to the basic ASIC physical-design process.

The major concepts covered are:

* Chip anatomy
* Pads
* Die
* Core
* Foundry IP
* Macros
* RISC-V ISA
* RTL
* EDA tools
* PDK
* Synthesis
* Gate-level netlist
* Floorplanning
* Power planning
* Placement
* Clock Tree Synthesis
* Routing
* Static Timing Analysis
* DRC
* LVS
* Signoff
* GDSII

The most important transformation is:

```text
RTL
  ↓
Gate-Level Netlist
  ↓
Physical Implementation
  ↓
Verification
  ↓
GDSII
```

---

# 27. Conclusion

The PicoRV32A project demonstrates how an RTL processor can be taken into an open-source ASIC implementation environment.

Using **OpenLane and the SKY130 PDK**, the project explores the connection between RTL design, synthesis, physical implementation, and verification.

The practical lab begins with configuring the PicoRV32A design, setting up the PDK and OpenLane environment, creating a run, and executing synthesis.

The synthesis stage produces a gate-level netlist along with cell, area, and timing reports. The measured design contains **14,876 total cells and 1,613 flip-flops**, resulting in a calculated flip-flop ratio of approximately **10.84%**.

The remaining physical-design stages build on this synthesized design and continue toward the final verified **GDSII layout**.

```text
PicoRV32A RTL
      ↓
   OpenLane
      ↓
   Synthesis
      ↓
Physical Design
      ↓
Verification
      ↓
    GDSII
```

This project provides a foundation for understanding how an RTL description eventually becomes a physical ASIC layout using open-source EDA tools and the SKY130 technology.
