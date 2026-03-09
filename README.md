# Circuit ATPG Simulator

A C++ command-line simulator for digital logic circuits, implementing fault simulation and automatic test pattern generation (ATPG) algorithms used in VLSI chip testing.

## What It Does

Given a digital circuit netlist, this tool can:
- Simulate circuit logic for a given set of inputs
- Automatically generate test patterns to detect stuck-at faults
- Measure fault coverage across a full circuit
- Use multiple ATPG strategies to maximize coverage efficiently

## Features

| Feature | Description |
|---------|-------------|
| **Logic Simulation** | Simulate circuit behavior for any input pattern |
| **LEV** | Compute gate levels (circuit depth analysis) |
| **SCOAP** | Controllability/Observability analysis for testability |
| **RTPG** | Random test pattern generation (binary or ternary mode) |
| **PFS** | Parallel fault simulation — detects multiple faults per pattern |
| **DFS** | Deductive fault simulation |
| **RFL** | Reduced fault list — eliminates redundant faults |
| **DALG** | D-Algorithm — targeted test pattern generation per fault |
| **PODEM** | Path-Oriented Decision Making — ATPG algorithm |
| **TPFC / DTPFC** | Test pattern fault coverage counting |
| **TPG** | Combined test pattern generation with 5 strategies (v0–v4) |

## Tech Stack

- **Language:** C++11
- **Build:** CMake 3.5+
- **Platform:** Linux / macOS

## Build

```bash
mkdir build && cd build
cmake ..
make
```

## Usage

Run the simulator interactively or pipe commands:

```bash
./build/simulator
# or
echo "READ ckts/c17.ckt
LEV lev_out.txt
QUIT" | ./build/simulator
```

### Core Commands

```bash
# Read a circuit
READ ckts/c17.ckt

# Logic simulation with an input pattern file
LOGICSIM pattern.txt

# Levelize the circuit
LEV output.txt

# SCOAP testability analysis
SCOAP output.txt

# Reduced fault list
RFL output.txt

# Random test pattern generation (5 patterns, binary mode)
RTPG 5 b output.txt

# Parallel fault simulation
PFS pattern.txt fault.txt output.txt

# Deductive fault simulation
DFS pattern.txt fault.txt output.txt
```

### ATPG Commands

```bash
# D-Algorithm for a specific fault (node 22, stuck-at-0)
DALG 22 0 output.txt

# PODEM for a specific fault
PODEM 22 0 output.txt
```

### TPG — Test Pattern Generation (5 Strategies)

```bash
# v0: Pure ATPG — finds a test pattern for every fault
TPG -rtpg v0 -atpg DALG DF-nl JF-v0 -out output.txt

# v1: Random patterns until fault coverage hits target (e.g. 80%)
TPG -rtpg v1 0.8 -atpg DALG DF-nl JF-v0 -out output.txt

# v2: Random + ATPG, stops when per-pattern improvement drops below threshold
TPG -rtpg v2 0.01 -atpg DALG DF-nl JF-v0 -out output.txt

# v3: Sliding window — stops when avg improvement over last K patterns < threshold
TPG -rtpg v3 10 0.001 -atpg DALG DF-nl JF-v0 -out output.txt

# v4: Best-of-Q selection — generates Q candidates, keeps the most effective one
TPG -rtpg v4 5 0.01 -atpg DALG DF-nl JF-v0 -out output.txt
```

### DALG Options

| Flag | Options | Description |
|------|---------|-------------|
| `DF-*` | `nl`, `nh`, `lh`, `cc` | Dominance frontier heuristic |
| `JF-*` | `v0`, `v1` | Justification frontier strategy |

## Example: Full ATPG Run on c17

```bash
echo "READ ckts/c17.ckt
TPG -rtpg v0 -atpg DALG DF-nl JF-v0 -out results.txt
QUIT" | ./build/simulator
```

Output:
```
Initial fault list size: 22
Running DALG for 21@0 ... Fault Coverage: 0.136
...
Fault Coverage: 1.0 (undetectable: 0)
Test patterns written to: results.txt
Total patterns: 9
```

## Circuit Files

Sample circuits in `ckts/` (ISCAS-85 benchmark suite):

| Circuit | Gates | Primary Inputs | Primary Outputs |
|---------|-------|----------------|-----------------|
| c17 | 6 | 5 | 2 |
| c432 | ~160 | 36 | 7 |
| c499 | ~202 | 41 | 32 |
| c880 | ~383 | 60 | 26 |
| c1355 | ~546 | 41 | 32 |
| c3540 | ~1669 | 50 | 22 |

## Project Structure

```
circuit-atpg-simulator/
├── src/          # C++ implementation files
├── includes/     # Header files
│   └── core/     # Algorithm headers
├── ckts/         # Circuit netlists (ISCAS-85 benchmarks)
├── main.cpp      # Entry point & command parser
└── CMakeLists.txt
```
