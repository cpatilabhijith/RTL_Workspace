# Day 7 — Floorplanning and Placement using OpenLane

## Introduction

After completing synthesis, the next step in the physical design flow is to convert the synthesized design into an actual physical arrangement. In this stage, the logical design is given a physical area and the standard cells are positioned inside the chip.

This work focuses on **floorplanning and placement of the PicoRV32A processor using OpenLane**. The generated layouts were inspected using **Magic** to understand how the design changes from an empty floorplan to a completely placed circuit.

The overall flow followed in this work is:

```text
Synthesized Netlist
        |
        v
   Floorplanning
        |
        v
   Power Planning
        |
        v
    Pin Placement
        |
        v
      Placement
        |
        v
Placement Optimization
```

---

## 1. Understanding the Physical Layout

A synthesized netlist only describes the logical connections between different gates. It does not specify where those gates should physically exist on the chip.

Physical design adds this information by defining:

* Chip dimensions
* Core dimensions
* Cell locations
* I/O locations
* Power distribution
* Routing space
* Placement constraints

The two important physical regions are the **die** and the **core**.

### Die

The die represents the complete silicon region allocated for the chip.

### Core

The core is the internal region where the standard cells are arranged. Some space around the core is required for I/O, power structures and other physical requirements.

```text
+--------------------------------+
|              DIE               |
|                                |
|       +----------------+       |
|       |                |       |
|       |      CORE      |       |
|       |                |       |
|       +----------------+       |
|                                |
+--------------------------------+
```
<img width="1914" height="955" alt="image" src="https://github.com/user-attachments/assets/e0b9c496-23b7-41e9-bef1-2f46a6d16701" />

---

## 2. Area, Utilization and Aspect Ratio

### Aspect Ratio

The shape of the core is determined using its aspect ratio.

```text
Aspect Ratio = Core Height / Core Width
```

When the value is close to `1`, the core is approximately square. Other values produce a rectangular core.

### Core Utilization

Utilization represents the percentage of the core area occupied by logic cells.

```text
Utilization =
(Cell Area / Core Area) × 100
```

Very high utilization can create routing congestion because there is less free space for wires and optimization cells. Therefore, suitable utilization is selected during floorplanning.

For this OpenLane run, the design-specific configuration used a target core utilization of approximately **35%**.

---

## 3. Floorplan Formation

Floorplanning determines the physical framework in which the design will be implemented.

The floorplanning stage mainly establishes:

```text
Core / Die Size
      |
      +---- Aspect Ratio
      |
      +---- Cell Utilization
      |
      +---- I/O Locations
      |
      +---- Power Distribution
      |
      +---- Reserved Regions
```

A properly planned floorplan provides sufficient space for placement and routing and helps avoid unnecessary congestion.

---

## 4. OpenLane Floorplan Parameters

OpenLane uses configuration variables to control the physical design flow.

Some important floorplanning parameters are:

```text
FP_CORE_UTIL
FP_ASPECT_RATIO
FP_SIZING
DIE_AREA

FP_IO_HMETAL
FP_IO_VMETAL
FP_IO_MODE

FP_PDN_VPITCH
FP_PDN_HPITCH
```

These parameters control aspects such as:

* Core utilization
* Core shape
* Die dimensions
* I/O metal layers
* Power-grid spacing

The default OpenLane configuration can be overridden by design-specific configuration files.

---

## 5. Pre-Placed and Fixed Cells

Certain cells or blocks may need to remain at predetermined locations instead of being freely moved by the placement tool.

Examples include:

* Memory blocks
* Clock-related cells
* Comparators
* Multiplexers
* Large reusable IP blocks

Such blocks can be assigned fixed locations during the physical design process. The remaining standard cells are then placed around these fixed regions.

This approach is useful when particular blocks have physical, timing or connectivity requirements.

---

## 6. Decoupling Capacitors

Digital circuits can experience temporary fluctuations in their supply voltage when a large number of cells switch simultaneously.

A **decoupling capacitor**, or decap, helps by acting as a small local source of charge during sudden current demand.

```text
        VDD
         |
    +----+----+
    |  Decap  |
    +----+----+
         |
       Circuit
         |
        VSS
```

The capacitor supplies current locally during switching and then recharges from the main power network.

During the floorplan stage, decap cells are inserted at suitable locations within the core.

---

## 7. Power Distribution Network

The power network is responsible for delivering supply voltage and ground to the cells across the chip.

The two primary power connections are:

```text
VDD → Supply
VSS → Ground
```

Instead of depending on a single connection, the power network distributes these signals through multiple structures.

```text
             VDD / VSS
                  |
            Power Rings
                  |
            Power Straps
                  |
          Standard Cell Rows
```

A distributed power network helps reduce effects such as:

* IR voltage drop
* Ground bounce
* Supply noise

Therefore, power planning is an important part of the floorplanning process.

---

## 8. I/O Pin Arrangement

The synthesized netlist describes how signals are connected, but it does not determine their physical positions.

Pin placement assigns the actual physical locations of input and output pins.

Pins can be distributed along:

```text
        TOP
   ----------------
   |              |
LEFT               RIGHT
   |              |
   ----------------
       BOTTOM
```
<img width="1895" height="959" alt="image" src="https://github.com/user-attachments/assets/a6fa92f6-62d1-40c6-a65a-b66828d413ab" />

The location of a pin can be selected based on signal connectivity and the position of important blocks.

---

## 9. Placement Blockages

Some portions of the core may need to be reserved for specific purposes.

A **placement blockage** prevents the automatic placement tool from putting standard cells into a particular region.

```text
+-------------------------+
| Standard Cell Area      |
|                         |
|    Reserved Region      |
|    / Blocked Area /     |
|                         |
| Standard Cell Area      |
+-------------------------+
```

This provides better control over the physical arrangement and prevents cells from occupying areas reserved for other physical structures.

---

## 10. Running the Floorplan Stage

The floorplan stage was executed using OpenLane with:

```tcl
run_floorplan
```

This stage performs several physical design operations, including:

```text
Netlist / DEF Preparation
        ↓
I/O Pin Placement
        ↓
Tap Cell Insertion
        ↓
Power Distribution Network
```

The completed floorplan was then inspected to verify the generated physical structures.

---

## 11. Floorplan Configuration Used in the Run

The technology-specific configuration modified the default core utilization for this design.

The important setting was:

```tcl
FP_CORE_UTIL = 35
```

The expanded OpenLane configuration confirmed that the floorplan run used a core utilization target of **35%**.

The floorplan generated for the design had approximately:

```text
Die Width  ≈ 660.7 µm
Die Height ≈ 671.4 µm
```

The dimensions were obtained from the generated DEF information.

---

## 12. Inspecting the Floorplan with Magic

The generated DEF and LEF information can be opened in Magic for visual inspection.

Example command:

```bash
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &
```

At the floorplan stage, the core does not yet contain all the standard cells.

The layout mainly shows:

* Core boundary
* I/O pins
* Standard-cell rows
* Tap cells
* Decap cells
* Power-related structures

Zooming into the layout makes individual fixed cells and I/O pins visible.
<img width="1920" height="983" alt="image" src="https://github.com/user-attachments/assets/31d10a83-2d36-4dde-8b11-03a3d40cdc6f" />
<img width="1918" height="1033" alt="Screenshot 2026-09-10 171116" src="https://github.com/user-attachments/assets/a2e809a0-a6f0-4f81-9462-a62654ea08fa" />
<img width="1920" height="983" alt="image" src="https://github.com/user-attachments/assets/0fcede35-0997-40eb-8347-02903a27293e" />

---

## 13. Starting the Placement Stage

After completing floorplanning, the next step is to position the standard cells.

The placement stage was executed using:

```tcl
run_placement
```

Placement determines where the physical instances corresponding to the synthesized netlist should be located.

The process can be represented as:

```text
Logical Netlist
      ↓
Physical Cell Mapping
      ↓
Global Placement
      ↓
Detailed Placement
      ↓
Placement Optimization
```

---

## 14. Mapping Logic to Physical Cells

Each logical gate in the synthesized netlist must correspond to an available physical cell from the standard-cell library.

The library provides information about:

* Cell dimensions
* Logic function
* Drive capability
* Timing characteristics
* Power characteristics

The same logical function may be available in different drive strengths.

For example:

```text
Small Cell  → Lower area
Large Cell  → Higher drive strength
```

The placement and optimization tools select suitable cells according to physical and timing requirements.


---

## 15. Placement Optimization

Once the cells are initially positioned, the tool evaluates the quality of the placement.

Important factors include:

```text
Wire Length
Capacitance
Delay
Congestion
Timing
```

If a connection is too long, the tool may insert a buffer or repeater.

```text
Before:

Cell -------------------------- Cell


After:

Cell -------- Buffer --------- Cell
```

Buffers help improve signal transition and reduce the effect of long interconnects.

---

## 16. Placement Results

The completed placement stage produced the following important statistics:

```text
Total Instances       : 21,699
Fixed Instances       : 6,354
Number of Nets        : 15,449

Design Area           : 420,473.3 µm²
Utilization           : 36%
Utilization with Pad  : 55%

Placement Rows        : 238
```

These values provide an indication of the physical size and complexity of the placed design.

The difference between the normal utilization and padded utilization represents the additional space considered during placement optimization.

---

## 17. Viewing the Final Placement

The generated placement DEF file is:

```text
picorv32a.placement.def
```

It can be viewed using Magic with the merged LEF.

```bash
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &
```

Unlike the floorplan view, the placement view contains a large number of standard cells distributed throughout the core.

A detailed view can show cells such as:

```text
Flip-Flops
Multiplexers
Buffers
AND/OR/Invert Cells
Other Standard Cells
```

This demonstrates how the synthesized logical design has been transformed into a physical arrangement.
<img width="1917" height="1003" alt="Screenshot 2026-09-10 171228" src="https://github.com/user-attachments/assets/c4fee6f3-3659-4697-8a1b-bebe4ff27166" />

---

## 18. Floorplan vs Placement

The main difference observed between the two stages is:

| Floorplanning                    | Placement                        |
| -------------------------------- | -------------------------------- |
| Defines physical boundaries      | Places standard cells            |
| Establishes core and die         | Fills the core with cells        |
| Places/defines I/O structure     | Optimizes cell positions         |
| Creates initial power structures | Evaluates wire length and timing |
| Inserts tap/decap structures     | Performs detailed placement      |

In simple terms:

```text
Floorplanning
      ↓
"Where should the design fit?"

Placement
      ↓
"Where should each cell go?"
```

---

## 19. Overall Learning

The physical design process became clearer by running the actual OpenLane stages and examining the generated layouts.

The important concepts covered were:

```text
Die and Core
Aspect Ratio
Utilization
Floorplanning
Pre-Placed Cells
Decoupling Capacitors
Power Planning
Pin Placement
Placement Blockages
Standard Cell Placement
Placement Optimization
Placement Statistics
```

The complete flow achieved so far is:

```text
RTL
 ↓
Synthesis
 ↓
Synthesized Netlist
 ↓
Floorplanning
 ↓
Power Planning
 ↓
Pin Placement
 ↓
Placement
 ↓
Placement Optimization
```

## Conclusion

This stage demonstrated how the synthesized PicoRV32A design is transformed from a logical netlist into a physical layout. The floorplan established the chip area, utilization, I/O arrangement and power-related structures, while placement assigned physical locations to the standard cells and optimized their arrangement.

The OpenLane run successfully progressed through both **floorplanning and placement**, and the resulting layouts were examined using Magic. The next physical design stages are **Clock Tree Synthesis (CTS), routing, and timing analysis**.
