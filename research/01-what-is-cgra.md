# What is a CGRA?

## 1. Introduction

A **Coarse-Grained Reconfigurable Architecture (CGRA)** is a reconfigurable computing architecture built from an array of relatively powerful processing elements (PEs) connected through a configurable interconnect network.

CGRAs occupy a design space between:

* general-purpose processors, which provide high programmability but relatively limited parallelism,
* GPUs and other throughput-oriented processors,
* FPGAs, which provide very fine-grained reconfigurability but can require significant configuration and routing resources,
* and ASICs, which provide high efficiency but are largely fixed after fabrication.

The central idea of a CGRA is to provide **hardware-like parallel execution with software-like reconfigurability**.

A CGRA typically consists of:

* Processing Elements (PEs)
* configurable interconnects
* registers and/or local memories
* configuration memory
* interfaces to external memory or a host processor

The architecture is configured so that a particular computation can execute spatially across multiple PEs.

---

## 2. Why CGRA?

Modern computing systems increasingly face a trade-off between flexibility, performance and energy efficiency.

A CPU is highly flexible, but many compute-intensive algorithms do not efficiently exploit a general-purpose instruction pipeline.

An ASIC can be extremely efficient for a specific workload, but changing the algorithm usually requires a new hardware implementation.

An FPGA provides a high degree of hardware reconfigurability, but its fine-grained programmable fabric introduces configuration, routing and resource overhead.

CGRA attempts to occupy the space between these approaches.

The fundamental objective is:

> Provide enough programmability to support multiple algorithms while retaining a relatively efficient spatial datapath.

This makes CGRAs interesting for workloads such as:

* digital signal processing
* image and video processing
* wireless communication
* numerical kernels
* scientific computing
* machine-learning inference
* edge AI

---

## 3. Basic CGRA Architecture

A simplified CGRA can be represented as an array of processing elements:

```text
             Configurable Interconnect
        ┌───────┬───────┬───────┬───────┐
        │  PE   │  PE   │  PE   │  PE   │
        ├───────┼───────┼───────┼───────┤
        │  PE   │  PE   │  PE   │  PE   │
        ├───────┼───────┼───────┼───────┤
        │  PE   │  PE   │  PE   │  PE   │
        └───────┴───────┴───────┴───────┘
                  │
                  ▼
          Local / Global Memory
```

The exact architecture varies considerably between CGRA implementations.

A PE may contain:

* ALU
* multiplier
* MAC unit
* comparator
* shifter
* register file
* local storage

Some architectures use homogeneous PEs, while others use heterogeneous functional units.

The interconnect is equally important. It determines which PEs can communicate and therefore strongly influences the types of computations that can be efficiently mapped onto the architecture.

---

## 4. Processing Elements

The Processing Element is the fundamental computational building block of a CGRA.

A simple PE might support operations such as:

```text
ADD
SUB
MUL
AND
OR
XOR
SHIFT
COMPARE
```

A more specialized PE could contain:

```text
MAC
SIMD operations
DSP functions
address generation
custom arithmetic
```

For an AI-oriented CGRA, multiply-accumulate operations are particularly important because matrix and convolution workloads contain large numbers of multiply-accumulate operations.

For example, a simple computation can be expressed as:

```text
y = ReLU(a*x + b)
```
This computation can be represented as a Data Flow Graph (DFG), where each operation is represented as a node and data dependencies are represented as edges:

```text
       a          
       │          
       ▼          
x ──> MUL ──> ADD ──> RELU ──> y
               ▲
               │
               b
```
The computation can then be spatially distributed across the CGRA:

```text
PE0       PE1        PE2
 │         │          │
MUL  ──>  ADD  ──>  RELU
```
Here, each operation is assigned to a different PE, while intermediate results are transferred between neighboring PEs through the configurable interconnect.

This illustrates how a computation can be transformed from an application-level expression into a data-flow representation and subsequently mapped onto the spatial resources of a CGRA.

---

## 5. Configurable Interconnect

The interconnect determines how data moves between PEs.

Common structures include:

* nearest-neighbor connections
* mesh networks
* buses
* crossbars
* hierarchical networks
* Network-on-Chip (NoC) structures

A simple nearest-neighbor architecture might look like:

```text
        PE ─── PE ─── PE
         │     │     │
        PE ─── PE ─── PE
         │     │     │
        PE ─── PE ─── PE
```

The choice of interconnect creates an important architectural trade-off.

A highly connected network provides flexibility but requires more routing resources and may increase power, area and timing complexity.

A simpler network reduces hardware cost but restricts the computations that can be mapped efficiently.

Therefore:

> CGRA performance depends not only on the computational capability of the PEs, but also on the ability of the interconnect to transport data between them.

---

## 6. Configuration

Unlike a conventional CPU, a CGRA typically exposes computation through spatial
configuration of processing elements and data paths rather than relying primarily
on a sequential instruction stream.

However, CGRA execution and configuration models vary significantly between
architectures. Some CGRAs use a relatively static configuration for a kernel,
while others support temporal configuration changes or instruction-driven control.

Instead, configuration information may determine:

* which operation each PE performs,
* which inputs are selected,
* how data moves between PEs,
* where results are sent,
* how memory accesses are performed,
* and, depending on the architecture, how configuration changes over multiple cycles.

The configuration may be stored in configuration memory.

Conceptually:

```text
Configuration
     │
     ▼
┌─────────────────────────────┐
│ PE configuration            │
│ Interconnect configuration  │
│ Memory configuration        │
└─────────────────────────────┘
     │
     ▼
Spatial computation
```

Some CGRAs use a single configuration for a kernel, while others support temporal changes in configuration.

This creates the possibility of **spatio-temporal computation**, where different operations are executed at different locations and at different cycles.

---

## 7. Mapping

One of the most important problems in CGRA research is **mapping**.

Given an application or computational kernel, the compiler must determine:

1. which operation should be executed on which PE,
2. at which cycle the operation should be executed,
3. how data should travel between PEs,
4. how memory accesses should be scheduled,
5. whether the computation fits within the available hardware resources.

A common representation of the computation is a **Data Flow Graph (DFG)**, which
represents operations as nodes and their data dependencies as edges.

For example, consider the following computation:

```text
Y = (A × B) + C
```

This computation can be represented as a DFG:

```text
A ──┐
    │
    ▼
   MUL ──┐
    ▲    │
    │    ▼
B ──┘   ADD ──> Y
          ▲
          │
          C
```

The DFG captures the dependencies between operations. The ADD operation cannot be executed until the result of the MUL operation is available.

The compiler then maps these operations onto the available processing elements (PEs). A simple spatial-temporal mapping can be illustrated as:

```text
Cycle 0
    PE0 = LOAD A
    PE1 = LOAD B
    PE2 = LOAD C

Cycle 1
    PE0 = MUL

Cycle 2
    PE1 = ADD
```

In this example, the intermediate result produced by PE0 must be transferred to PE1 through the CGRA interconnect before the ADD operation can be completed.

The mapping must satisfy both computation dependencies and architectural
constraints such as PE availability, operation latency and communication paths.

Modern CGRA research therefore treats mapping as a central compiler problem involving scheduling, placement and routing.

---

## 8. Scheduling, Placement and Routing

The mapping problem can be separated conceptually into several related tasks.

### Scheduling

Determine **when** an operation executes.

```text
Operation A → cycle 0
Operation B → cycle 1
Operation C → cycle 2
```

### Placement

Determine **where** an operation executes.

```text
ADD → PE3
MUL → PE7
SUB → PE8
```

### Routing

Determine how data moves between operations.

```text
PE3 → PE4 → PE7
```

These decisions are strongly coupled.

A mathematically valid schedule may not be physically routable.

Similarly, an operation may have to be moved to another PE because the required communication path is unavailable.

This is one of the reasons CGRA compiler and mapping research remains an important field.

---

## 9. CGRA vs FPGA

CGRA and FPGA are both reconfigurable architectures, but they operate at different levels of granularity.

### FPGA

An FPGA typically provides:

* LUTs
* flip-flops
* programmable routing
* block RAM
* DSP blocks
* configurable I/O

The programmable fabric is relatively fine-grained.

### CGRA

A CGRA typically provides:

* word-level processing elements
* configurable datapaths
* configurable interconnect
* local memories/registers
* configuration memory

The computational granularity is significantly larger.

Conceptually:

```text
FPGA

LUT LUT FF LUT LUT
 │   │   │   │   │
fine-grained programmable fabric


CGRA

┌────┐ ┌────┐ ┌────┐
│ PE │─│ PE │─│ PE │
└────┘ └────┘ └────┘

word-level computational fabric
```

This distinction is important for this project because the first implementation target will be an FPGA, while the architecture being explored is CGRA-like.

In other words:

> FPGA is the implementation platform; CGRA is the architectural model being implemented.

---

## 10. CGRA vs ASIC

An ASIC implements a fixed architecture optimized for a particular application or workload.

A CGRA sacrifices some of the ASIC's specialization in exchange for reconfigurability.

A simplified comparison is:

| Architecture | Flexibility | Potential efficiency | Hardware specialization |
| ------------ | ----------: | -------------------: | ----------------------: |
| CPU          |        High |           Low–Medium |                     Low |
| GPU          |      Medium |          Medium–High |                  Medium |
| FPGA         |        High |          Medium–High |                  Medium |
| CGRA         | Medium–High |                 High |                    High |
| ASIC         |         Low |            Very High |               Very High |

These categories are qualitative rather than absolute. Actual performance, energy efficiency, area and programmability depend strongly on the workload, architecture, technology and implementation.

---

## 11. Why CGRA is Interesting for AI

Many machine-learning workloads contain highly regular computation.

For example, matrix multiplication:

```text
C = A × B
```

can be decomposed into many multiply-accumulate operations.

A CGRA can potentially exploit this structure by distributing operations across multiple PEs.

Conceptually:

```text
        Matrix / Tensor Data
                 │
                 ▼
       ┌───────────────────┐
       │   CGRA Array      │
       │                   │
       │ PE PE PE PE       │
       │ PE PE PE PE       │
       │ PE PE PE PE       │
       │ PE PE PE PE       │
       └───────────────────┘
                 │
                 ▼
             Results
```

This makes CGRA-like architectures interesting for:

* convolution
* matrix multiplication
* vector operations
* activation functions
* DSP kernels
* quantized neural-network inference

However, AI workloads are not automatically a perfect match for CGRAs.

Memory bandwidth, data movement, irregular operations, control flow and mapping overhead can become limiting factors.

Therefore the architecture and compiler must be designed together.

---

## 12. The Role of the Compiler

A CGRA is only useful if applications can be mapped onto it efficiently.

This creates a hardware/software co-design problem.

A simplified compilation flow is:

```text
Application
     │
     ▼
Intermediate Representation
     │
     ▼
Data Flow Graph
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
     │
     ▼
CGRA
```

This is one of the most important concepts for the `cgra-lab` project.

The project should therefore not become only an RTL implementation.

The longer-term objective will be to explore the complete stack:

```text
Application
     ↓
Model / Algorithm
     ↓
Compiler / Mapper
     ↓
Configuration
     ↓
CGLA
     ↓
FPGA
     ↓
Measured performance
```

---

## 13. Research Questions for cgra-lab

The project will investigate the following questions:

1. What architectural features make a CGRA efficient?
2. How should processing elements be designed?
3. What interconnect topology provides a useful balance between flexibility and hardware cost?
4. How should computation be mapped onto the PE array?
5. How much of the mapping process can be automated?
6. How should memory be organized?
7. Which AI workloads are a good match for a small CGRA?
8. What are the performance and energy trade-offs?
9. How does a CGRA compare with an FPGA implementation of the same kernel?
10. Can a simple linear-array architecture provide a useful foundation for an AI accelerator?

The final question motivates the experimental architecture explored in this repository.

---

## 14. From CGRA to CGLA

In this project, **CGLA (Coarse-Grained Linear Array)** is used as the name of an experimental architecture derived from CGRA principles.

CGLA is not intended to represent a universally standardized architectural category.

Instead, it describes the project's specific design direction:

```text
PE → PE → PE → PE → PE → PE → PE → PE
```

The goal is to investigate whether a simpler one-dimensional architecture can provide:

* low implementation complexity,
* predictable communication,
* efficient streaming,
* simple configuration,
* low hardware overhead,
* and sufficient computational density for selected AI workloads.

The linear topology is deliberately chosen as a first experimental point in the CGRA design space rather than as a claim that linear arrays are generally superior to two-dimensional or more highly connected architectures.

The CGLA architecture will therefore be treated as an experimental research platform rather than as a replacement definition for CGRA.

---

## 15. Planned Research Path

The project will proceed in several stages:

```text
CGRA Research
      │
      ▼
Architecture Exploration
      │
      ▼
CGLA Architecture
      │
      ▼
RTL Prototype
      │
      ▼
FPGA Implementation
      │
      ▼
Mapping & Configuration
      │
      ▼
AI Workloads
      │
      ▼
Edge AI Demonstration
```

The initial implementation will focus on small computational kernels rather than a complete neural-network model.

Possible progression:

```text
1. Integer arithmetic
2. Vector operations
3. Matrix multiplication
4. MAC arrays
5. Neural-network kernels
6. Quantized ML models
7. ONNX-based workloads
8. Small edge-AI applications
9. More complex models
```

A model such as Moonshine may eventually be investigated, but only after the underlying architecture, memory system, compiler/mapping flow and AI kernels have been validated.

---

## 16. Summary

CGRA architectures occupy a design space between programmable processors and fixed-function accelerators, combining spatial computation with reconfigurable hardware resources.

Their key characteristics include:

* spatial computation,
* coarse-grained processing elements,
* configurable interconnect,
* reconfigurable datapaths,
* parallel execution,
* and compiler-driven mapping.

The most important challenge is not simply building an array of PEs.

The real challenge is designing the complete system:

> **Architecture + Memory + Interconnect + Mapping + Compiler + Workload**

This project will use these principles as the foundation for exploring a small, practical CGLA-based accelerator.

## Initial References

* Wijtvliet, M., Waeijen, L., & Corporaal, H. — *Coarse grained reconfigurable architectures in the past 25 years: overview and classification*, SAMOS 2016/2017.
* Choi, K. — *Coarse-Grained Reconfigurable Array: Architecture and Application Mapping*, IPSJ Transactions on System and LSI Design Methodology, 2011.
* Podobas, A., Sano, K., & Matsuoka, S. — *A Survey on Coarse-Grained Reconfigurable Architectures from a Performance Perspective*, 2020.
* Du, Y.-M. et al. — *Survey of Coarse-Grained Reconfigurable Architectures Mapping Algorithms*, Journal of Computer Science and Technology, 2026.
