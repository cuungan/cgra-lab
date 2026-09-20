# CGRA Research References

This document collects the main academic and technical references used by the `cgra-lab` project.

The references are organized by topic so that the research can evolve from CGRA fundamentals toward architecture design, mapping, compilation and AI acceleration.

---

## 1. CGRA Architecture and Classification

### [1] Wijtvliet, Waeijen & Corporaal — CGRA Classification

M. Wijtvliet, L. Waeijen, and H. Corporaal,
**"Coarse grained reconfigurable architectures in the past 25 years: overview and classification,"**
2016 International Conference on Embedded Computer Systems: Architectures, Modeling and Simulation (SAMOS), 2016/2017, pp. 235–244.

DOI: https://doi.org/10.1109/SAMOS.2016.7818353

This work provides an overview and classification of CGRA architectures and is useful for understanding the architectural design space.

---

### [2] Choi — Architecture and Application Mapping

K. Choi,
**"Coarse-Grained Reconfigurable Array: Architecture and Application Mapping,"**
IPSJ Transactions on System and LSI Design Methodology, Vol. 4, 2011, pp. 31–46.

DOI: https://doi.org/10.2197/ipsjtsldm.4.31

This paper provides an introduction to CGRA architectures and application mapping approaches.

---

## 2. CGRA Performance and Design Space

### [3] Podobas, Sano & Matsuoka — CGRA Performance Survey

A. Podobas, K. Sano, and S. Matsuoka,
**"A Survey on Coarse-Grained Reconfigurable Architectures From a Performance Perspective,"**
IEEE Access, Vol. 8, 2020, pp. 146719–146743.

DOI: https://doi.org/10.1109/ACCESS.2020.3012084

arXiv: https://arxiv.org/abs/2004.04509

This survey covers nearly three decades of CGRA research and analyzes architectural and performance characteristics across different designs.

---

## 3. CGRA Mapping

### [4] Du et al. — CGRA Mapping Algorithms Survey

Y.-M. Du, K.-Y. Chang, J.-L. Shi, Y.-Q. Zhou, T.-H. Zheng, Z. Wang, and R. Hong,
**"Survey of Coarse-Grained Reconfigurable Architectures Mapping Algorithms,"**
Journal of Computer Science and Technology, Vol. 41, No. 2, 2026, pp. 742–760.

DOI: https://doi.org/10.1007/s11390-026-5611-4

This is a particularly relevant recent reference for `cgra-lab`.

The paper analyzes CGRA mapping as the relationship between application tasks and processing elements and discusses scheduling, placement and routing approaches.

It also reviews machine-learning-related mapping approaches.

---

## 4. CGRA Compilation

### [5] Wang et al. — MLIR-Based CGRA Compilation

Y. Wang, C. Tirelli, G. Ansaloni, L. Pozzi, and D. Atienza,
**"An MLIR-based Compilation Framework for Control Flow Management on CGRAs,"**
arXiv preprint, 2025.

arXiv: https://arxiv.org/abs/2508.02167

This work is relevant to the compiler direction of the project because it investigates MLIR-based compilation and control-flow management for CGRAs.

The project may use this type of compiler infrastructure as inspiration for a future mapping/compiler flow.

---

## 5. Research Topics Derived from the Literature

The references above motivate the following research areas for `cgra-lab`:

### Architecture

* Processing Element design
* PE functionality
* register organization
* local memory
* interconnect topology
* configuration mechanism
* homogeneous vs heterogeneous arrays

### Mapping

* Data Dependence Graphs
* scheduling
* placement
* routing
* resource constraints
* modulo scheduling
* spatial mapping
* temporal mapping

### Compiler

* intermediate representations
* graph-based representations
* MLIR
* compiler passes
* hardware-aware optimization
* configuration generation

### AI Acceleration

* matrix multiplication
* MAC operations
* convolution
* vector operations
* quantized neural networks
* ONNX operators
* edge-AI workloads

---

## 6. Planned Reference Categories

As the project develops, additional references will be added under the following categories:

```text
CGRA Architecture
       │
       ├── PE design
       ├── Interconnect
       ├── Memory
       └── Configuration
       
Mapping
       │
       ├── Scheduling
       ├── Placement
       ├── Routing
       └── Optimization

Compiler
       │
       ├── LLVM
       ├── MLIR
       ├── IR design
       └── Code generation

AI Acceleration
       │
       ├── CNN
       ├── Transformers
       ├── Matrix multiplication
       ├── Quantization
       └── Edge AI

CGLA
       │
       ├── Linear-array architectures
       ├── Dataflow
       ├── Streaming
       └── Low-power architectures
```

---

## 7. Notes

The goal of this reference collection is not to maximize the number of citations.

Instead, the project will prioritize:

1. peer-reviewed research,
2. original architecture papers,
3. high-quality surveys,
4. recent compiler and mapping research,
5. reproducible implementations where available.

CGLA will be treated as an experimental architecture developed within this project rather than as a claim that "CGLA" is a universally standardized CGRA category.
