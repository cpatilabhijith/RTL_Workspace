# Sequence Detector – RTL to Gate-Level Simulation

## Overview

This repository documents the complete RTL-to-Gate-Level Simulation flow of a Verilog sequence detector for the target sequence **0111101**.

The same testbench and input sequence are used for both RTL simulation and Gate-Level Simulation (GLS). The final comparison shows that the synthesized implementation preserves the functional behavior of the RTL design, with **7 detections** in both simulations and the first detection occurring at approximately **154 ns**.

## Design Flow

```text
RTL
 ↓
Pre-synth Simulation
 ↓
GTKWave
 ↓
Yosys Synthesis
 ↓
vsdbabysoc.synth.v / Synthesized Netlist
 ↓
Post-synthesis Gate-Level Simulation (GLS)
 ↓
GTKWave
 ↓
Same Functional Behavior
```

---

## 1. RTL Design

The sequence detector is implemented as a finite state machine using a 3-bit state register.

- **Target sequence:** `0111101`
- **Number of states:** 7
- **State width:** 3 bits
- **Input:** `din`
- **Output:** `detected`
- **Reset:** synchronous, active high
- **Detection:** `detected` is asserted when the complete target sequence is recognized.

### RTL Code

```verilog
`timescale 1ns/1ps

module sequence_detector (
    input  wire clk,
    input  wire reset,
    input  wire din,
    output reg  detected
);

    localparam integer STATE_W = 3;
    localparam integer NUM_STATES = 7;
    // Target sequence: 0111101

    reg [STATE_W-1:0] state;
    reg [STATE_W-1:0] next_state;
    reg next_detected;

    always @(*) begin
        next_state = 'd0;
        next_detected = 1'b0;

        case (state)
            0: begin
                if (din == 1'b0) begin
                    next_state = 1;
                    next_detected = 1'b0;
                end else begin
                    next_state = 0;
                    next_detected = 1'b0;
                end
            end

            1: begin
                if (din == 1'b0) begin
                    next_state = 1;
                    next_detected = 1'b0;
                end else begin
                    next_state = 2;
                    next_detected = 1'b0;
                end
            end

            2: begin
                if (din == 1'b0) begin
                    next_state = 1;
                    next_detected = 1'b0;
                end else begin
                    next_state = 3;
                    next_detected = 1'b0;
                end
            end

            3: begin
                if (din == 1'b0) begin
                    next_state = 1;
                    next_detected = 1'b0;
                end else begin
                    next_state = 4;
                    next_detected = 1'b0;
                end
            end

            4: begin
                if (din == 1'b0) begin
                    next_state = 1;
                    next_detected = 1'b0;
                end else begin
                    next_state = 5;
                    next_detected = 1'b0;
                end
            end

            5: begin
                if (din == 1'b0) begin
                    next_state = 6;
                    next_detected = 1'b0;
                end else begin
                    next_state = 0;
                    next_detected = 1'b0;
                end
            end

            6: begin
                if (din == 1'b0) begin
                    next_state = 1;
                    next_detected = 1'b0;
                end else begin
                    next_state = 2;
                    next_detected = 1'b1;
                end
            end

            default: begin
                next_state = 'd0;
                next_detected = 1'b0;
            end
        endcase
    end

    always @(posedge clk) begin
        if (reset) begin
            state <= 'd0;
            detected <= 1'b0;
        end else begin
            state <= next_state;
            detected <= next_detected;
        end
    end

endmodule
```

---

## 2. Testbench

The testbench generates the clock, applies reset, drives the input sequence bit-by-bit, records the detection pulses, and prints the final detection count.

### Important Testbench Parameters

| Parameter | Value |
|---|---|
| Clock | `#7 clk = ~clk` |
| Target sequence | `0111101` |
| Reset | Active high |
| Testbench style | Same input sequence for RTL and GLS |
| Assessment instance | `24eg104c36` |
| Detection count observed | `7` |

### Testbench Code

```verilog
`timescale 1ns/1ps

module tb;

    reg clk = 1'b0;
    reg reset = 1'b1;
    reg din = 1'b0;
    wire detected;

    sequence_detector dut (
        .clk(clk),
        .reset(reset),
        .din(din),
        .detected(detected)
    );

    always #7 clk = ~clk;

    // Assessment instance: 24eg104c36

    task drive_bit(input reg b);
        begin
            @(negedge clk);
            din = b;
            @(posedge clk);
            #1;
            $display("TIME=%0t NS DIN=%b DETECTED=%b", $time, din, detected);
        end
    endtask

    integer detection_count = 0;

    always @(negedge clk) begin
        if (!reset && detected)
            detection_count = detection_count + 1;
    end

    initial begin
        $dumpfile("dump.vcd");
        $dumpvars(0, tb);

        // Initial reset.
        reset = 1'b1;
        repeat (2) @(posedge clk);
        @(negedge clk);
        reset = 1'b0;

        // Test sequence
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b1);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b0);
        drive_bit(1'b0);

        // Final reset.
        @(negedge clk);
        reset = 1'b1;
        repeat (2) @(posedge clk);

        #1;
        $display("FINAL_DETECTION_COUNT=%0d", detection_count);
        $finish;
    end

endmodule
```

---

## 3. Pre-Synthesis / RTL Simulation

The RTL testbench was simulated and the generated VCD waveform was viewed in GTKWave.

The waveform shows:

- `clk` running continuously.
- `reset` asserted initially and again at the end.
- `din` changing according to the testbench sequence.
- `detected` pulsing when the target sequence `0111101` is recognized.
- `detection_count` reaching **7**.

### RTL GTKWave Evidence

<img width="1920" height="1080" alt="dumpvcd" src="https://github.com/user-attachments/assets/5aaef3a8-3af5-4359-85ab-ddb005e48f15" />


The uploaded RTL waveform shows the signals `clk`, `reset`, `din`, `detection_count`, and `detected`. The detection pulses correspond to successful sequence recognition.

---

## 4. Yosys Synthesis

Yosys was used to read and synthesize the RTL into a gate-level representation.

The synthesis flow included design hierarchy analysis, optimization, statistics generation, and a final consistency check.

### Yosys Check Result

```text
Checking module sequence_detector...
Found and reported 0 problems.
```

### Synthesized Design Statistics

| Item | Count |
|---|---:|
| Wires | 23 |
| Wire bits | 29 |
| Public wires | 5 |
| Public wire bits | 11 |
| Ports | 4 |
| Port bits | 4 |
| Memories | 0 |
| Memory bits | 0 |
| Processes | 0 |
| Cells | 26 |

### Cell Breakdown

| Cell | Count |
|---|---:|
| `$ _ANDNOT_` | 6 |
| `$ _AND_` | 3 |
| `$ _DFF_P_` | 7 |
| `$ _NOR_` | 3 |
| `$ _ORNOT_` | 2 |
| `$ _OR_` | 4 |
| `$ _SDFF_PP0_` | 1 |
| **Total** | **26** |

> The cell names above are taken from the Yosys statistics shown in the supplied synthesis output screenshot.

### Yosys Statistics Evidence

<img width="1920" height="1080" alt="outputstats" src="https://github.com/user-attachments/assets/8536b4a0-7d04-434b-8f17-1332fa17ae1e" />


The supplied screenshot shows the synthesized `sequence_detector` module and confirms that Yosys completed the check with **0 reported problems**.

---

## 5. Synthesized Netlist / Logic Representation

After synthesis, the RTL is represented using flip-flops and combinational logic cells.

The generated logic diagram shows:

- State and detection registers.
- AND / OR / NOR / ANDNOT / ORNOT logic.
- Clock connectivity.
- Combinational logic implementing the next-state and detection equations.

### Synthesized Logic Diagram

![Synthesized logic graph](Screenshot 2026-08-29 112307.png)

This graph represents the synthesized gate-level structure produced by Yosys.

---

## 6. Post-Synthesis Gate-Level Simulation (GLS)

The synthesized netlist was simulated with the same testbench and input sequence used for RTL simulation.

The GLS waveform contains the corresponding clock, reset, input, detection count, and detected signals.

### GLS GTKWave Evidence

<img width="1920" height="1080" alt="gls_dump" src="https://github.com/user-attachments/assets/b6857f15-7c8d-4398-94cc-ae257cf75314" />


The GLS waveform shows the same logical detection behavior as the RTL waveform.

---

## 7. RTL vs GLS Comparison

The comparison is based on the same testbench and the same input sequence.

| Parameter | RTL Simulation | GLS |
|---|---:|---:|
| Target sequence | `0111101` | `0111101` |
| First detection | ~154 ns | ~154 ns |
| Total detections | 7 | 7 |
| Final detection count | 7 | 7 |

According to the supplied comparison report, both simulations successfully detect the target sequence `0111101`, with the detection count reaching **7** in both cases.

---

## 8. Functional Verification

The functional behavior can be summarized as:

```text
Input sequence
      ↓
Sequence detector FSM
      ↓
Target = 0111101
      ↓
Detection pulses
      ↓
7 successful detections
      ↓
RTL result = GLS result
```

The GTKWave results show matching logical behavior between the RTL and gate-level simulations. The synthesized implementation therefore preserves the intended sequence-detection functionality for the given testbench.

---

## 9. Final Conclusion

The synthesized implementation **preserves the functional behavior of the RTL design for the given testbench**. Both RTL and Gate-Level Simulation detect the target sequence `0111101` and produce **seven detection pulses**, as observed in the GTKWave waveforms. The GLS implementation can introduce small gate-level propagation delays, but the logical detection behavior remains unchanged.

---

## 10. Repository Evidence

Use the following screenshots as the supporting evidence for the flow documented in this README:

1. **RTL GTKWave:** 

<img width="1920" height="1080" alt="rtl_gtkwave" src="https://github.com/user-attachments/assets/02ab9d82-7eec-4597-a891-a4c3cfd1b038" />

2. **GLS GTKWave:** 

<img width="1920" height="1080" alt="gls_dump" src="https://github.com/user-attachments/assets/b6857f15-7c8d-4398-94cc-ae257cf75314" />

3. **Yosys synthesis statistics:** 

<img width="1920" height="1080" alt="yosys_statistics" src="https://github.com/user-attachments/assets/6019d82a-37d4-4179-91df-07a44fc872f3" />

4. **Yosys generated logic diagram:**

<img width="888" height="815" alt="synthesized_logic" src="https://github.com/user-attachments/assets/635f85f3-40ff-4abd-be87-05e6d20d5328" />

5. **Yosys synthesis terminal/output:** 

<img width="1920" height="1080" alt="yosys_terminal" src="https://github.com/user-attachments/assets/8c30cef8-13e1-466a-b6e1-929dc847d06d" />


The final RTL-vs-GLS conclusion is also documented in the supplied comparison report.

---

## 11. Result

```text
Target sequence        : 0111101
RTL detections         : 7
GLS detections         : 7
RTL first detection    : ~154 ns
GLS first detection    : ~154 ns
Functional match       : YES
```

### Final Inference

**RTL → Synthesis → GLS preserves the same functional behavior for this testbench.**
