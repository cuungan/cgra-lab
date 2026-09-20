CGRA Lab

Exploring Coarse-Grained Reconfigurable Architectures (CGRA) and building a CGLA (LA here is Linear Array) based AI accelerator prototype on FPGA.

Project Goal

This project explores the design principles, architecture, programming models, and implementation challenges of CGRAs, with a particular focus on low-power AI acceleration.

The long-term goal is to develop a small experimental CGRA architecture and implement a CGLA prototype on FPGA.

The project will progress from architectural research to RTL implementation, mapping, and AI workloads.
```text
Roadmap
CGRA Research
      │
      ▼
Architecture Exploration
      │
      ▼
CGLA Architecture
      │
      ▼
FPGA RTL Prototype
      │
      ▼
Mapping & Configuration
      │
      ▼
AI Workloads
      │
      ▼
Edge AI Demonstration
Research Topics
What is a CGRA?
CGRA vs FPGA, GPU and ASIC
Processing Elements (PEs)
Interconnect architectures
Memory and data movement
Configuration models
Spatial computation
Scheduling and mapping
CGRA compilers
MLIR and accelerator compilation
AI/ML acceleration
Low-power accelerator architectures
CGLA
```
The CGLA architecture will serve as a simple experimental platform for exploring CGRA concepts.

Initial research will investigate:

Processing Element design
Linear PE arrays
Inter-PE communication
Configuration mechanisms
Dataflow
Local memory
FPGA implementation
Performance and resource utilization
AI Workloads

After validating the basic architecture, the project will investigate mapping representative AI workloads onto the accelerator.

Potential targets include:

Matrix and vector operations
Neural-network kernels
ONNX operators
Small ML models
Lightweight speech-processing workloads
Moonshine and similar compact speech-recognition models
Project Structure
```text
cgra-lab/
│
├── research/          # CGRA research and technical notes
├── architectures/     # Architecture definitions
│   └── cgla/
├── rtl/               # FPGA/RTL implementation
│   ├── pe/
│   ├── interconnect/
│   └── cgra/
├── simulation/        # Simulation and verification
├── compiler/          # Mapping and compilation experiments
├── models/            # AI models and model experiments
├── benchmarks/        # Performance and resource benchmarks
└── docs/              # Additional documentation
```
Status

Phase 0 — Research

The project is currently in the research and architecture exploration phase.

The first objective is to build a solid understanding of CGRA architectures through primary research papers, technical publications, and open-source implementations before starting the FPGA prototype.

References

A curated list of academic papers, books, technical resources, and open-source projects will be maintained in:

research/references.md

Author: Cahit U. Ungan
Focus: FPGA · Embedded AI · Computer Architecture · AI Accelerators
