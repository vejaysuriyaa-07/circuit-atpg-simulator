# Circuit ATPG Simulator

Built this to actually understand what happens under the hood in tools like Tessent and TetraMAX — instead of just using them as a black box. It's a C++ simulator that parses ISCAS-85 circuit netlists and runs fault simulation + automatic test pattern generation from scratch.

Started with basic logic simulation and kept adding algorithms on top. Ended up implementing DALG and PODEM, a full BIST flow, and a scan chain simulator for sequential circuits.

## What's in here

**ATPG core** (`src/`, `includes/`) — the main simulator with an interactive command interface

| Command | What it does |
|---------|-------------|
| `LEV` | Levelizes the circuit (topological sort) |
| `SCOAP` | Controllability/observability analysis |
| `RTPG` | Random pattern generation |
| `PFS` | Parallel fault simulation |
| `DFS` | Deductive fault simulation |
| `RFL` | Reduces the fault list |
| `DALG` | D-Algorithm — targeted ATPG per fault |
| `PODEM` | Path-Oriented Decision Making ATPG |
| `TPG` | Combined ATPG with 5 different strategies |

**BIST** (`bist/`) — standalone LFSR/MISR built-in self-test simulator, tested on ISCAS-85 benchmarks

**Scan chain** (`scan/`) — full-scan insertion for sequential circuits, models pseudo-PI/PO equivalence

## Build

```bash
mkdir build && cd build
cmake ..
make
```

## Running it

Interactive mode or pipe commands in:

```bash
./build/simulator
```

```bash
echo "READ ckts/c17.ckt
LEV lev_out.txt
QUIT" | ./build/simulator
```

### Basic commands

```bash
READ ckts/c17.ckt
LOGICSIM pattern.txt
LEV output.txt
SCOAP output.txt
RFL output.txt
RTPG 5 b output.txt
PFS pattern.txt fault.txt output.txt
DFS pattern.txt fault.txt output.txt
DALG 22 0 output.txt
PODEM 22 0 output.txt
```

### TPG strategies

```bash
# v0: pure ATPG, finds a pattern for every single fault
TPG -rtpg v0 -atpg DALG DF-nl JF-v0 -out output.txt

# v1: random until coverage hits a target
TPG -rtpg v1 0.8 -atpg DALG DF-nl JF-v0 -out output.txt

# v2: random + ATPG, stops when improvement per pattern drops off
TPG -rtpg v2 0.01 -atpg DALG DF-nl JF-v0 -out output.txt

# v3: sliding window over last K patterns
TPG -rtpg v3 10 0.001 -atpg DALG DF-nl JF-v0 -out output.txt

# v4: generates Q candidates per step, keeps the best one
TPG -rtpg v4 5 0.01 -atpg DALG DF-nl JF-v0 -out output.txt
```

## Example run on c17

```bash
echo "READ ckts/c17.ckt
TPG -rtpg v0 -atpg DALG DF-nl JF-v0 -out results.txt
QUIT" | ./build/simulator
```

```
Initial fault list size: 22
Running DALG for 21@0 ... Fault Coverage: 0.136
...
Fault Coverage: 1.0 (undetectable: 0)
Total patterns: 9
```

## Circuits

All netlists are ISCAS-85 benchmarks in `ckts/`:

| Circuit | Gates | PIs | POs |
|---------|-------|-----|-----|
| c17 | 6 | 5 | 2 |
| c432 | ~160 | 36 | 7 |
| c499 | ~202 | 41 | 32 |
| c880 | ~383 | 60 | 26 |
| c1355 | ~546 | 41 | 32 |
| c3540 | ~1669 | 50 | 22 |

## Subprojects

### BIST (`bist/`)

Implements the LFSR → simulate → MISR compress loop with stuck-at fault coverage tracking. Getting 98%+ on c432 and c880 took some debugging of the phase shifting logic for circuits with more PIs than the LFSR width.

```bash
cd bist && mkdir build && cd build && cmake .. && make
./bist-simulator ../../ckts/c432.ckt -n 2000
```

### Scan Chain (`scan/`)

Full scan insertion for sequential `.bench` circuits. The key insight is that once you insert scan, every FF state is controllable and observable — the circuit behaves like a combinational one between pseudo-PIs and pseudo-POs. Tested on s27 and a small pipeline circuit.

```bash
cd scan && mkdir build && cd build && cmake .. && make
./scan-simulator ../benchmarks/s27.bench -n 1000
```

## Structure

```
circuit-atpg-simulator/
├── src/           C++ implementation
├── includes/      headers
│   └── core/
├── ckts/          ISCAS-85 netlists
├── bist/          BIST subproject
├── scan/          scan chain subproject
├── main.cpp
└── CMakeLists.txt
```
