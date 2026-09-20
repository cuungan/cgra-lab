# CGRA Architecture Taxonomy

## 1. Purpose

A CGRA is not a single fixed architecture.

Different CGRA implementations make different choices regarding:

* processing elements,
* interconnect topology,
* memory organization,
* configuration mechanism,
* execution model,
* reconfiguration granularity,
* and compiler/mapping strategy.

Understanding these choices is important before designing a new architecture.

This document studies the main architectural dimensions of CGRAs and uses them to define the design space for the `cgra-lab` project.

---

## 2. Common CGRA Structure

A typical CGRA contains four major components:

```text
                Host Processor
                      │
                      ▼
              ┌───────────────┐
              │ Configuration │
              │    Memory     │
              └───────┬───────┘
                      │
                      ▼
       ┌──────────────────────────────┐
       │       CGRA Processing Array  │
       │                              │
       │   PE ─ PE ─ PE ─ PE          │
       │    │    │    │    │          │
       │   PE ─ PE ─ PE ─ PE          │
       │    │    │    │    │          │
       │   PE ─ PE ─ PE ─ PE          │
       │                              │
       └──────────────┬───────────────┘
                      │
                      ▼
                Data Memory
```

The exact organization differs significantly between architectures.

A 2024 survey of CGRAs for radio baseband processing describes the common structure as a PE array, programmable interconnect, configuration memory and controller, with memory systems supporting both data and configuration information.

---

# 3. Processing Element Architecture

The Processing Element (PE) is the basic computational unit.

A simple PE can be represented as:

```text
              ┌─────────────┐
 Input A ────►│             │
 Input B ────►│     ALU     │───► Result
              │             │
 Config ─────►│             │
              └─────────────┘
```

A more capable PE may contain:

```text
             ┌────────────────────┐
             │        PE          │
             │                    │
Input A ────►│ Register File      │
Input B ────►│       │            │
             │       ▼            │
             │   Functional Unit  │
             │       │            │
             │       ▼            │
             │   Output Register  │
             └────────────────────┘
```

Typical operations include:

* ADD
* SUB
* MUL
* MAC
* AND
* OR
* XOR
* SHIFT
* COMPARE
* MIN/MAX

The required PE functionality depends strongly on the target workload.

For example, an architecture targeting DSP and neural-network workloads may benefit from efficient multiply-accumulate operations.

---

# 4. Homogeneous vs Heterogeneous PEs

## 4.1 Homogeneous Array

All PEs have approximately the same functionality.

```text
┌────┐ ┌────┐ ┌────┐ ┌────┐
│ PE │ │ PE │ │ PE │ │ PE │
│ALU │ │ALU │ │ALU │ │ALU │
└────┘ └────┘ └────┘ └────┘
```

Advantages:

* simple architecture
* simpler compiler model
* easier physical design
* predictable resource availability

Disadvantages:

* specialized operations may require multiple PEs
* potentially lower efficiency for heterogeneous workloads

---

## 4.2 Heterogeneous Array

Different PEs provide different capabilities.

```text
┌────┐ ┌────┐ ┌────┐ ┌────┐
│ ALU│ │ MUL│ │ MAC│ │ DSP│
└────┘ └────┘ └────┘ └────┘
```

Advantages:

* specialized operations can be executed efficiently
* potentially better performance for complex workloads

Disadvantages:

* more complicated mapping
* less flexibility
* more complicated hardware
* potentially more difficult resource balancing

The choice between homogeneous and heterogeneous PEs is one of the important CGRA architectural trade-offs.

---

# 5. Interconnect Topology

The interconnect determines how PEs communicate.

This is one of the most important architectural decisions in a CGRA.

A simplified model is:

```text
PE ─── PE ─── PE
│      │      │
PE ─── PE ─── PE
│      │      │
PE ─── PE ─── PE
```

Possible interconnect structures include:

* nearest-neighbor networks
* mesh networks
* buses
* crossbars
* hierarchical networks
* Network-on-Chip structures

Interconnect flexibility comes with hardware cost. Recent CGRA literature identifies the interconnect as a major contributor to both area and energy, making it a critical design dimension rather than merely a connectivity detail.

---

# 6. Nearest-Neighbor Interconnect

In a nearest-neighbor architecture, each PE communicates primarily with adjacent PEs.

```text
PE ─── PE ─── PE ─── PE
│      │      │      │
PE ─── PE ─── PE ─── PE
```

Advantages:

* simple routing
* relatively low hardware cost
* predictable communication
* good physical scalability

Disadvantages:

* long-distance communication requires multiple hops
* mapping may become difficult for applications with non-local dependencies

This topology is particularly interesting for the CGLA direction.

---

# 7. Mesh Interconnect

A two-dimensional mesh provides horizontal and vertical communication.

```text
PE ─── PE ─── PE ─── PE
│      │      │      │
PE ─── PE ─── PE ─── PE
│      │      │      │
PE ─── PE ─── PE ─── PE
│      │      │      │
PE ─── PE ─── PE ─── PE
```

The mesh provides significantly more spatial flexibility than a one-dimensional array.

However, additional routing resources increase implementation complexity.

The mesh is therefore a common reference architecture when discussing CGRA design space.

---

# 8. Crossbar Interconnect

A crossbar provides highly flexible connectivity.

Conceptually:

```text
       ┌───────────────┐
PE0 ──►│               │
PE1 ──►│   Crossbar    │──► PE0
PE2 ──►│               │──► PE1
PE3 ──►│               │──► PE2
       └───────────────┘
```

Advantages:

* high connectivity
* flexible mapping
* easier support for arbitrary communication patterns

Disadvantages:

* large area
* high routing complexity
* potentially high power consumption
* poor scalability for large arrays

For a small research prototype a crossbar can be attractive, but for a scalable low-power architecture it may become expensive.

---

# 9. Linear Array Architecture

A linear array simplifies the spatial organization:

```text
PE0 ─ PE1 ─ PE2 ─ PE3 ─ PE4 ─ PE5 ─ PE6
```

Each PE primarily communicates with its neighboring elements.

This reduces the dimensionality of the routing problem.

Potential advantages include:

* simple and regular interconnect
* predictable data movement
* simple physical implementation
* natural support for streaming pipelines

Potential disadvantages include:

* limited communication locality
* longer paths for non-local dependencies
* potentially lower utilization for irregular algorithms
* reduced mapping flexibility compared with a 2D array

These trade-offs motivate the CGLA experiment in this project.

---

# 10. Linear Array vs 2D Mesh

A conceptual comparison:

| Feature                     |           Linear Array |                   2D Mesh |
| --------------------------- | ---------------------: | ------------------------: |
| Interconnect complexity     |                  Lower |                    Higher |
| Routing flexibility         |                  Lower |                    Higher |
| Physical layout             |                 Simple |              More complex |
| Long-distance communication |         More expensive |             More flexible |
| Streaming workloads         |     Potentially strong |                    Strong |
| Irregular workloads         |         More difficult |             More flexible |
| Compiler complexity         |      Potentially lower |                    Higher |
| Low-power potential         | Potentially attractive | Depends on implementation |

These are architectural tendencies, not universal performance guarantees.

Actual results depend on workload, PE capabilities, memory system and mapping algorithm.

---

# 11. Memory Architecture

Computation alone does not determine CGRA performance.

Data must reach the PEs efficiently.

A simplified memory hierarchy is:

```text
             Main Memory
                  │
                  ▼
            Global Buffer
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
    Local Memory         Local Memory
       PE0                   PE1
        │                     │
        ▼                     ▼
       PE0                   PE1
```

Possible memory structures include:

* register files
* local SRAM
* distributed memories
* shared buffers
* scratchpad memories
* external DRAM

For AI workloads, data reuse is particularly important.

Moving data can consume more energy than performing an arithmetic operation.

Therefore the mapping strategy should eventually consider computation and data movement together.

---

# 12. Configuration Architecture

The CGRA needs configuration information describing how the array should operate.

A configuration may specify:

```text
PE operation
input selection
output routing
register behavior
memory access
control information
```

A simple configuration word might conceptually look like:

```text
┌────────┬────────┬────────┬────────┐
│ opcode │ input  │ output │ control│
└────────┴────────┴────────┴────────┘
```

The configuration can be:

* static for a complete kernel
* changed between kernels
* changed periodically
* dynamically updated during execution

The amount and timing of reconfiguration influence both flexibility and overhead.

---

# 13. Static vs Dynamic Reconfiguration

## Static

The CGRA is configured before a kernel begins execution.

```text
Configure
   │
   ▼
Execute kernel
   │
   ▼
Reconfigure
   │
   ▼
Execute next kernel
```

Advantages:

* simpler hardware
* predictable execution
* low runtime control overhead

This is attractive for an initial FPGA prototype.

---

## Dynamic

The configuration changes while computation is running.

```text
Config A → Config B → Config C → Config D
       │
       ▼
    Execution
```

Advantages:

* potentially better hardware utilization
* supports larger computations than the physical array
* enables temporal reuse of PEs

Disadvantages:

* more complex control
* configuration bandwidth requirements
* additional runtime overhead

---

# 14. Spatial vs Temporal Computation

CGRA computation can be viewed in two dimensions.

### Spatial

Different operations execute simultaneously on different PEs.

```text
PE0 = ADD
PE1 = MUL
PE2 = SUB
PE3 = SHIFT
```

### Temporal

The same hardware executes different operations at different cycles.

```text
Cycle 0 → ADD
Cycle 1 → MUL
Cycle 2 → SUB
Cycle 3 → SHIFT
```

Real CGRAs can combine both approaches.

This spatio-temporal aspect is central to understanding why CGRA mapping is more complicated than simply assigning operations to PEs.

---

# 15. Mapping Implications

Architecture and compiler cannot be designed independently.

Suppose an application contains:

```text
A → MUL → ADD → MUL → SUB
```

A linear array might map this naturally:

```text
PE0      PE1      PE2      PE3      PE4
MUL  →   ADD  →   MUL  →   SUB
```

A more complicated dependency graph might require data to move backwards or over multiple hops.

The mapping algorithm must therefore solve several related problems:

```text
Application Graph
       │
       ▼
   Scheduling
       │
       ▼
    Placement
       │
       ▼
     Routing
       │
       ▼
 Configuration
```

The 2026 survey of CGRA mapping algorithms explicitly treats mapping as the relationship between application tasks and PEs and discusses the major stages and bottlenecks of this process.

---

# 16. Architecture and Mapping Are Coupled

A key lesson from CGRA research is:

> The best architecture cannot be evaluated independently of the mapping strategy.

For example:

A highly flexible interconnect may make mapping easier but increase area and energy.

A very simple interconnect may reduce hardware cost but make mapping substantially harder.

Therefore:

```text
Architecture
      ↕
Mapping Algorithm
      ↕
Compiler
      ↕
Workload
```

should be considered as a coupled system.

This is one of the central principles that will guide `cgra-lab`.

---

# 17. Major CGRA Design Dimensions

The architecture can therefore be described using several independent dimensions.

```text
                    CGRA
                     │
      ┌──────────────┼──────────────┐
      │              │              │
      ▼              ▼              ▼
     PE          Interconnect     Memory
      │              │              │
      ▼              ▼              ▼
 Homogeneous      Mesh/Line       Local/Shared
 Heterogeneous    Bus/Crossbar    SRAM/Buffer
      │              │              │
      └──────────────┼──────────────┘
                     ▼
              Configuration
                     │
                     ▼
             Static / Dynamic
                     │
                     ▼
                  Mapping
                     │
                     ▼
                Workloads
```

This multidimensional view is more useful than treating CGRA as a single architecture class.

---

# 18. Design Space for cgra-lab

The initial project will intentionally select a small point in this design space.

### Proposed first architecture

```text
PE type:
    Homogeneous

PE operation:
    Integer ALU + MAC

Interconnect:
    1D nearest-neighbor

Memory:
    Small local registers / buffers

Configuration:
    Static initially

Execution:
    Spatial + pipelined

Target:
    FPGA prototype

Workloads:
    Arithmetic kernels
    Matrix multiplication
    MAC
    Small neural-network operators
```

Conceptually:

```text
             CGLA

Input
  │
  ▼
┌────┐   ┌────┐   ┌────┐   ┌────┐   ┌────┐
│ PE │ → │ PE │ → │ PE │ → │ PE │ → │ PE │
└────┘   └────┘   └────┘   └────┘   └────┘
  │        │        │        │        │
  ▼        ▼        ▼        ▼        ▼
Local    Local    Local    Local    Local
Buffer   Buffer   Buffer   Buffer   Buffer
```

This is intentionally much simpler than a general-purpose CGRA.

The purpose is not to claim that a linear architecture is universally superior.

The purpose is to create a controlled experimental platform where architectural trade-offs can be measured.

---

# 19. Why Start With a Linear Array?

The linear topology offers several useful properties for a first research prototype.

### 19.1 Simple RTL

A regular PE chain is relatively straightforward to describe in RTL.

### 19.2 Predictable routing

Each PE primarily communicates with neighboring PEs.

### 19.3 Simple mapping model

The initial mapper can operate on a restricted topology.

### 19.4 Streaming computation

Many signal-processing and ML kernels naturally expose producer-consumer pipelines.

### 19.5 Easier experimentation

A small architecture allows us to measure:

* frequency
* LUT usage
* FF usage
* BRAM usage
* DSP usage
* latency
* throughput
* energy/power

before adding architectural complexity.

---

# 20. Limitations of the Linear Approach

The architecture also has clear limitations.

### Communication distance

An operation requiring data from a distant PE may require multiple hops.

### Mapping constraints

Some graphs that fit naturally on a 2D mesh may be difficult to map onto a line.

### Limited parallel communication

A linear topology provides fewer simultaneous communication paths.

### Irregular workloads

Algorithms with complex communication patterns may experience poor utilization.

These limitations are not problems to hide.

They are part of the experiment.

---

# 21. Experimental Hypothesis

The central hypothesis of the first CGLA experiment is:

> A small, regular, nearest-neighbor coarse-grained linear array can provide useful acceleration for selected streaming and AI kernels while reducing interconnect and implementation complexity compared with a more general CGRA.

This hypothesis must be evaluated experimentally.

The project will therefore measure both computational performance and implementation cost.

---

# 22. Evaluation Method

The architecture should eventually be evaluated using the following metrics:

### Hardware

* LUT utilization
* FF utilization
* BRAM utilization
* DSP utilization
* maximum clock frequency
* estimated power

### Performance

* latency
* throughput
* operations/cycle
* utilization
* speedup versus software baseline

### Efficiency

* operations/W
* performance/mm² where meaningful
* energy per operation

### Mapping

* mapping success rate
* compiler/runtime
* PE utilization
* routing overhead
* configuration size

The goal is to compare architectures using measurable data rather than architectural assumptions.

---

# 23. Research Progression

The planned progression is:

```text
CGRA Literature
       │
       ▼
Architecture Taxonomy
       │
       ▼
CGLA Specification
       │
       ▼
PE RTL
       │
       ▼
Interconnect RTL
       │
       ▼
CGLA RTL
       │
       ▼
FPGA Prototype
       │
       ▼
Mapper
       │
       ▼
AI Kernels
       │
       ▼
Benchmark
       │
       ▼
Architecture Optimization
```

Later versions may investigate:

```text
1D CGLA
   ↓
2D CGRA
   ↓
Hybrid architecture
```

This allows architectural complexity to be added only when measurements demonstrate a need for it.

---

# 24. Key References

The taxonomy in this document is primarily informed by established CGRA surveys and architecture/mapping literature.

* Wijtvliet, Waeijen & Corporaal, *Coarse grained reconfigurable architectures in the past 25 years: overview and classification*, SAMOS 2016/2017. DOI: 10.1109/SAMOS.2016.7818353.
* Podobas, Sano & Matsuoka, *A Survey on Coarse-Grained Reconfigurable Architectures From a Performance Perspective*, IEEE Access, 2020. DOI: 10.1109/ACCESS.2020.3012084.
* Choi, *Coarse-Grained Reconfigurable Array: Architecture and Application Mapping*, IPSJ Transactions on System and LSI Design Methodology, 2011. DOI: 10.2197/ipsjtsldm.4.31.
* Du et al., *Survey of Coarse-Grained Reconfigurable Architectures Mapping Algorithms*, Journal of Computer Science and Technology, 2026. DOI: 10.1007/s11390-026-5611-4.
* *Coarse-grained reconfigurable architectures for radio baseband processing: A survey*, Journal of Systems Architecture, 2024. DOI: 10.1016/j.sysarc.2024.103243.

---

# 25. Conclusion

CGRA architecture is fundamentally a trade-off between:

```text
Flexibility
     ↕
Hardware Cost
     ↕
Communication Cost
     ↕
Performance
     ↕
Energy Efficiency
```

The `cgra-lab` project will deliberately begin with a constrained architecture:

> **A homogeneous coarse-grained linear array with nearest-neighbor communication and simple configurable processing elements.**

This architecture will be called **CGLA** within the project.

The next research step is to define the CGLA architecture precisely enough to implement it in RTL.
