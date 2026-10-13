# VSDBabySoC — Static Timing Analysis (STA) Across 16 PVT Corners

## 1. Introduction

This project documents Static Timing Analysis (STA) of the VSDBabySoC design using OpenSTA/OpenROAD and the SKY130 standard-cell library.

The objective is to evaluate the timing behavior of the synthesized and physically implemented design under different process, voltage, and temperature (PVT) conditions.

The analysis includes setup timing, hold timing, Worst Negative Slack (WNS), Total Negative Slack (TNS), and timing verification across 16 PVT corners.

**Project highlights:**
- Design: VSDBabySoC (RISC-V-based System-on-Chip)
- Technology: SKY130
- Standard-cell library: SKY130 HD
- Timing tools: OpenSTA and OpenROAD
- Analysis: Static Timing Analysis
- Corners analyzed: 16 PVT corners
- Timing checks: Setup and hold
- Implementation stage: Post-route STA

## 2. About VSDBabySoC

VSDBabySoC is a small mixed-signal System-on-Chip that integrates a RISC-V-based processor with analog IP blocks.

The main design components include:

- **RVMYTH:** RISC-V-based processor core.
- **PLL:** Phase-Locked Loop used to generate the system clock.
- **DAC:** Digital-to-Analog Converter for analog output.

The purpose of STA is to verify that data signals propagate through the digital logic within the required timing limits under different operating conditions.

## 3. Objective of Static Timing Analysis

Static Timing Analysis verifies the timing behavior of a digital circuit without requiring simulation of every possible input combination.

The primary objectives are:

1. Verify setup timing across supported PVT corners.
2. Verify hold timing across supported PVT corners.
3. Identify critical timing paths.
4. Calculate WNS and TNS.
5. Evaluate the effect of process, voltage, and temperature variations.
6. Identify timing violations requiring optimization.
7. Compare timing behavior between operating corners.

## 4. Tools and Technology

| Component | Description |
|---|---|
| Design | VSDBabySoC |
| PDK | SKY130 |
| Standard-cell library | `sky130_fd_sc_hd` |
| Synthesis | Yosys |
| Technology mapping | ABC |
| Physical design | OpenLANE / OpenROAD |
| Static timing analysis | OpenSTA |
| Timing constraints | SDC |
| Physical parasitics | Routed interconnect estimates or extracted parasitics, depending on the analysis stage |

The standard-cell Liberty files provide timing information for cells, including propagation delay, setup time, hold time, input capacitance, and output transition characteristics.

## 5. Timing Analysis Flow

The overall design flow is:

```text
VSDBabySoC RTL
      |
      v
   Synthesis
      |
      v
Gate-Level Netlist
      |
      v
  Floorplanning
      |
      v
   Placement
      |
      v
Clock Tree Synthesis
      |
      v
     Routing
      |
      v
Post-Route Timing Analysis
      |
      v
Analyze 16 PVT Corners
      |
      v
Setup and Hold Verification
      |
      v
WNS / TNS Evaluation
      |
      v
Identify Critical Paths
      |
      v
Timing Optimization
```

Post-route STA evaluates timing using the physical implementation and the parasitic information available to the analysis. The exact accuracy depends on whether extracted or estimated parasitics are used.

## 6. PVT Corner Analysis

PVT stands for Process, Voltage, and Temperature.

Different combinations of these parameters affect standard-cell delays and timing behavior.

### 6.1 Process

- **FF (Fast-Fast):** Fast process corner.
- **SS (Slow-Slow):** Slow process corner.
- **TT (Typical-Typical):** Typical process corner.

### 6.2 Voltage

The voltage suffix identifies the operating voltage in the corner name.

For example:

- `1v28` represents 1.28 V.
- `1v65` represents 1.65 V.
- `1v80` represents 1.80 V.
- `1v95` represents 1.95 V.

### 6.3 Temperature

The temperature suffix identifies the operating temperature.

For example:

- `n40C` represents −40 °C.
- `025C` represents 25 °C.
- `100C` represents 100 °C.

The combined corner name identifies the process, temperature, and voltage conditions used during timing analysis.

## 7. The 16 PVT Corners

The analysis covers the following 16 corners.

| No. | PVT Corner | Process | Temperature | Voltage |
|---:|---|---|---:|---:|
| 1 | `FF_100C_1v65` | FF | 100 °C | 1.65 V |
| 2 | `FF_100C_1v95` | FF | 100 °C | 1.95 V |
| 3 | `FF_n40C_1v56` | FF | −40 °C | 1.56 V |
| 4 | `FF_n40C_1v65` | FF | −40 °C | 1.65 V |
| 5 | `FF_n40C_1v76` | FF | −40 °C | 1.76 V |
| 6 | `FF_n40C_1v95` | FF | −40 °C | 1.95 V |
| 7 | `SS_100C_1v40` | SS | 100 °C | 1.40 V |
| 8 | `SS_100C_1v60` | SS | 100 °C | 1.60 V |
| 9 | `SS_n40C_1v28` | SS | −40 °C | 1.28 V |
| 10 | `SS_n40C_1v35` | SS | −40 °C | 1.35 V |
| 11 | `SS_n40C_1v40` | SS | −40 °C | 1.40 V |
| 12 | `SS_n40C_1v44` | SS | −40 °C | 1.44 V |
| 13 | `SS_n40C_1v60` | SS | −40 °C | 1.60 V |
| 14 | `SS_n40C_1v76` | SS | −40 °C | 1.76 V |
| 15 | `TT_025C_1v80` | TT | 25 °C | 1.80 V |
| 16 | `TT_100C_1v80` | TT | 100 °C | 1.80 V |

## 8. Timing Equations

### 8.1 Setup Slack

Setup timing checks whether data arrives early enough before the capture clock edge.

```text
Setup Slack = Required Arrival Time - Data Arrival Time
```

For a simplified single-cycle path:

```text
Setup Slack =
Clock Period
- Setup Time
- Clock Uncertainty
- Data Path Delay
```

The actual STA calculation also accounts for launch and capture clock arrival times, clock skew, and other applicable timing constraints.

**Interpretation:**
- Positive setup slack: Setup requirement is satisfied.
- Zero setup slack: The path meets the timing limit exactly.
- Negative setup slack: Setup timing violation.

### 8.2 Hold Slack

Hold timing checks whether data remains stable for the required time after the capture clock edge.

```text
Hold Slack = Data Arrival Time - Required Arrival Time
```

The exact calculation depends on minimum data-path delay, clock arrival times, hold requirements, and timing uncertainty.

**Interpretation:**
- Positive hold slack: Hold requirement is satisfied.
- Zero hold slack: The path meets the hold limit exactly.
- Negative hold slack: Hold timing violation.

### 8.3 Worst Negative Slack (WNS)

WNS identifies the worst slack among the analyzed paths.

```text
WNS = Minimum Slack
```

A negative setup WNS indicates that at least one analyzed setup path violates its timing requirement.

### 8.4 Total Negative Slack (TNS)

TNS represents the sum of negative slack values across violating paths.

```text
TNS = Sum of Negative Slack Values
```

A negative TNS indicates accumulated timing violations. A larger negative magnitude generally indicates a greater aggregate timing problem.

## 9. Post-Route STA Results

The post-route timing report shows that the design has different timing behavior across the analyzed PVT corners.

The reported results indicate:

- All FF corners satisfy setup timing.
- Both TT corners satisfy setup timing.
- `SS_100C_1v60` and `SS_n40C_1v76` satisfy setup timing.
- Six SS corners show setup timing violations.
- Hold timing is satisfied at all 16 analyzed corners.

### 9.1 Setup Timing Results for SS Corners

The following values are taken from the reported post-route STA summary.

| PVT Corner | Setup WNS (ns) | Setup TNS (ns) | Status |
|---|---:|---:|---|
| `SS_n40C_1v28` | -29.32 | -14575.39 | FAIL |
| `SS_n40C_1v35` | -16.45 | -6570.44 | FAIL |
| `SS_n40C_1v40` | -10.99 | -3336.45 | FAIL |
| `SS_n40C_1v44` | -7.72 | -1909.84 | FAIL |
| `SS_100C_1v40` | -3.51 | -326.35 | FAIL |
| `SS_n40C_1v60` | -0.30 | -1.19 | FAIL |
| `SS_100C_1v60` | Not listed here | Not listed here | PASS |
| `SS_n40C_1v76` | Not listed here | Not listed here | PASS |

The exact WNS and TNS values for the passing corners should be copied from their corresponding original report rows.

### 9.2 Worst-Case Setup Corner

The worst setup timing result occurs at:

```text
Corner: SS_n40C_1v28

Setup WNS = -29.32 ns
Setup TNS = -14575.39 ns
```

This corner has the most negative setup WNS among the reported corners and therefore represents the most severe setup timing problem in the provided results.

The large negative TNS indicates that many timing paths contribute to the overall setup violation.

### 9.3 Setup Timing Observations

The results show that setup timing is more challenging in several slow-process corners.

The worst reported violation occurs at `SS_n40C_1v28`, while `SS_n40C_1v60` has a smaller negative setup WNS of −0.30 ns.

The difference between these results demonstrates the importance of analyzing the design under multiple operating conditions instead of relying on a single typical corner.

## 10. Hold Timing Results

The reported post-route STA summary indicates that hold timing passes across all 16 corners.

```text
Hold timing violations = 0
Hold WNS               = 0.00 ns
Hold TNS               = 0.00 ns
```

The report also shows positive hold worst slack at every corner.

A hold WNS of 0.00 ns and TNS of 0.00 ns indicate that there are no negative hold-slack violations in the reported analysis.

**Note:** A zero WNS in this summary does not necessarily mean that a path has exactly zero slack. STA reports commonly use zero as the WNS floor when there are no negative-slack paths.

## 11. Summary of Timing Results

| Parameter | Result |
|---|---|
| Total PVT corners | 16 |
| FF setup status | PASS |
| TT setup status | PASS |
| SS setup status | Six corners fail; two pass |
| Hold timing | PASS at all analyzed corners |
| Hold violations | 0 |
| Worst setup corner | `SS_n40C_1v28` |
| Worst setup WNS | -29.32 ns |
| Worst setup TNS | -14575.39 ns |
| Overall setup timing | Not closed across all corners |
| Overall hold timing | Satisfied in the reported analysis |

## 12. Analysis of the Results

The STA results identify setup timing as the primary issue in the current design.

Although the FF and TT corners pass setup timing, six SS corners show negative setup slack.

The most critical result is at `SS_n40C_1v28`, where the setup WNS is −29.32 ns and the setup TNS is −14575.39 ns.

This indicates that timing optimization is required before claiming setup timing closure across all corners.

Possible optimization steps include:

1. Inspect the worst setup paths using detailed timing reports.
2. Identify the logic levels and cells contributing most to the critical path delay.
3. Review the clock period and all timing constraints for correctness.
4. Check whether the intended clock source and clock period are correctly defined.
5. Review high-fanout nets and interconnect delays.
6. Evaluate cell sizing and buffering where appropriate.
7. Re-run synthesis or physical implementation after making changes.
8. Re-run setup and hold analysis across all 16 corners.
9. Verify that any setup improvement does not introduce hold violations.

Because the worst violation is substantial, verify the timing constraints and clock definition before attempting aggressive cell-level optimization.

## 13. OpenSTA/OpenROAD Timing Analysis

A typical post-route analysis requires the correct physical database, timing libraries, design netlist, timing constraints, and parasitic information.

A simplified OpenROAD Tcl sequence is shown below. File paths and library selection must match the actual run.

```tcl
# Load the physical database
read_db <design_database.odb>

# Load the timing libraries for the selected corner
read_liberty <corner_liberty_file.lib>

# Load the design netlist if required by the database workflow
read_verilog <post_route_netlist.v>

# Link the design
link_design vsdbabysoc

# Load timing constraints
read_sdc <design_constraints.sdc>

# Load extracted parasitics when available
read_spef <extracted_parasitics.spef>

# Enable propagated clock analysis
set_propagated_clock [all_clocks]

# Report setup and hold paths
report_checks -path_delay min_max \
    -format full_clock_expanded -digits 4

# Report clock skew
report_clock_skew -digits 4
```

This is a template rather than a guaranteed drop-in script. The correct database, netlist, parasitic file, and library corner must be loaded consistently. Some flows already contain the netlist and parasitic information in the database or load them through other commands.

## 14. Timing Report Interpretation

When examining an STA report, focus on the following fields:

**Startpoint:** The launch point of the timing path.

**Endpoint:** The capture point or endpoint of the timing path.

**Path Group:** The clock or timing group associated with the path.

**Path Type:** Indicates whether the report is evaluating maximum-delay or minimum-delay timing.

**Data Arrival Time:** The time at which data reaches the endpoint.

**Data Required Time:** The latest or earliest permitted arrival time, depending on the check.

**Slack:** The difference between required time and arrival time, with the sign convention depending on the timing check.

For setup analysis, prioritize paths with the most negative setup slack. For hold analysis, inspect paths with negative hold slack if any violations are reported.

## 15. Conclusion

The VSDBabySoC post-route STA analysis evaluates timing across 16 PVT corners using the SKY130 standard-cell library.

The results show that hold timing passes at all analyzed corners, while setup timing fails at six SS corners.

The worst reported setup result is at `SS_n40C_1v28`, with a WNS of −29.32 ns and a TNS of −14575.39 ns.

These results demonstrate why multi-corner STA is essential for physical design verification. Passing setup timing at FF and TT corners alone is not sufficient to establish timing closure across all operating conditions.

The next step is to inspect the worst critical paths, verify the timing constraints and clock definition, optimize the data paths, and rerun the full multi-corner STA flow.

**Final status:** Hold timing is satisfied in the reported analysis, but setup timing requires further investigation and optimization before the design can be considered setup-clean across all 16 PVT corners.

## 16. References

- [VSDBabySoC project directory](https://github.com/cpatilabhijith/RTL_Workspace/tree/main/VSDBabySoC)
- [OpenROAD Project](https://github.com/The-OpenROAD-Project/OpenROAD)
- [OpenSTA](https://github.com/The-OpenROAD-Project/OpenSTA)
- [SKY130 PDK](https://github.com/google/skywater-pdk)
