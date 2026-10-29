# 4×4 Reversible Multiplier with Balanced Design Parameters

![Verilog](https://img.shields.io/badge/HDL-Verilog-blue)
![FPGA](https://img.shields.io/badge/FPGA-Nexys%20A7%20(Artix--7)-orange)
![Tool](https://img.shields.io/badge/Tool-Xilinx%20Vivado-red)
![IEEE](https://img.shields.io/badge/Published-IEEE%20IEMECON%202025-00629B)

A Verilog implementation of an optimized **4×4 reversible multiplier** built from **Peres** and **TSG** reversible logic gates. The design was simulated in **Xilinx Vivado** and validated in hardware on a **Nexys A7 (Artix-7, xc7a100tcsg324-1)** FPGA.

> This repository accompanies the paper *Design and FPGA Implementation of an Optimized 4×4 Reversible Multiplier with Balanced Design Parameters*, presented at the 2025 13th International Conference on Intelligent Embedded, MicroElectronics, Communication and Optical Networks (IEMECON). DOI: [10.1109/IEMECON69302.2025.11365691](https://doi.org/10.1109/IEMECON69302.2025.11365691)

---

## Table of Contents
1. [Motivation](#motivation)
2. [Reversible Gates Used](#reversible-gates-used)
3. [Architecture](#architecture)
4. [Results](#results)
5. [FPGA Implementation](#fpga-implementation)
6. [Repository Structure](#repository-structure)
7. [Getting Started](#getting-started)
8. [Citation](#citation)
9. [Authors](#authors)

---

## Motivation

Conventional (irreversible) logic gates such as AND and OR discard information: several input combinations map to the same output, so the inputs cannot be recovered. Landauer showed in 1961 that every bit of information erased dissipates at least *kT·ln 2* joules as heat, and Bennett later showed that logic built from **reversible** gates, where inputs can always be recovered uniquely from outputs, can avoid this loss in principle.

Reversible logic matters for **low-power VLSI** and **quantum arithmetic**, and the multiplier is one of the most important arithmetic blocks. However, reversible multipliers are hard to optimize: designs that reduce one parameter tend to worsen another. Common figures of merit are:

| Parameter | Meaning |
|---|---|
| **Gate count** | Total number of reversible gates in the circuit |
| **Constant (ancilla) inputs** | Inputs fixed to 0 or 1 |
| **Garbage outputs** | Extra outputs that are not used for further computation |
| **Quantum cost (QC)** | Number of primitive 1×1 and 2×2 quantum gates needed to realize the circuit |
| **TRLIC** | Total Reversible Logic Implementation Cost |

This work proposes a multiplier that aims to **balance** these parameters instead of optimizing only one of them.

---

## Reversible Gates Used

**Peres gate (3×3, quantum cost 4)**
Inputs (A, B, C), outputs:
- P = A
- Q = A ⊕ B
- R = AB ⊕ C

Used as an **AND gate** (with C = 0) for partial-product generation, and as a **half adder**.

**TSG gate (4×4, quantum cost 4)**
Inputs (A, B, C, D), outputs:
- P = A
- Q = A'C' ⊕ B'
- R = (A'C' ⊕ B')·D
- S = (A'C' ⊕ B')·D ⊕ (AB ⊕ C)

Used as a **reversible full adder** (with D = 0).

---

## Architecture

Two 4-bit operands A (A3..A0) and B (B3..B0) are multiplied to produce an 8-bit product (P7..P0). The design generates 16 partial products with AND operations and sums them with reversible half and full adders in a ripple-carry structure, split into three sequential blocks:

| Block | Gates | Outputs |
|---|---|---|
| **1** – first partial products | 10 Peres + 2 TSG | P0, P1 |
| **2** – second partial product | 5 Peres + 3 TSG | P2 |
| **3** – third partial products | 5 Peres + 3 TSG | P3, P4, P5, P6, P7 |

The output of each block feeds the next. In total the design uses **20 Peres + 8 TSG = 28 gates**, giving a quantum cost of 20 × 4 + 8 × 4 = **112**.

The architecture is built from repeating Peres/TSG cells and is designed to extend to **N×N** multipliers using the same gates.

---

## Results

### Design parameters

| Parameter | Proposed design |
|---|---|
| Gate count | 28 |
| Constant inputs | 28 |
| Garbage outputs | 52 |
| Quantum cost | 112 |
| TRLIC | 220 |

### Comparison with prior work

| Compared against | Quantum cost improvement | TRLIC improvement |
|---|---|---|
| Design [10] in the paper | **31.70%** | 19.41% |
| Design [5] in the paper | 41.66% | **30.59%** |

The headline results are up to **31.7% lower quantum cost** and up to **30.6% better TRLIC** than previously reported reversible multipliers.

**Trade-off note:** the design reaches its low quantum cost and TRLIC with a garbage-output count (52) that is higher than several of the compared designs. The "balanced parameters" claim refers to the overall combination of gate count, constant inputs, quantum cost and TRLIC, and not to the lowest value in every column.

### FPGA timing and power (Artix-7 xc7a100tcsg324-1)

| Metric | Value |
|---|---|
| Delay | 0.9 ns |
| Static power | 0.110 W |
| Dynamic power | 4.006 W |

---

## FPGA Implementation

The design was verified in two stages:

1. **Simulation** in Xilinx Vivado with two test vectors:
   - `a = 0100`, `b = 0100` (4 × 4 = 16)
   - `a = 0101`, `b = 0111` (5 × 7 = 35)
2. **Hardware validation** on a **Nexys A7** board. Simulated and FPGA outputs match.

**Board I/O mapping**

| Signal | Pins | Peripheral |
|---|---|---|
| a0, a1, a2, a3 | J15, L16, M13, R15 | Switches |
| b0, b1, b2, b3 | R17, T18, V18, R13 | Switches |
| p0 … p7 | H17, K15, J13, N14, R18, V17, U17, U16 | LEDs |

A lit LED represents binary 1 and an unlit LED represents binary 0.

---

## Repository Structure

> Adjust this section to match the actual files in your repository.

```
.
├── src/            # Verilog sources (Peres gate, TSG gate, 4x4 multiplier top module)
├── tb/             # Testbench(es) used for simulation
├── constraints/    # Nexys A7 .xdc constraints file
├── docs/           # Waveforms, block diagrams, paper
└── README.md
```

---

## Getting Started

### Requirements
- Xilinx Vivado (used for simulation, synthesis and implementation)
- Digilent Nexys A7-100T board (Artix-7 xc7a100tcsg324-1) for hardware testing

### Simulate
1. Create a new Vivado project and add the sources from `src/` and the testbench from `tb/`.
2. Set the testbench as the simulation top module.
3. Run behavioral simulation and compare the product outputs against the expected values (for example 4 × 4 = 16 and 5 × 7 = 35).

### Run on hardware
1. Add the constraints file from `constraints/` (switch pins for inputs, LED pins for outputs).
2. Run synthesis, implementation and generate the bitstream.
3. Program the Nexys A7 board, set the input switches, and read the 8-bit product on the LEDs.

---

## Citation

If you use this work, please cite:

```bibtex
@inproceedings{saha2025reversible,
  title     = {Design and FPGA Implementation of an Optimized 4x4 Reversible Multiplier with Balanced Design Parameters},
  author    = {Saha, Mihir Lal and Roy, Anamitra and Das, Anushmita and Talukder, Ahana and Maity, Agnik and Sinha, Antarik},
  booktitle = {2025 13th International Conference on Intelligent Embedded, MicroElectronics, Communication and Optical Networks (IEMECON)},
  year      = {2025},
  doi       = {10.1109/IEMECON69302.2025.11365691}
}
```

---

## Authors

Department of Electronics and Communication Engineering, Institute of Engineering & Management (IEM), Kolkata

- Mihir Lal Saha
- Anamitra Roy
- Anushmita Das
- Ahana Talukder
- Agnik Maity
- Antarik Sinha

---

## License

Add your preferred license here (for example MIT).
