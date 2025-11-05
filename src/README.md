# Source Code: Engineering Contributions

This directory contains the modified Python source files that demonstrate the key engineering enhancements I implemented for the DeepFaceLab MVE framework. These are not the complete framework, but rather targeted code contributions that solve specific performance and workflow problems.

---

### 1. `MergeMasked.py` (Bi-Directional Masking & QoL Overhaul)

*   **Original Purpose:** This file contains the core logic for the interactive face merging process.
*   **My Enhancements:** I implemented a complete overhaul of the mask modification system to improve precision and user experience. My key contributions include:
    *   **Bi-Directional Control System:** Engineered the core logic for independent, per-axis expansion and erosion of the mask.
    *   **Bug Fixes:** Diagnosed and fixed a bug in the underlying OpenCV implementation that caused inverted and unpredictable control behavior.
    *   **Exclusion Zone Preservation:** Added logic to ensure manually drawn exclusion masks are respected during the expansion process.
    *   **Ergonomic Key Remapping:** (Implemented in `InteractiveMergerSubprocessor.py`, but driven by the features in this file) Redesigned the control scheme for a faster, more intuitive workflow.

---

### 2. `Model.py` (XSeg Performance Optimization)

*   **Original Purpose:** This file defines the model architecture and data loading pipeline for the XSeg face segmentation trainer.
*   **My Enhancement:** I identified and resolved a critical CPU bottleneck that was causing the GPU to be underutilized.
    *   **Performance Optimization:** I refactored the data loader, removing an artificial 8-core cap and implementing new logic to intelligently allocate `(Total CPU Cores - 2)` processes for data generation. On a 16-core CPU, this resulted in a **3.5x increase in data generation parallelism**, leading to smoother and faster training.
