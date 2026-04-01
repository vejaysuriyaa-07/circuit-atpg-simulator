# Scan Chain Simulator

A full-scan insertion and test simulator for sequential circuits. Parses ISCAS-89 `.bench` netlists, inserts a scan chain across all flip-flops, and measures stuck-at fault coverage using pseudo-random patterns.

## The Core Idea

Sequential circuits are hard to test because flip-flop states are not directly controllable or observable from the primary I/Os. Full scan solves this by chaining every DFF through a serial shift register:

- **Shift mode** (SE=1): test data is shifted serially into all FFs — makes every FF state directly controllable
- **Capture mode** (SE=0): one clock cycle of normal operation — FF next-states are captured
- **Shift-out**: captured FF states are shifted out serially — makes every FF state directly observable

After full scan insertion, the circuit is functionally equivalent to a combinational circuit where:
- **Pseudo-PIs** = Primary Inputs + FF Q-outputs (fully controllable via shift-in)
- **Pseudo-POs** = Primary Outputs + FF D-inputs (fully observable via shift-out)

This lets you apply standard combinational ATPG/random testing to a sequential circuit.

## Architecture

```
LFSR → [PI pattern + FF pattern] → shift-in → capture → shift-out → compare
                                                    ↑
                                           fault injection here
```

**Circuit** — Parses `.bench` format, levelizes the combinational cone (DFFs are treated as cut points), evaluates gates in topological order, and supports per-gate fault injection.

**ScanChain** — Models the serial scan register. `shiftIn` loads a pattern directly into all FF states. `shiftOut` reads the FF D-input values after capture.

**ScanController** — Generates patterns via a 32-bit Galois LFSR (separate bits for PIs and FF states), runs fault-free simulation, then re-simulates each undetected fault and checks if any pseudo-PO differs.

## Build

```bash
mkdir build && cd build
cmake .. && make
```

## Usage

```
./scan-simulator <circuit.bench> [-n num_patterns] [-seed value] [-o output_file]
```

| Flag | Default | Description |
|------|---------|-------------|
| `-n` | 1024 | Number of scan test patterns |
| `-seed` | 0xACE1 | LFSR seed |
| `-o` | — | Write per-pattern CSV coverage data |

## Benchmark Format (.bench)

```
INPUT(name)
OUTPUT(name)
gate = AND(in1, in2)
gate = DFF(d_input)
```

Supported gate types: `AND`, `OR`, `NAND`, `NOR`, `NOT`, `BUFF`, `XOR`, `XNOR`, `DFF`

## Examples

```bash
./scan-simulator ../benchmarks/s27.bench
./scan-simulator ../benchmarks/s27.bench -n 2000 -o coverage.csv
./scan-simulator ../benchmarks/s_simple.bench -n 500
```

## Sample Output

```
============================================
      Scan Chain Simulation Results
============================================
  Circuit:            s27
  Primary Inputs:     4
  Primary Outputs:    1
  Flip-Flops:         3
  Scan Chain Length:  3
  Pseudo-PIs:         7
  Pseudo-POs:         4
  Total Gates:        14
  Patterns Applied:   1000
  Total Faults:       28
  Detected Faults:    28
  Fault Coverage:     100.00%
============================================
```

## Fault Coverage Results

| Circuit   | FFs | Pseudo-PIs | Patterns | Fault Coverage |
|-----------|-----|------------|----------|----------------|
| s27       | 3   | 7          | 1000     | 100.00%        |
| s_simple  | 3   | 6          | 1000     | 100.00%        |
