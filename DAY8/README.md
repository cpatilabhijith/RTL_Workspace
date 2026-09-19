# Day 8 — Sky130 CMOS Characterization, Fabrication & Layout Verification

<p align="center">
 <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/024ef40a-08c8-4284-bbae-345fdea6c976" />
</p>

<h1 align="center">Sky130 CMOS Inverter Study</h1>

<p align="center">
  <b>Device Physics → SPICE → Layout Extraction → Timing → DRC</b>
</p>

<p align="center">
  A hands-on study of CMOS inverter characterization, the 16-mask fabrication
  flow, Sky130 layout extraction, ngspice simulation, and physical verification.
</p>

---

## 📌 About 8TH Day

Day 8 moves from the higher-level digital implementation flow into the
**transistor, fabrication, and standard-cell characterization level**.

The work combines two major areas:

* **CMOS inverter characterization using SPICE**
* **CMOS fabrication using the 16-mask process**

The practical portion then connects these concepts to an actual
**Sky130 inverter layout** using Magic and ngspice.

The complete practical flow followed was:

```text
Sky130 Standard Cell
        │
        ▼
   Magic Layout
        │
        ▼
   Layer Inspection
        │
        ▼
   Layout Extraction
        │
        ▼
    SPICE Netlist
        │
        ▼
   Model Correction
        │
        ▼
    ngspice Run
        │
        ▼
Transient Waveforms
        │
        ▼
Delay / Transition Measurements
        │
        ▼
     DRC Analysis
```

---

# 📚 Contents

1. [CMOS Inverter Fundamentals](#1--cmos-inverter-fundamentals)
2. [SPICE Netlist Structure](#2--spice-netlist-structure)
3. [DC Characterization](#3--dc-characterization)
4. [Switching Point VM](#4--switching-point-vm)
5. [Transient Characterization](#5--transient-characterization)
6. [16-Mask CMOS Fabrication](#6--16-mask-cmos-fabrication)
7. [Sky130 Standard-Cell Layout](#7--sky130-standard-cell-layout)
8. [Magic Layout Inspection](#8--magic-layout-inspection)
9. [Layout-to-SPICE Extraction](#9--layout-to-spice-extraction)
10. [Sky130 Device Models](#10--sky130-device-models)
11. [Extracted Netlist Simulation](#11--extracted-netlist-simulation)
12. [Timing Measurements](#12--timing-measurements)
13. [DRC Rule Investigation](#13--drc-rule-investigation)
14. [Results](#14--results)
15. [Complete Day 8 Flow](#15--complete-day-8-flow)
16. [Tools](#16--tools)
17. [Conclusion](#17--conclusion)

---

# 1. 🔲 CMOS Inverter Fundamentals

The CMOS inverter is one of the simplest examples of complementary
MOS transistor logic.

It contains:

* One PMOS transistor
* One NMOS transistor
* Common gate input
* Common drain output
* PMOS connected toward the supply
* NMOS connected toward ground

### Basic Representation

```text
                 VPWR
                   │
                ┌─────┐
        A ──────┤ PMOS│
                └──┬──┘
                   │
                   ├──────── Y
                   │
                ┌──┴──┐
        A ──────┤ NMOS│
                └──┬──┘
                   │
                  VGND
```
<img width="2001" height="1824" alt="image" src="https://github.com/user-attachments/assets/4c2b8ca5-2fc6-4c3d-b963-0b5d568391ae" />

The circuit performs logical inversion:

```text
A = 0  →  Y = 1
A = 1  →  Y = 0
```

### Truth Table

| Input A | PMOS | NMOS | Output Y |
| :-----: | :--: | :--: | :------: |
|    0    |  ON  |  OFF |     1    |
|    1    |  OFF |  ON  |     0    |

---

# 2. 🧪 SPICE Netlist Structure

SPICE describes a circuit using text-based circuit statements.

A useful way to understand a SPICE deck is to divide it into three parts:

### 1. Connectivity

Defines how devices are connected.

```text
Drain → Gate → Source → Bulk
```

### 2. Device Parameters

Examples include:

* Width
* Length
* Resistance
* Capacitance
* Supply voltage

### 3. Device Models

The model tells SPICE how a particular transistor behaves electrically.

For example:

```spice
M1 Y A VPWR VPWR PMOS W=0.375u L=0.25u
M2 Y A VGND VGND NMOS W=0.375u L=0.25u
```

The extracted Sky130 layout later uses BSIM4 device models from the
standard-cell library.

---

# 3. 📈 DC Characterization

A CMOS inverter can first be studied without considering switching speed.

The input voltage is gradually increased from ground to the supply voltage,
while the output voltage is observed.

A typical transfer curve looks like:

```text
VOUT
 │
 │████████████
 │           \
 │            \
 │             \
 │              ███████
 │
 └──────────────────────── VIN
              VM
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/86654137-1600-463c-92a4-32920ce7ab6a" />

This curve is called the:

**Voltage Transfer Characteristic (VTC)**

It provides information about the inverter's logic behavior and switching
region.

---

# 4. ⚖️ Switching Point VM

The inverter switching threshold is represented by:

```text
VM
```

At the switching point:

```text
VIN = VOUT = VM
```

At this operating condition, the PMOS and NMOS currents balance:

```text
IDP = -IDN
```

Both devices are in saturation around the idealized switching point.

### Why VM Matters

The value of `VM` depends on transistor strength.

The relative sizing can be expressed as:

```text
W/L
```

If the PMOS is made wider, its drive capability increases and the switching
point moves toward a higher input voltage.

This is related to the lower carrier mobility of holes compared with
electrons, which is one reason PMOS devices are often wider in a balanced
CMOS inverter.

---

# 5. ⏱️ Transient Characterization

DC analysis tells us **where** the inverter switches.

Transient analysis tells us **how quickly** it switches.

A pulse signal can be applied to the input:

```text
VIN

HIGH ──────┐       ┌──────
           │       │
           │       │
LOW ───────┴───────┴──────── time
```

The output responds in the opposite direction:

```text
VOUT

HIGH ──────┐       ┌──────
           │       │
           └───────┘
LOW
           ──────────────── time
```

Two important timing measurements are obtained.

### Transition Time

Transition time describes how quickly the output changes between voltage
levels.

For this experiment, the measurement was taken between:

```text
20% → 80%
```

for the relevant output transition.

### Propagation Delay

Propagation delay measures the time difference between the input and output
50% voltage crossings.

```text
Input 50% crossing
        │
        │<------ delay ------>│
                              │
                       Output 50% crossing
```

These measurements are important for understanding standard-cell timing.

---

# 6. 🏭 16-Mask CMOS Fabrication

A CMOS device is constructed through multiple semiconductor processing
steps.

The fabrication study followed the progression from a silicon wafer to
transistors, contacts, interconnect, and final passivation.

```text
Silicon Wafer
     ↓
Isolation
     ↓
Well Formation
     ↓
Gate Preparation
     ↓
LDD Formation
     ↓
Source / Drain
     ↓
Contacts
     ↓
Local Interconnect
     ↓
Metal Layers
     ↓
Passivation
     ↓
Finished CMOS Structure
```

---

## 6.1 Starting Wafer

The process begins with a P-type silicon substrate.

The substrate acts as the foundation for the transistor structures.

A `<100>` silicon orientation is commonly used because of its favorable
silicon/oxide interface characteristics.
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f1734e88-8d32-43bc-bce1-1a3bed356400" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c090fc01-eeb8-498c-adcb-aad3e4ad5cfa" />
                        
---

## 6.2 Isolation Formation — Mask 1

The active areas are defined while other regions are isolated.

The process includes:

```text
Oxide
  ↓
Silicon Nitride
  ↓
Photoresist
  ↓
Patterning
  ↓
LOCOS Oxidation
```

LOCOS produces field oxide around the active region.

A characteristic effect of LOCOS is the:

**Bird's-beak**

where the oxide extends laterally underneath the masking layer.

---

## 6.3 Well Formation — Mask 2

Well regions establish the required transistor body regions.

The process involves dopant implantation followed by thermal processing.

Conceptually:

```text
Dopant Implantation
        ↓
Drive-In / Activation
        ↓
Well Formation
```

The PMOS is formed inside an N-type well, while the NMOS is formed in a
P-type region.

---

## 6.4 Gate Preparation — Masks 4, 5 & 6

The gate region determines where the transistor channel is controlled.

Important steps include:

```text
Sacrificial Oxide
       ↓
Threshold Adjustment
       ↓
Gate Oxide
       ↓
Polysilicon Deposition
       ↓
Polysilicon Doping
       ↓
Gate Patterning
```

The gate length and gate oxide characteristics strongly influence transistor
behavior.

---

## 6.5 LDD Formation — Masks 7 & 8

Lightly Doped Drain structures are introduced before the heavy source/drain
implant.

```text
        LDD             LDD
         ↓               ↓
─────────┐             ┌─────────
         │    GATE     │
─────────┴─────────────┴─────────
```

The LDD structure helps reduce the electric field near the drain and limits
hot-carrier-related degradation.

Sidewall spacers are then formed around the gate.

---

## 6.6 Source / Drain Implantation — Masks 9 & 10

The heavily doped source and drain regions are formed after the spacer
formation.

```text
NMOS → N+ Source / Drain
PMOS → P+ Source / Drain
```

Thermal annealing is used to activate the implanted dopants and repair
implantation-related crystal damage.

---

## 6.7 Local Interconnect — Mask 11

After transistor formation, the device terminals need electrical access.

The process includes:

```text
Surface Preparation
       ↓
Titanium Deposition
       ↓
Thermal Processing
       ↓
Local Interconnect Formation
       ↓
Pattern Definition
```

Local interconnect provides short-range connections between device regions.

---

## 6.8 Metal and Passivation — Masks 12–16

Higher-level interconnect is built using repeated dielectric, contact,
metal, and planarization steps.

A simplified representation is:

```text
Passivation
───────────────
Metal Layer
───────────────
Via
───────────────
Metal Layer
───────────────
Contact
───────────────
Device
───────────────
Silicon
```

The final mask opens the required regions in the passivation layer so that
the bond-pad connections can be accessed.

---

# 7. 🧱 Sky130 Standard-Cell Layout

For the practical experiment, a Sky130 inverter standard-cell layout was
used.

The reference design was obtained from the standard-cell design repository
and opened with Magic.

The cell contains important physical structures such as:

* PMOS
* NMOS
* Polysilicon gate
* Diffusion
* Contacts
* Metal
* VPWR
* VGND
* Input A
* Output Y

### Layout Concept

```text
       VPWR
══════════════════════

       PMOS
 ┌──────────────────┐
 │                  │
 └────────┬─────────┘
          │
          │ Y
          │
 ┌────────┴─────────┐
 │       NMOS       │
 │                  │
 └──────────────────┘

══════════════════════
       VGND

        ↑
        │
   POLY GATE
        │
        A
```
<img width="1915" height="1007" alt="Screenshot 2026-09-11 071219" src="https://github.com/user-attachments/assets/129280fb-915f-4f6f-973f-0684aa31f61e" />

---

# 8. 🔎 Magic Layout Inspection

Magic was used to inspect the physical implementation of the inverter.

The technology file used was:

```text
sky130A.tech
```
<img width="1918" height="1078" alt="Screenshot 2026-09-12 074055" src="https://github.com/user-attachments/assets/33488956-31fe-4027-b310-d8e1595fa154" />

The layout was launched using:

```bash
magic -T sky130A.tech sky130_inv.mag &
```

Once the layout was opened, the transistor regions and routing layers could
be inspected directly.

### Identifying Devices

Magic's `what` command was used from the `tkcon` console.

The selected regions were checked to identify:

```text
nmos
pmos
```

This provided a direct connection between the physical layout and the
transistor-level circuit.

---

# 9. 🔄 Layout-to-SPICE Extraction

One of the important practical parts of this experiment was generating a
SPICE representation directly from the physical layout.

Inside Magic:

```text
extract all
```

was used to extract the layout information.

The extraction-to-SPICE conversion was then performed using:

```text
ext2spice cthresh 0 zthresh 0
ext2spice
```

This generated:

```text
sky130_inv.ext
sky130_inv.spice
```

The extracted file contains transistor connectivity and parasitic
information obtained from the physical geometry.

---

# 10. 📦 Sky130 Device Models

The first extracted SPICE file could not immediately be used as a complete
simulation deck.

The extracted transistor instances referenced model names associated with
Magic's extraction environment.

The available Sky130 model libraries were inspected inside the standard-cell
repository.

Important model files included:

```text
pshort.lib
nshort.lib
```

The usable model names identified were:

```text
pshort_model.0
nshort_model.0
```

These correspond to the PMOS and NMOS BSIM4 models required for the
simulation.

### Model Mapping

| Device | Library      | Model            |
| :----: | ------------ | ---------------- |
|  PMOS  | `pshort.lib` | `pshort_model.0` |
|  NMOS  | `nshort.lib` | `nshort_model.0` |

---

# 11. 🧪 Extracted Netlist Simulation

The extracted circuit was modified into a usable ngspice simulation deck.

The model libraries were included using:

```spice
.include ./libs/pshort.lib
.include ./libs/nshort.lib
```
<img width="1915" height="1010" alt="Screenshot 2026-09-12 183043" src="https://github.com/user-attachments/assets/48e95cb1-8965-459a-a5cc-84c6c2c43c56" />

A subcircuit was defined for the inverter:

```spice
.subckt sky130_inv A Y VPWR VGND
```

The transistor instances were then connected to the appropriate Sky130
models.

An input pulse was added to stimulate the inverter.

Conceptually:

```text
Pulse Input
     │
     ▼
Sky130 CMOS Inverter
     │
     ▼
Output Waveform
```

The transient simulation was executed using ngspice.

The initial raw extraction produced a model/subcircuit error, but after
correcting the model references and simulation deck, the transient
simulation completed successfully.

---

# 12. 📊 Timing Measurements

The simulated output waveform was examined in ngspice.

The input and output transitions were measured using the waveform cursor.

### Measured Results

| Measurement     | Definition             | Measured Value |
| --------------- | ---------------------- | -------------: |
| Rise Transition | 20% → 80%              | **0.06194 ns** |
| Fall Transition | 20% → 80%              | **0.04267 ns** |
| Cell Rise Delay | 50% input → 50% output | **0.05953 ns** |
| Cell Fall Delay | 50% input → 50% output | **0.05941 ns** |

### Timing Interpretation

```text
                 OUTPUT
                   │
          80% ────●
                  /
                 /
          20% ─●
               │
               └──────── time

             Rise Transition
```

The measured values demonstrate how physical layout, transistor properties,
and parasitic capacitances contribute to the timing behavior of a standard
cell.

---

# 13. 🛡️ DRC Rule Investigation

Design Rule Checking verifies whether a physical layout satisfies the
manufacturing constraints of the technology.

Instead of only checking whether DRC reports errors, the experiment
investigated **why** individual violations occurred.

Magic's command:

```text
drc why
```

was used to obtain the specific rule associated with a violation.

### Metal3 Examples

Two observed rules were:

```text
Metal3 spacing < 0.3um
```

and

```text
Metal3 minimum area < 0.24um²
```

This demonstrates that a DRC error is not simply a generic failure; it can
be traced to a particular geometric manufacturing requirement.

---

## VIA2 Inspection

A Metal3 test structure containing multiple via/contact cuts was inspected.

The following commands were used:

```text
paint m3contact
cif see VIA2
```

The VIA2 geometry could then be examined separately.

A selected region measured approximately:

```text
0.190 µm × 0.200 µm
```

which corresponds to an area of approximately:

```text
0.038 µm²
```

---

## Poly Layer Inspection

The poly layer was also examined using Magic.

A selected poly region showed approximately:

```text
0.335 µm × 0.210 µm
```

Area:

```text
≈ 0.070 µm²
```

The reference test structure identified this region as:

```text
poly.9
```

---

# 14. 📋 Results

The Day 8 practical work produced results in three major categories.

### Device Characterization

```text
CMOS Inverter
      ↓
DC / Transient Analysis
      ↓
VM + Timing Parameters
```

### Layout Extraction

```text
Magic Layout
      ↓
Extraction
      ↓
SPICE Netlist
      ↓
Sky130 Models
      ↓
ngspice
```

### Physical Verification

```text
Layout
  ↓
DRC
  ↓
Rule Identification
  ↓
Geometry Inspection
```
<img width="1918" height="988" alt="Screenshot 2026-09-12 185953" src="https://github.com/user-attachments/assets/f3bc0724-6357-4af5-a060-2923772f029f" />
<img width="1918" height="1037" alt="Screenshot 2026-09-12 192000" src="https://github.com/user-attachments/assets/96b1c32b-1aa6-4aeb-805f-2b8a9f05e832" />
<img width="1917" height="1002" alt="Screenshot 2026-09-13 083533" src="https://github.com/user-attachments/assets/58db4fbe-6521-4075-848d-9f2b83fea6dc" />

---

# 15. 🔁 Complete Day 8 Flow

```text
                    CMOS DEVICE STUDY
                           │
                           ▼
                    CMOS INVERTER
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
        DC CHARACTERIZATION       TRANSIENT ANALYSIS
              │                         │
              ▼                         ▼
             VM                  RISE / FALL TIME
                                        │
                                        ▼
                                  CELL DELAY
              │
              └────────────┬────────────┘
                           │
                           ▼
                   SKY130 CELL LAYOUT
                           │
                           ▼
                         MAGIC
                           │
                           ▼
                  LAYOUT EXTRACTION
                           │
                           ▼
                     SPICE NETLIST
                           │
                           ▼
                    MODEL CORRECTION
                           │
                           ▼
                       NGSPICE
                           │
                           ▼
                  WAVEFORM MEASUREMENT
                           │
                           ▼
                          DRC
                           │
                           ▼
                  RULE / GEOMETRY CHECK
```

---

# 16. 🛠️ Tools & Technologies

| Tool / Technology | Role                                  |
| ----------------- | ------------------------------------- |
| **Sky130**        | CMOS technology / PDK                 |
| **Magic**         | Layout editing and extraction         |
| **ngspice**       | SPICE circuit simulation              |
| **OpenLane**      | RTL-to-GDS implementation environment |
| **Linux**         | VLSI development environment          |
| **Git**           | Version control                       |
| **GitHub**        | Project documentation                 |

---

# 17. 📌 Key Learning Points

| Concept            | What I Learned                                    |
| ------------------ | ------------------------------------------------- |
| CMOS Inverter      | Complementary PMOS/NMOS switching                 |
| VTC                | Relationship between input and output voltage     |
| VM                 | Inverter switching threshold                      |
| SPICE              | Transistor-level circuit simulation               |
| Transient Analysis | Time-domain switching behavior                    |
| Rise/Fall Time     | Output transition speed                           |
| Cell Delay         | Input-to-output propagation delay                 |
| 16-Mask Process    | Major CMOS fabrication stages                     |
| LOCOS              | Isolation technique                               |
| LDD                | Electric-field reduction near drain               |
| Spacer             | Controls heavy source/drain implant location      |
| Magic              | Physical layout inspection and extraction         |
| Extraction         | Converts layout geometry into circuit information |
| BSIM4              | Transistor compact modeling                       |
| Sky130 Models      | Technology-specific transistor behavior           |
| DRC                | Manufacturing-rule verification                   |
| VIA2               | Inter-metal connection structure                  |

---

# 🚀 Conclusion

Day 8 provided a transistor-level view of the VLSI design process.

The study began with the electrical behavior of a CMOS inverter and moved
through switching-threshold analysis, transient timing, semiconductor
fabrication, physical layout, SPICE extraction, technology-specific device
models, and DRC investigation.

The most important practical connection was:

```text
PHYSICAL GEOMETRY
       ↓
TRANSISTOR EXTRACTION
       ↓
SPICE NETLIST
       ↓
DEVICE MODELS
       ↓
TRANSIENT SIMULATION
       ↓
TIMING RESULTS
```

This demonstrates how a physical standard-cell layout can be converted back
into an electrical model and analyzed using SPICE.

The work also shows the relationship between:

```text
Fabrication Rules
       ↓
Physical Layout
       ↓
Parasitics
       ↓
Circuit Behavior
       ↓
Timing
```

These concepts form an important foundation for further work in:

* RTL Design
* Standard Cell Design
* ASIC Physical Design
* CMOS Circuit Design
* Cell Characterization
* SPICE Simulation
* Layout Engineering
* DRC / LVS Verification

---

<p align="center">
  <b>Sky130 CMOS Inverter — Device to Layout</b>
</p>

<p align="center">
  Fabrication → Characterization → Extraction → Simulation → Verification
</p>
