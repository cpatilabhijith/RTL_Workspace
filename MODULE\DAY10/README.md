# Day 10 — Final Steps for RTL2GDS Using TritonRoute and OpenSTA

**Post-CTS to GDSII signoff** — power distribution, detailed routing, parasitic extraction, and final static timing analysis — using an **OpenLANE + SKY130 + TritonRoute/OpenSTA** flow.

This module documents the remaining physical design stages after clock tree synthesis: PDN generation, global and detailed routing, SPEF parasitic extraction, post-route STA signoff, and GDSII streamout.

| | |
|---|---|
| **Technology** | SKY130 |
| **Toolchain** | OpenLANE · TritonRoute · FastRoute · OpenSTA · Magic |
| **Analysis Type** | Post-Route Static Timing Analysis (STA) |
| **Scope** | PDN, Routing, Parasitic Extraction, Timing Signoff, GDSII |

---

## Table of Contents

1. [Overview of the Post-CTS to GDSII Flow](#1-overview-of-the-post-cts-to-gdsii-flow)
2. [Power Distribution Network (PDN)](#2-power-distribution-network-pdn)
3. [Detailed Routing with TritonRoute](#3-detailed-routing-with-tritonroute)
4. [Parasitic Extraction (SPEF)](#4-parasitic-extraction-spef)
5. [Post-Route Static Timing Analysis (OpenSTA)](#5-post-route-static-timing-analysis-opensta)
6. [GDSII Streamout and DRC Verification](#6-gdsii-streamout-and-drc-verification)
7. [Key Concepts Summary](#7-key-concepts-summary)
8. [Overall RTL2GDS Flow Recap](#8-overall-rtl2gds-flow-recap)
9. [Learning Outcome](#9-learning-outcome)

---

# 1. Overview of the Post-CTS to GDSII Flow

Once Clock Tree Synthesis (CTS) is complete and timing closure is verified on propagated clocks, the remaining flow steps carry the design from a placed-and-clocked netlist to a fabrication-ready layout:

| Stage | Purpose |
|---|---|
| **Power Distribution Network (PDN)** | Synthesizes the power (V_DD) and ground (V_SS) rails and straps |
| **Global & Detailed Routing** | Uses FastRoute (global) and TritonRoute (detailed) to physically connect all signal nets per design rules |
| **SPEF Extraction** | Extracts parasitic resistance and capacitance from the final routed interconnects |
| **Post-Route Timing Analysis** | Runs final STA with OpenSTA on real parasitic data |
| **GDSII Streamout** | Converts the final routed DEF/LEF layout into GDSII using Magic or KLayout |

Each stage moves the design closer to silicon accuracy: placement and CTS estimates are progressively replaced by real geometry, real parasitics, and real design-rule-clean routing.

---

# 2. Power Distribution Network (PDN)

In a standard OpenLANE run, the PDN step creates the power grid — straps and rings — after CTS, so that clock buffers and logic cells can tap directly into robust rails.

### Commands to Run PDN

Inside the OpenLANE interactive shell (`./flow.tcl -interactive`):

```tcl
# Ensure your design is loaded up to CTS
# If resuming from a previous run:
package require openlane 0.9
prep -design picorv32a -tag <run_tag>

# Generate the Power Distribution Network
gen_pdn
```

### What `gen_pdn` Does

- Reads `pdn.cfg` (or the technology configuration rules).
- Creates standard cell rails on `met1`.
- Creates power and ground straps on higher metal layers (typically `met4` and `met5` in SKY130).
- Adds vias connecting `met1` rails to the higher-layer straps.

**Output:** Updates the design DEF file in `runs/<run_tag>/tmp/floorplan/` or `runs/<run_tag>/results/cts/`.

---

# 3. Detailed Routing with TritonRoute

Routing is split into two major phases:

- **Global Routing** — handled by FastRoute / Global Route, which breaks the routing problem into a grid of global routing cells (Gcells) and generates route guides.
- **Detailed Routing** — handled by TritonRoute, which honors the pre-processed routing guides and places actual metal wires and vias following SKY130 DRC design rules.

### TritonRoute Key Features

- **Honors pre-processed route guides:** reads guides created during global routing and ensures wires do not deviate outside these designated areas.
- **Inter-guide connectivity and layer assignment:** handles transitions across multiple metal layers using optimal via insertion.
- **Intra- and inter-layer DRC handling:** respects minimum width, spacing, end-of-line spacing, and enclosure rules to avoid design rule violations.
- **Routing topology algorithm:** uses integer linear programming (ILP) and path-finding algorithms to minimize wirelength and via count.

### Commands to Run Routing

```tcl
# Run the complete routing stage (Global Route + Detailed Route)
run_routing
```

### Key Routing Environment Variables

You can customize routing behavior in the interactive flow prior to calling `run_routing`:

```tcl
# Routing strategy: 0 (default/fast), 1, 2, or 3 (more effort/higher DRC clean rate)
set ::env(ROUTING_STRATEGY) 0

# Set routing optimization / DRC violation fixing effort
set ::env(GLB_RT_ADJUSTMENT) 0.15

# Set TritonRoute processing cores
set ::env(ROUTING_CORES) 4
```

### Outputs of the Routing Stage

| Output | Path |
|---|---|
| Routed DEF | `runs/<run_tag>/results/routing/<design_name>.def` |
| Route Guides | `runs/<run_tag>/tmp/routing/fastroute.guide` |
| DRC Report | `runs/<run_tag>/reports/routing/tritonRoute.drc` |

---

# 4. Parasitic Extraction (SPEF)

Post-routing, wire resistance (R) and capacitance (C) are no longer estimates based on wire-length models — they are extracted from actual polygon geometry.

OpenLANE uses **SPEF-Extractor** to parse the routed DEF and generate a standard parasitic format (`.spef`) file:

```bash
# SPEF extraction is executed automatically by run_routing,
# or can be run manually using OpenLANE scripts:
python3 /openlane/scripts/spef_extractor/main.py \
  --def_file runs/<run_tag>/results/routing/<design_name>.def \
  --lef_file runs/<run_tag>/tmp/merged.lef \
  --wire_model <wire_model>
```

**Output:** `runs/<run_tag>/results/routing/<design_name>.spef`

---

# 5. Post-Route Static Timing Analysis (OpenSTA)

Final timing signoff is performed using OpenSTA by coupling the extracted `.spef` file with the synthesized netlist and SDC constraints.

### Launching OpenSTA with Parasitics

Start OpenROAD/OpenSTA from a terminal or inside OpenLANE:

```bash
openroad
```

### OpenSTA Script for Post-Route Signoff (`post_route_sta.tcl`)

```tcl
# 1. Read LEF files
read_lef /openlane/designs/<design_name>/runs/<run_tag>/tmp/merged.lef

# 2. Read Liberty files (corners)
read_liberty -min $::env(LIB_FASTEST)
read_liberty -max $::env(LIB_SLOWEST)

# 3. Read routed Verilog netlist
read_verilog /openlane/designs/<design_name>/runs/<run_tag>/results/routing/<design_name>.resized.v

# 4. Link design
link_design <top_module_name>

# 5. Read SDC constraints
read_sdc /openlane/designs/<design_name>/src/<design_name>.sdc

# 6. Set clock propagation (real clock network)
set_propagated_clock [all_clocks]

# 7. Read extracted parasitics (SPEF)
read_spef /openlane/designs/<design_name>/runs/<run_tag>/results/routing/<design_name>.spef

# 8. Report timing checks
report_checks -path_delay min_max -fields {slew cap input nets fanout} -format full_clock_expanded -digits 4
report_worst_slack -min
report_worst_slack -max
report_tns
report_wns
```

This is the true signoff STA run: real clock latency, real parasitics, both min (hold) and max (setup) corners — the last checkpoint before layout is committed to silicon.

---

# 6. GDSII Streamout and DRC Verification

The final step is generating the binary GDSII file used for chip fabrication.

```tcl
# Generate GDSII inside the OpenLANE interactive shell
run_magic
```

### Final Files to Verify

| Artifact | Path |
|---|---|
| GDSII file | `runs/<run_tag>/results/magic/<design_name>.gds` |
| Magic DRC Report | `runs/<run_tag>/reports/magic/magic.drc` |
| LVS Report | `runs/<run_tag>/reports/netgen/lvs.rpt` |

A clean GDSII is only signoff-ready once the DRC report shows zero violations and the LVS report confirms layout-vs-schematic match.

---

# 7. Key Concepts Summary

| Concept | Description |
|---|---|
| **PDN** | Power grid (rails + straps + vias) that delivers V_DD/V_SS to every cell |
| **Global Routing** | Coarse-grained routing that assigns nets to Gcells and produces route guides |
| **Detailed Routing** | Fine-grained, DRC-clean wire and via placement following the route guides |
| **SPEF** | Standard Parasitic Exchange Format — real extracted R/C of routed interconnect |
| **Post-Route STA** | Timing signoff using real parasitics and propagated clocks, at min/max corners |
| **WNS / TNS** | Worst Negative Slack / Total Negative Slack — key signoff timing metrics |
| **GDSII** | Binary layout format used for tape-out and fabrication |
| **DRC / LVS** | Design Rule Check and Layout-vs-Schematic — final physical/functional verification |

---

# 8. Overall RTL2GDS Flow Recap

```
Synthesis → Floorplan → Placement → CTS → PDN → Global Route → Detailed Route
    → SPEF Extraction → Post-Route STA (Signoff) → DRC/LVS → GDSII Streamout
```

Each stage refines the design's physical realism — from an abstract netlist to a fully verified, fabrication-ready layout — with timing signoff acting as the final gate before tape-out.

---

# 9. Learning Outcome

After completing this module, you should be able to:

1. Explain the purpose of each stage from post-CTS to GDSII.
2. Generate a Power Distribution Network and describe what `gen_pdn` produces.
3. Distinguish global routing from detailed routing and describe TritonRoute's role.
4. Configure key routing environment variables and interpret DRC reports.
5. Explain why parasitic extraction (SPEF) is necessary for accurate signoff timing.
6. Write and run a post-route OpenSTA script using real parasitics and propagated clocks.
7. Interpret WNS, TNS, and worst-slack reports for min/max timing corners.
8. Generate GDSII output and verify DRC/LVS reports before tape-out.

---

## Notes

Command syntax and environment-variable names can differ between OpenLANE/OpenROAD releases. Verify all commands against the version installed in your environment before executing them.
