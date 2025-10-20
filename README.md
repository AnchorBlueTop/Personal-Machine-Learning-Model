<h1 align="center">AI Creative Production: An Engineering Case Study</h1>

<p align="center">
  <a href="https://arxiv.org/abs/2005.05535">
    <img src="https://static.arxiv.org/static/browse/0.3.0/images/icons/favicon.ico" width=14></img>
    This project is built upon the foundational framework of DeepFaceLab, detailed in arXiv:2005.05535.
  </a>
</p>

---

### 1. Executive Summary

This repository is a comprehensive engineering case study of the custom ecosystem I built for producing professional-grade AI digital art. My creative work, which involves self-insertion deepfakes into iconic Bollywood scenes for my **[@wholeface_](https://www.instagram.com/wholeface_)** portfolio (9.5k+ followers), demanded a level of performance and stability that off-the-shelf tools could not provide.

To solve this, I engineered a complete, end-to-end solution spanning custom hardware, a meticulously documented production workflow, and a data-driven model training methodology. This document details the engineering decisions, technical processes, and expert knowledge developed over hundreds of hours of hands-on work.

![resized intro](https://github.com/AnchorBlueTop/Personal-Machine-Learning-Model/assets/98157644/8d3a0b5b-c235-4103-aced-795cfdf86a56)

---

### 2. The System Architecture: A No-Compromises ML Ecosystem

To enable the intensive, multi-day training sessions required for high-fidelity output, I designed and built a specialized hardware and software environment from the ground up.

#### Hardware Foundation: The ML Workstation

The system was engineered for a single purpose: maximum, sustained performance and thermal stability for 24/7 operation.

| Component | Specification | Engineering Purpose |
|---|---|---|
| **CPU** | AMD Ryzen 9 9950X3D (Delidded)| AVX-512 support for accelerated CPU-bound tasks like the SOT color transfer algorithm during the final merge process. |
| **GPU** | NVIDIA RTX 5090 (32GB) | Maximum available VRAM to enable training of higher-resolution models and larger batch sizes. |
| **Cooling**| Custom Water Loop w/ External MO-RA IV Radiator | To decouple heat dissipation from the workstation. The external radiator, wall-mounted in an adjacent room, allows for silent, continuous, and stable operation without thermal throttling, which is critical for multi-day training runs. |

*(Images of the PC build and external MO-RA setup will be included here to visually showcase the engineering effort.)*

---

### 3. Engineering Contributions: Workflow & Performance Enhancements

My deep involvement with this workflow naturally led me to identify and solve critical inefficiencies and bugs within the original DeepFaceLab software. I have moved beyond being a user to actively engineering improvements to the toolset. My contributions focus on reducing manual labor, improving user experience, and optimizing performance.

#### Bi-Directional Masking: A Quality-of-Life Overhaul
*   **Problem:** The default mask controls were unintuitive and lacked the fine-grained control needed to handle complex head shapes and occlusions, leading to background bleed and poor mask boundaries.
*   **Solution:** I implemented a complete overhaul of the mask modification system. This included:
    1.  **Developing a Bi-Directional Control System:** I engineered a feature that allows for independent, per-axis expansion and erosion of the mask (top, bottom, left, right).
    2.  **Fixing Core Logic Bugs:** I diagnosed and fixed a bug in the underlying OpenCV implementation that caused inverted and unpredictable control behavior.
    3.  **Preserving Exclusion Zones:** I added logic to ensure that manually drawn exclusion masks (e.g., for glasses or beards) are respected and preserved during the expansion process.
    4.  **Ergonomic Key Remapping:** I redesigned the entire control scheme for the interactive merger, creating an intuitive, clustered key layout that significantly improves workflow speed.
*   **Impact:** This enhancement transforms a frustrating process into a powerful, precise tool, enabling the creation of much cleaner and more realistic masks.

    *(Here you will insert 1-2 images showcasing the results of the new bi-directional masking.)*

#### XSeg Training Pipeline: Eliminating CPU Bottlenecks
*   **Problem:** While analyzing training performance, I identified a severe producer-consumer bottleneck in the XSeg training pipeline. The GPU was frequently idle ("starving") while waiting for the CPU to prepare data batches, indicated by a cyclical fast-slow-slow iteration pattern.
*   **Solution:** I refactored the data loader for the XSeg model by removing an artificial 8-core cap and implementing new logic to intelligently allocate `(Total CPU Cores - 2)` processes for data generation.
*   **Impact:** On my 16-core CPU, this increased data generation parallelism by **3.5x** (from 4 to 14 processes), completely eliminating the GPU starvation issue and leading to smoother, faster training performance.

---

### 4. The End-to-End Production Workflow (150+ Hours Per Project)

Each video is the result of a painstaking, multi-stage production pipeline that I have refined for quality and consistency.

1.  **Data Acquisition (MP4 to PNG):** Source and destination video clips are carefully selected and decomposed into high-quality PNG image sequences.
2.  **Face Extraction & Alignment:** Faces are extracted from every frame. This is the most labor-intensive stage.
    *   **Manual XSeg Polygon Drawing:** I manually draw precise segmentation polygons around the destination face. This often requires **hours of meticulous work per video**, with complex scenes sometimes requiring over **2,000 individual face alignments**.
3.  **XSeg Model Training:** A dedicated XSeg segmentation model is trained on the manually labeled data to produce high-quality, context-aware masks.
4.  **SAEHD Model Training:** The core deepfake model is trained. This is a multi-day, data-driven process where I make critical decisions based on performance metrics. *(See Section 5 for a detailed breakdown).*
5.  **Merging & Compositing:** The trained model is used to merge my face onto the destination frames, followed by final color grading and video assembly.

---

### 5. The Art of Model Training: An Engineering Methodology

Training is not a "fire-and-forget" process. It is an iterative cycle of analysis and intervention, where I make informed, data-driven decisions based on performance metrics to guide the model toward an optimal result.

#### How to Make Informed Decisions During Training

Knowing when to proceed to the next phase comes down to looking at the face previews and interpreting model loss values. The model loss usually follows a logarithmic trend downwards during the initial generalization phase before stagnating during the refinement and final sharpening phases.

![Loss Curve Graph](https://github.com/AnchorBlueTop/Personal-Machine-Learning-Model/assets/98157644/341e3a21-cd55-4cda-960c-3043a56717f4)

The entire training process is a balancing act between two key metrics, which are logged continuously:

![Loss Value Log](https://github.com/AnchorBlueTop/Personal-Machine-Learning-Model/assets/98157644/5899a485-cdb6-47b1-bed6-be0a89458dfb)

*   **SRC (Source) Loss:** This measures how accurately the model can reconstruct my own face. A lower value means the model is getting better at reproducing my specific facial features.
*   **DST (Destination) Loss:** This measures how accurately the model can reconstruct the face of the actor in the target video. A lower value indicates better performance in capturing the target's expressions and head movements.

Essentially, once the loss values no longer improve at a significant rate (I typically use a threshold of less than **0.010 improvement over a 25-minute interval**), and there isn't a notable difference in the visual previews, it's time to move to the next step or phase. This data-driven approach is critical to avoid overfitting and achieve a high-quality result.

#### My Three-Phase Training Process

My methodology is broken down into three distinct phases, with progression between them governed by the data described above.

##### Phase 1: Generalization (Random Warp Enabled)
*   **Objective:** To teach the model the general structure, expressions, and angles of the faces. Fine detail is not the priority.
*   **Key Settings:** `Random Warp (RW)`: **Enabled**, `Flip DST faces randomly`: **Enabled**.

##### Phase 2: Refinement & Detail Acquisition (Random Warp Disabled)
*   **Objective:** To learn the fine details, textures, and subtle nuances of the face.
*   **Key Settings & Sub-stages:**
    1.  **Initial Refinement:** `RW` is **disabled**, `LRD` is **enabled**, and `MS-SSIM` loss function is introduced.
    2.  **Pose Refinement:** `Uniform Yaw (UY)` is enabled for several hours to improve side profiles.
    3.  **Feature Priority:** `Eyes and Mouth Priority (EMP)` is selectively enabled to fix specific blurriness, used with caution to avoid artifacts.

##### Phase 3: Final Sharpening (GAN)
*   **Objective:** To apply a final layer of sharpness and realism.
*   **Execution:** `GAN` is enabled at a low power setting (`0.1`) for a very short duration (**10-15 minutes**) to avoid over-sharpening artifacts.

--- 

### 6. Model Architectures & Parameters: An Empirical Analysis

There is no single "best" model. The optimal architecture is a delicate balance of resolution, model complexity (dimensions), VRAM usage, and training time. Through hundreds of hours of empirical testing, I have found that the `LIAE-UDT` architecture provides a strong balance of source face likeness and adaptation to destination lighting.

However, I've learned that while model parameters can be fine-tuned, the single most important factor for a high-quality result is well-matched, high-quality lighting in both the source and destination facesets.

#### Understanding the Key Parameters

My training methodology is based on a deep, practical understanding of how each parameter affects the final output.

*   **Autoencoder Dimensions (AE Dims):** This can be thought of as the model's total "knowledge capacity"—the size of its internal library for facial features. A larger AE allows the model to store a more diverse range of information.

*   **Encoder/Decoder Dimensions (E/D Dims):** If the Autoencoder is the library, the Encoder and Decoder are the "PhD-level readers and writers" that access it. The Encoder "reads" a face and summarizes it for the library, while the Decoder "writes" a new face based on that summary. Higher dimensions allow them to capture and reproduce more intricate details and nuances.

*   **Batch Size:** This is critical for training stability. Through extensive testing, I have found that a batch size below 8 often leads to training instability and poor quality outputs, such as misaligned or missing eyes.

#### Model Configurations & Findings

Below are my most frequently used and tested model configurations, representing a history of my work and hardware progression.

##### Primary Production Models (RTX 4090)
*This library of models was developed on the RTX 4090. The 480p model, with over 4.8 million iterations, represents my most refined and stable configuration on that platform.*

| Resolution | **480x480 (Main)** | 512x512 |
|---|---|---|
| **Architecture** | LIAE-UDT | DF-UD |
| **AE Dims** | 292 | 256 |
| **Encoder Dims**| 78 | 72 |
| **Decoder Dims**| 78 | 72 |
| **Mask Dims** | 32 | 18 |
| **Batch Size** | 10 | 10 |

##### High-Performance Models (RTX 5090)
*These models leverage the increased VRAM of the new hardware to push for higher resolutions and model complexity.*

| Parameter | High Res (Balanced) | High Res (Testing) |
|---|---|---|
| **Resolution** | 544x544 | 544x544 |
| **AE Dims** | 394 | 402 |
| **Encoder Dims** | 80 | 98 |
| **Decoder Dims** | 80 | 98 |
| **Mask Dims**| 32 | 32 |
| **Batch Size** | 10 | 10 |

##### A Note on the AdaBelief Optimizer
While the AdaBelief optimizer can sometimes accelerate the initial learning phase, I have concluded that its severe VRAM penalty and workflow friction make it suboptimal for my process. Reusing a model trained with AdaBelief on a new target often requires restarting the training from scratch, which negates the benefits of a well-trained base model. For these reasons, I currently favor the standard RMSprop optimizer for its stability and reusability.

#### Training Previews & Results

The following images are examples of the output quality achieved through my refined training methodology.

![Preview 1](https://github.com/AnchorBlueTop/Personal-Machine-Learning-Model/assets/98157644/2026d10e-061a-4c12-96eb-2ca999f0be03)
![Preview 2](https://github.com/AnchorBlueTop/Personal-Machine-Learning-Model/assets/98157644/a7bb75a2-ec37-4460-8633-c591cd110d37)
![Preview 3](https://github.com/AnchorBlueTop/Personal-Machine-Learning-Model/assets/98157644/6a97dd3d-abf4-4327-9a08-3c6412fedadb)

---
