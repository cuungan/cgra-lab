# CGLA Architecture Specification

## 1. Purpose

This document defines the first experimental architecture of the `cgra-lab` project.

The architecture is called:

> **CGLA — Coarse-Grained Linear Array**

CGLA is used in this project to describe a deliberately simplified, one-dimensional CGRA architecture.

It is not intended to redefine the term CGRA or claim that CGLA is a universally standardized architecture category.

The objective is to create a small and measurable architecture that can be implemented on an FPGA and later extended with more advanced mapping, memory and AI capabilities.

---

# 2. Design Philosophy

The first CGLA implementation should be:

* small,
* regular,
* synthesizable,
* easy to simulate,
* easy to map,
* measurable,
* and extensible.

The project will therefore avoid implementing a highly general CGRA in the first iteration.

Instead, the architecture will start with a restricted design space:

```text
Data width:
    16 bit

PE:
    Homogeneous

Operations:
    ADD
    SUB
    MUL
    MAC
    AND
    OR
    XOR
    SHIFT

Interconnect:
    1D nearest-neighbor

Configuration:
    Static initially

Memory:
    Register-based initially

Execution:
    Pipelined

Implementation:
    FPGA RTL
```

The architecture can later be expanded if measurements justify additional complexity.

---

# 3. High-Level Architecture

The first CGLA will consist of a chain of processing elements.

```text
                    CGLA

         ┌────┐    ┌────┐    ┌────┐    
Input ──►│PE0 │───►│PE1 │───►│PE2 │───► Output 
         └────┘    └────┘    └────┘    
            │         │         │      
            ▼         ▼         ▼      
          Regs      Regs      Regs     
```

Each PE performs a coarse-grained operation on 16-bit data.

The basic communication model is:

```text
PE[i] ↔ PE[i+1]
```

The initial workloads will primarily use forward streaming, while the interconnect supports bidirectional communication between neighboring PEs.

Later versions may add:

* bypass paths,
* broadcast,
* multicast,
* longer-distance connections,
* local memories,
* dynamic configuration.

---

# 4. Processing Element

The Processing Element is the fundamental computational block.

A first-generation PE is conceptually:

```text
              ┌─────────────────────────┐
              │           PE            │
              │                         │
Input A ─────►│                         │
              │       Input MUX         │
Input B ─────►│            │            │
              │            ▼            │
              │      Functional Unit    │
              │            │            │
              │            ▼            │
              │      Output Register    │
              │                         │
Config ──────►│      Control Logic      │
              └───────────┬─────────────┘
                          │
                          ▼
                       Output
```

The PE should remain intentionally simple.

---

# 5. Datapath Width

The first implementation uses:

> **16-bit signed integer datapaths**

This gives a useful starting point for edge-AI and DSP-oriented experiments while keeping the hardware manageable.

The architecture should be parameterized so that the RTL can later support:

```text
DATA_WIDTH = 8
DATA_WIDTH = 16
DATA_WIDTH = 32
```

However, the first benchmark configuration will use:

```text
DATA_WIDTH = 16
```

The parameterization is important because future experiments may compare:

* 8-bit quantized inference,
* 16-bit fixed-point computation,
* 32-bit integer computation.

---

# 6. Arithmetic Operations

The first PE will support:

```text
ADD
SUB
MUL
MAC
```

The MAC operation is particularly important for AI workloads:

```text
ACC = A × B + ACC
```

A conceptual datapath is:

```text
       A ─────┐
              ▼
             MUL
              │
              ▼
       ACC ─► ADD ───► ACC
```

The MAC operation may require a wider internal accumulator.

Therefore:

```text
Input:
    16 bit

Multiply:
    16 × 16 → 32 bit

Accumulator:
    32 bit initially
```

The external datapath remains 16-bit while the internal MAC path can use 32-bit precision.

This distinction will be important when we later investigate quantization and overflow behavior.

---

# 7. Logical Operations

The PE will also support basic bitwise operations:

```text
AND
OR
XOR
```

These operations are not primarily intended for neural-network acceleration.

They are included because they:

* simplify general-purpose kernel experiments,
* provide useful RTL verification cases,
* allow simple control/data-processing algorithms to be mapped,
* and make the PE a more useful generic computational element.

---

# 8. Shift Operations

The initial PE will support:

```text
SLL
SRL
SRA
```

where:

* SLL = shift left logical
* SRL = shift right logical
* SRA = shift right arithmetic

Arithmetic right shift is particularly useful for fixed-point experiments.

For example, a fixed-point multiplication can approximately be scaled using:

```text
result = product >>> FRACTIONAL_BITS
```

This will allow the architecture to experiment with fixed-point neural-network kernels without requiring floating-point hardware.

---

# 9. PE Register Structure

The first PE will contain a small register structure.

Conceptually:

```text
             ┌─────────────┐
Input A ────►│             │
Input B ────►│   Register  │
             │    Bank     │
             └──────┬──────┘
                    │
                    ▼
                 ALU/MAC
                    │
                    ▼
              Output Register
```

The first implementation does not require a large register file.

A minimal implementation can use:

```text
R0
R1
ACC
OUT
```

The exact number of registers will be parameterized after the first RTL experiments.

---

# 10. PE Inputs and Outputs

Each PE will initially expose:

```text
left_in
right_in
result
```

Conceptually:

```text
left_in ──────► Input MUX ────► ALU/MAC ────► result
                    ▲
                    │
right_in ───────────┘
```

The PE can therefore select operands from:

* the left neighboring PE,
* the right neighboring PE,
* local registers,
* external input.

The initial workloads will primarily use forward streaming:

```text
PE0 → PE1 → PE2 → PE3
```

The underlying nearest-neighbor interconnect remains bidirectional,
allowing data to move between adjacent PEs in either direction.

---

# 11. Interconnect

The first CGLA uses a nearest-neighbor linear interconnect.

```text
PE0 ─── PE1 ─── PE2 ─── PE3 ─── PE4
```

Each PE has access primarily to its neighboring PE.

The architecture intentionally avoids a full crossbar.

This reduces:

* routing complexity,
* multiplexer count,
* configuration bits,
* wiring,
* and potentially power.

Interconnect is a significant architectural cost in CGRAs, so restricting connectivity is a meaningful experimental choice rather than merely a simplification.

---

# 12. Interconnect Version 1

The first implementation will use a bidirectional nearest-neighbor
interconnect between adjacent PEs.

Conceptually:

PE0 ↔ PE1 ↔ PE2 ↔ PE3

Each PE can receive data from its left and right neighboring PE.
The initial workloads will primarily use forward streaming:

PE0 → PE1 → PE2 → PE3

This provides a simple and regular communication structure while
keeping the RTL interface extensible for future architectural
extensions.

---

# 13. Configuration

Each PE needs configuration information specifying its operation.

A first configuration word can contain:

```text
┌────────┬──────────┬──────────┬─────────┐
│ opcode │ input_sel│ feedback │ control │
└────────┴──────────┴──────────┴─────────┘
```

The exact bit allocation will be defined after the operation set is finalized.

For example:

```text
opcode

0000 ADD
0001 SUB
0010 MUL
0011 MAC
0100 AND
0101 OR
0110 XOR
0111 SLL
1000 SRL
1001 SRA
```

The remaining encoding space is reserved for future operations.

---

# 14. Configuration Memory

The first version will use static configuration.

Conceptually:

```text
Configuration Memory
        │
        ├────► PE0 configuration
        ├────► PE1 configuration
        ├────► PE2 configuration
        └────► PE3 configuration
```

A configuration can be loaded before executing a kernel.

The architecture will therefore initially follow:

```text
LOAD CONFIGURATION
        ↓
LOAD DATA
        ↓
EXECUTE
        ↓
READ RESULT
```

Dynamic reconfiguration will be investigated later.

---

# 15. Contexts

A **context** represents the configuration of the array for a particular execution phase.

For example:

```text
Context 0:
    PE0 = MUL
    PE1 = ADD
    PE2 = MAC
    PE3 = ADD
```

A second context could be:

```text
Context 1:
    PE0 = ADD
    PE1 = MUL
    PE2 = MUL
    PE3 = MAC
```

The first CGLA implementation will use one context at a time.

Later versions may support:

```text
Context 0 → Context 1 → Context 2
```

without requiring a complete stop of the accelerator.

---

# 16. Pipeline

The architecture should be designed as a pipeline.

Example:

```text
Cycle 0:
    PE0

Cycle 1:
    PE0 → PE1

Cycle 2:
    PE0 → PE1 → PE2

Cycle 3:
    PE0 → PE1 → PE2 → PE3
```

After the pipeline is filled, multiple operations can execute simultaneously.

For a sufficiently regular streaming workload:

```text
Throughput ≈ one result per cycle
```

may become possible.

This is a target to be measured, not an assumption.

---

# 17. Latency vs Throughput

The project will explicitly distinguish latency from throughput.

For example, a four-PE pipeline may have:

```text
Latency:
    several cycles

Steady-state throughput:
    potentially 1 result/cycle
```

This distinction becomes particularly important for AI workloads where many independent data items pass through the same computational pipeline.

---

# 18. Memory Architecture — Version 1

The first implementation deliberately avoids a complex memory subsystem.

Initial architecture:

```text
Host / FPGA
     │
     ▼
Input Buffer
     │
     ▼
CGLA
     │
     ▼
Output Buffer
```

The first implementation can use FPGA registers and/or BRAM for input/output storage.

A larger scratchpad architecture will be introduced only after the compute pipeline has been validated.

This separation allows the project to answer an important question:

> Is the CGLA compute architecture working correctly before memory-system complexity is introduced?

---

# 19. Host Interface

The initial CGLA should be controlled by a simple host interface.

Conceptually:

```text
             Host
              │
       ┌──────┼──────┐
       ▼      ▼      ▼
    Config   Input  Control
       │      │      │
       └──────┼──────┘
              ▼
             CGLA
              │
              ▼
            Output
```

The first FPGA implementation does not require a full CPU subsystem.

A simple register-mapped interface or testbench-controlled interface is sufficient.

Later versions may integrate:

* MicroBlaze
* RISC-V
* Zynq PS
* AXI4-Lite
* AXI4-Stream

---

# 20. AXI Integration — Later Stage

The eventual architecture may use:

```text
AXI4-Lite
    │
    ├── configuration
    ├── control
    └── status

AXI4-Stream
    │
    ├── input stream
    └── output stream
```

This would allow the CGLA to become a reusable FPGA accelerator IP block.

However, AXI should not be introduced before the internal architecture has been validated.

---

# 21. First RTL Module Structure

The initial RTL project is expected to contain:

```text
rtl/
├── pe/
│   ├── cgla_pe.vhd
│   ├── cgla_alu.vhd
│   └── cgla_mac.vhd
│
├── interconnect/
│   └── cgla_link.vhd
│
└── cgra/
    ├── cgla_array.vhd
    └── cgla_config.vhd
```

The exact file structure may change during implementation.

The important architectural hierarchy is:

```text
CGLA Array
    │
    ├── PE
    │    ├── ALU
    │    ├── MAC
    │    ├── Registers
    │    └── Control
    │
    ├── Interconnect
    │
    └── Configuration
```

---

# 22. Parameterization

The architecture should be parameterized where practical.

Initial parameters:

```text
DATA_WIDTH      = 16
NUM_PES         = 4
ACC_WIDTH       = 32
NUM_REGS        = 4
```

The first implementation should therefore be able to instantiate:

```text
4 PE
8 PE
16 PE
32 PE
```

without fundamentally changing the PE RTL.

This will make FPGA resource and scaling experiments much easier.

---

# 23. First Benchmark

The first benchmark should not be a neural network.

The initial benchmark will be a simple streaming computation.

Example:

```text
Y = ((A × B) + C) × D
```

A possible mapping is:

```text
PE0          PE1          PE2
MUL     →    ADD     →    MUL
```

The pipeline becomes:

```text
A,B ─► MUL ─► +C ─► ×D ─► Y
```

This benchmark tests:

* multiplication,
* addition,
* pipeline behavior,
* configuration,
* data movement,
* latency,
* throughput.

---

# 24. Second Benchmark — MAC

The next benchmark will be:

```text
ACC = Σ(Ai × Bi)
```

Conceptually:

```text
A0 × B0 ─┐
A1 × B1 ─┤
A2 × B2 ─┼──► ACC
A3 × B3 ─┤
...      ┘
```

This benchmark is important because MAC operations are fundamental to many DSP and AI workloads.

A recent 2026 CGRA study also specifically targets configurable MAC execution for edge workloads, illustrating the continued importance of MAC efficiency in CGRA design.

---

# 25. Third Benchmark — Matrix Multiplication

After the MAC benchmark:

```text
C = A × B
```

will become the first significant AI-oriented kernel.

The initial version will use small matrices:

```text
2 × 2
4 × 4
8 × 8
```

The goal is not maximum performance.

The goal is to investigate:

* data reuse,
* PE utilization,
* MAC mapping,
* streaming,
* accumulation,
* configuration.

---

# 26. AI Workload Progression

The planned workload progression is:

```text
Arithmetic
    ↓
Vector operations
    ↓
MAC
    ↓
Matrix multiplication
    ↓
Convolution
    ↓
Quantized neural-network operators
    ↓
Small neural-network model
    ↓
ONNX operators
    ↓
More complex AI workloads
```

Only after these stages will more complex models such as speech/transformer workloads be considered.

This prevents the project from hiding architectural problems behind a large software stack.

---

# 27. Fixed-Point Strategy

The initial architecture will avoid floating-point hardware.

Instead, the project will investigate:

```text
Integer
   ↓
Fixed-point
   ↓
Quantized AI
```

For example:

```text
Q8.8
```

could represent a signed 16-bit fixed-point value with:

```text
8 integer bits
8 fractional bits
```

A later experiment can compare:

```text
INT8
INT16
FP32 software baseline
```

The goal is to determine how much precision is actually required for the target workloads.

---

# 28. Verification Strategy

Verification will proceed incrementally.

### Level 1 — PE

Test:

```text
ADD
SUB
MUL
MAC
AND
OR
XOR
SHIFT
```

### Level 2 — PE chain

Test:

```text
PE0 → PE1
```

### Level 3 — CGLA

Test:

```text
PE0 → PE1 → PE2 → PE3
```

### Level 4 — configuration

Verify that changing configuration changes computation correctly.

### Level 5 — benchmark

Compare RTL results against a software reference model.

---

# 29. Software Reference Model

Every benchmark should have a simple software reference.

For example:

```text
Python / NumPy
       │
       ▼
Expected result
       │
       ├──────────────┐
       │              │
       ▼              ▼
    RTL result     FPGA result
```

This allows the project to distinguish:

* algorithm errors,
* mapping errors,
* RTL errors,
* FPGA integration errors.

---

# 30. Performance Metrics

Each implementation will report:

### Functional

* correctness
* numerical error
* overflow behavior

### FPGA

* LUTs
* FFs
* BRAM
* DSP
* Fmax

### Performance

* latency
* throughput
* cycles/result
* operations/cycle

### Energy

When measurement becomes possible:

* power
* energy/result
* operations/W

---

# 31. First Experimental Questions

The first implementation should answer:

### Q1

How many PEs provide useful speedup before routing becomes a problem?

### Q2

Does a 1D nearest-neighbor network provide sufficient bandwidth for streaming kernels?

### Q3

How much does MAC support improve useful throughput?

### Q4

How much FPGA DSP resource does the architecture consume?

### Q5

How does 8/16/32-bit datapath width affect:

* resource usage,
* frequency,
* throughput,
* numerical accuracy?

### Q6

How difficult is it to map simple computational graphs onto the linear topology?

These questions will determine the next architectural iteration.

---

# 32. Initial Architecture Summary

The first CGLA version is therefore:

```text
┌─────────────────────────────────────────────┐
│                    CGLA                     │
│                                             │
│  ┌────┐   ┌────┐   ┌────┐   ┌────┐        │
│  │ PE │ → │ PE │ → │ PE │ → │ PE │        │
│  └────┘   └────┘   └────┘   └────┘        │
│     │        │        │        │            │
│     ▼        ▼        ▼        ▼            │
│   Regs     Regs     Regs     Regs           │
│                                             │
│      1D nearest-neighbor interconnect       │
│                                             │
│      Static configuration — v1              │
└─────────────────────────────────────────────┘
```

### Version 1 Parameters

```text
PE count       = 4
Data width     = 16 bit
Accumulator    = 32 bit
PE type        = homogeneous
Interconnect   = 1D nearest-neighbor
Configuration  = static
Memory         = registers / FPGA BRAM
Arithmetic     = integer / fixed-point
Target         = FPGA
```

---

# 33. Planned Evolution

The architecture will evolve experimentally:

```text
CGLA v0.1
   │
   ├── 4 PE
   ├── 16-bit
   ├── ADD/MUL/MAC
   └── static configuration
          │
          ▼
CGLA v0.2
   │
   ├── configurable PE
   ├── better local registers
   └── benchmark suite
          │
          ▼
CGLA v0.3
   │
   ├── mapper
   ├── configuration generator
   └── automated tests
          │
          ▼
CGLA v0.4
   │
   ├── BRAM/scratchpad
   ├── streaming interface
   └── AXI integration
          │
          ▼
CGLA v1.0
   │
   ├── AI kernels
   ├── quantization
   ├── performance analysis
   └── FPGA demonstration
```

The architecture should only become more complex when experimental results justify the additional hardware.

---

# 34. Design Principle

The central design principle of CGLA is:

> **Keep the architecture regular enough to map and implement easily, but programmable enough to demonstrate meaningful acceleration.**

The project is therefore not attempting to build the most general CGRA.

It is attempting to understand how much useful acceleration can be obtained from a deliberately simple coarse-grained linear architecture.

---

# 35. Next Step

The next implementation step is to define the PE precisely at RTL level.

The PE specification will define:

```text
Inputs
Outputs
Registers
Opcode encoding
ALU behavior
MAC behavior
Pipeline registers
Reset behavior
Configuration interface
```

Only after this specification is frozen should RTL implementation begin.
