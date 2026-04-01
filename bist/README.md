# BIST Simulator

A Built-In Self-Test simulator for ISCAS-85 benchmark circuits. Uses an LFSR to generate pseudo-random test patterns, simulates the circuit, compresses output responses through a MISR, and measures stuck-at fault coverage via parallel fault simulation.

## Architecture

**LFSR** — Galois-configuration linear feedback shift register. Generates pseudo-random bits using a primitive polynomial for the given width. For circuits with more primary inputs than the LFSR width, the register is clocked multiple times per pattern (phase shifting) to fill all inputs.

**MISR** — Multiple Input Signature Register. Same shift-register structure as the LFSR but XORs incoming output response bits into the state at each clock. The final state is the circuit's signature.

**Fault Simulation** — For each pattern, every undetected stuck-at fault is injected by forcing the faulted gate's output to its stuck value, then re-simulating. If any primary output differs from the fault-free response, the fault is marked detected.

## Build

```bash
mkdir build && cd build
cmake .. && make
```

## Usage

```
./bist-simulator <circuit.ckt> [-n num_patterns] [-seed value] [-misr width] [-o output_file]
```

| Flag | Default | Description |
|------|---------|-------------|
| `-n` | 1024 | Number of test patterns |
| `-seed` | 0xACE1 | LFSR seed |
| `-misr` | 32 | MISR width in bits |
| `-o` | — | Write per-pattern CSV data to file |

## Examples

```bash
./bist-simulator ../../ckts/c17.ckt
./bist-simulator ../../ckts/c432.ckt -n 2000
./bist-simulator ../../ckts/c880.ckt -n 2000 -o coverage.csv
```

## Sample Output

```
============================================
         BIST Simulation Results
============================================
  Circuit:            c432
  Primary Inputs:     36
  Primary Outputs:    7
  Total Gates:        432
  LFSR Width:         32
  MISR Width:         32
  Patterns Applied:   2000
  Total Faults:       864
  Detected Faults:    854
  Fault Coverage:     98.84%
  MISR Signature:     0x84887867
============================================
```

## Fault Coverage Results

| Circuit | Patterns | Fault Coverage |
|---------|----------|----------------|
| c17     | 1024     | 100.00%        |
| c432    | 2000     | 98.84%         |
| c880    | 2000     | 97.50%         |
