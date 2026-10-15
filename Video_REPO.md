# RTL-and-Physical-Design-Workshop

# 🎬 VSD RTL Design Workshop & VSDBabySoC — Video Presentation

## 📌 Overview

This presentation showcases my learning journey through the **VSD RTL Design Workshop (Day 1 to Day 10)** and my implementation work on the **VSDBabySoC project**.

The presentation covers the concepts learned during the workshop, supported by practical examples involving RTL design, simulation, synthesis, Gate-Level Simulation (GLS), physical design, and Static Timing Analysis (STA).

My project repository contains the RTL-to-GLS workflow, PVT timing analysis, and physical-design implementation through routing, followed by post-route timing analysis.

🔗 **GitHub Repository:** [RTL_Workspace — VSDBabySoC](https://github.com/cpatilabhijith/RTL_Workspace/tree/main/VSDBabySoC)

## ▶️ Watch the Video

**[🎥 Click here to watch my VSD RTL Workshop and VSDBabySoC presentation](https://docs.google.com/videos/d/1p8jiK6PsSokO4Wmd68xFPuoJacXnEShvqU7XNQThUHw/edit?usp=drivesdk)**
                          (or)
https://docs.google.com/videos/d/1p8jiK6PsSokO4Wmd68xFPuoJacXnEShvqU7XNQThUHw/edit?usp=drivesdk

## 📊 PowerPoint Presentation

The PowerPoint presentation explains the workshop modules, digital-design concepts, implementation flow, tool outputs, and practical results from the VSDBabySoC project.

**[📥 Click here to view my Workshop PowerPoint](https://docs.google.com/presentation/d/1DZ7ou1deMmAGXBX3_yd5pJt63aDwPylM/edit?usp=drivesdk&ouid=106668357490989954010&rtpof=true&sd=true)**

## 📚 Topics Covered

### 1. RTL Design Workshop — Day 1 to Day 5

- RTL design, simulation, and synthesis
- Flip-flops and synchronous/asynchronous resets
- PVT corners, Liberty timing files, and design hierarchy
- Combinational and sequential logic optimization
- Gate-Level Simulation (GLS)
- Blocking and non-blocking assignments
- Latches, `case` statements, and `generate` loops

### 2. Physical Design — Day 6 to Day 10

- RTL-to-GDSII design-flow concepts
- Synthesis and standard-cell analysis
- Floorplanning and power planning
- Global and detailed placement
- Clock Tree Synthesis (CTS)
- CMOS inverter, SPICE, and layout concepts
- Setup and hold timing analysis
- Global and detailed routing
- Design Rule Check (DRC), Layout Versus Schematic (LVS), antenna checks, and sign-off concepts

### 3. VSDBabySoC Project

- RVMYTH RISC-V processor core
- PLL and DAC macro integration
- RTL simulation and synthesis
- Gate-Level Netlist generation and GLS
- Floorplanning and macro placement
- Standard-cell placement and clock tree synthesis
- Global and detailed routing using the OpenROAD flow
- Post-route STA across 16 PVT corners
- Critical-path analysis and timing-report interpretation

## 🛠️ Tools and Technologies

- Verilog HDL
- Icarus Verilog
- GTKWave
- Yosys
- OpenROAD-flow-scripts (ORFS)
- OpenROAD
- OpenSTA
- SKY130 HD PDK
- `sky130_fd_sc_hd` standard-cell library
- OpenLane concepts and physical-verification tools, where applicable

## 📈 Key Implementation Results

The VSDBabySoC physical-design flow has progressed through routing, followed by post-route static timing analysis.

| Parameter | Result |
|---|---|
| PVT corners analyzed | 16 |
| Setup timing passed | 10 corners |
| Setup timing violations | 6 corners |
| Hold timing | Passed at all 16 corners |
| Worst setup corner | `SS_n40C_1v28` |
| Worst setup WNS | −29.320 ns |
| Worst setup TNS | −14575.390 ns |

The results show that setup timing passes at all six fast-fast (FF) corners, both typical-typical (TT) corners, and two slow-slow (SS) corners. Six SS corners still have setup violations.

These results helped me understand how process, voltage, and temperature affect circuit delay and why critical-path optimization is important for timing closure.

For detailed configuration, commands, physical-design screenshots, PVT tables, and timing observations, refer to the [VSDBabySoC project documentation](https://github.com/cpatilabhijith/RTL_Workspace/tree/main/VSDBabySoC).

## 🎯 Key Learnings

This workshop strengthened my understanding of the digital VLSI design process, from RTL coding and functional verification to synthesis, physical implementation, and timing analysis.

Working on VSDBabySoC helped me connect theoretical concepts with practical implementation, understand the role of standard-cell libraries and physical-design tools, and analyze timing behavior across multiple PVT corners.

It also provided practical experience interpreting placement, clock-tree, routing, and static timing reports.

## 👨‍💻 Author

**C. Abhijith**  
Electronics and Communication Engineering (ECE - C)
Registration No.: **24EG104C36**  
GitHub: [cpatilabhijith](https://github.com/cpatilabhijith)

---

*Note: The project has been completed through routing and post-route timing analysis. Setup timing violations remain at six PVT corners. DRC, LVS, antenna, and final GDSII sign-off should only be claimed after verifying the corresponding reports.*
