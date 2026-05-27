# FreqMambaGAN: A Frequency-Decoupled Mamba-Enhanced CycleGAN for Underwater Image Enhancement

<p align="center">
  <strong>Reviewer-facing academic repository for the FreqMambaGAN manuscript</strong>
</p>


<p align="center">
  <a href="LICENSE"><img alt="License: AGPL-3.0" src="https://img.shields.io/badge/License-AGPL--3.0-blue.svg"></a>
  <img alt="Task" src="https://img.shields.io/badge/Task-Underwater%20Image%20Enhancement-informational">
  <img alt="Status" src="https://img.shields.io/badge/Status-Reviewer%20Supplement-lightgrey">
</p>


## Overview

**FreqMambaGAN** is a frequency-decoupled, Mamba-enhanced cycle-adversarial framework for underwater image enhancement. The method is designed to address common underwater degradation patterns, including color cast, low contrast, scattering-induced haze, and texture/detail attenuation.

The framework follows a CycleGAN-style bidirectional translation paradigm and introduces a frequency-aware generator bottleneck. Low-frequency components are used to guide color and illumination restoration, while high-frequency components support texture and structural detail recovery. Selective state-space modeling is adopted to improve long-range dependency modeling with a computationally efficient design. Skip-attention reconstruction and multi-stage optimization are further used to improve stability and visual plausibility.

This repository is provided primarily as a **peer-review supplement**. It helps reviewers understand the algorithmic workflow without exposing private source code, patent-sensitive engineering details, fixed hyperparameter settings, dataset paths, or implementation-specific operators.

## Repository Contents

| File                                         | Description                                                  |
| -------------------------------------------- | ------------------------------------------------------------ |
| `README.md`                                  | Repository-level introduction, usage guidance, and scope statement. |
| `FreqMambaGAN_secure_academic_pseudocode.md` | Symbolic, reviewer-facing pseudocode of the proposed method. |
| `LICENSE`                                    | Repository license.                                          |

## Method Highlights

- **Cycle-consistent bidirectional translation**: learns mappings between degraded underwater images and reference-quality images under paired and unpaired training settings.
- **Frequency-decoupled feature restoration**: separates encoded features into low-frequency and high-frequency components to decouple global color/illumination correction from local texture/detail restoration.
- **Mamba-enhanced feature modeling**: uses selective state-space modeling abstractions to capture long-range dependencies efficiently.
- **Skip-attention reconstruction**: preserves shallow spatial details during encoder-decoder reconstruction.
- **Multi-stage optimization**: combines supervised warm-up, cycle-adversarial learning, perceptual/TV regularization, paired anchor guidance, and lightweight physics-inspired constraints.
- **Reviewer-safe disclosure**: presents the public academic logic while abstracting patent-sensitive implementation details.

## Algorithmic Workflow

The high-level workflow of FreqMambaGAN can be summarized as follows:

```text
Input degraded underwater image
        │
        ▼
Public preprocessing protocol
        │
        ▼
Encoder: shallow-to-deep feature extraction
        │
        ▼
Frequency-decoupled Mamba bottleneck
        ├── Low-frequency branch: color and illumination modeling
        └── High-frequency branch: texture and structural modeling
        │
        ▼
Protected feature fusion
        │
        ▼
Decoder with skip-attention reconstruction
        │
        ▼
Enhanced underwater image
```

The detailed symbolic training and inference procedures are provided in:

> [`FreqMambaGAN_secure_academic_pseudocode.md`](FreqMambaGAN_secure_academic_pseudocode.md)

## Reviewer Guidance

For peer review, the recommended reading order is:

1. Read the method section of the manuscript to understand the architectural motivation.
2. Open `FreqMambaGAN_secure_academic_pseudocode.md` to inspect the symbolic algorithmic workflow.
3. Compare the pseudocode with Figures 2-6 and Sections 2.1-2.6 of the manuscript.
4. Use the workflow and symbol table to verify the roles of the generator, discriminator, frequency-decoupled bottleneck, skip-attention reconstruction, loss terms, and multi-stage training phases.

The pseudocode is intentionally written in a symbolic academic style rather than executable programming syntax. This design improves readability for reviewers while avoiding disclosure of proprietary engineering details.

## Confidentiality and Scope

This repository does **not** include:

- executable training or inference source code;
- dataset directories or private data paths;
- trained model checkpoints;
- fixed numerical hyperparameters;
- hardware-specific acceleration settings;
- private operators or implementation-level optimization rules;
- source-level details that would allow direct reproduction of patent-sensitive components.

Instead, it provides a clear and structured academic description of the public algorithmic logic, including the input-output flow, module relationships, optimization stages, and validation-based model selection strategy.

## Reproducibility Statement

The pseudocode is intended to improve the transparency of the proposed framework at the conceptual and methodological levels. Exact engineering configurations, implementation-specific optimizations, and protected internal modules are withheld for intellectual-property protection. Experimental results should therefore be interpreted together with the manuscript, where datasets, evaluation metrics, and comparative results are reported.

## Citation

If this work is useful for your research, please cite the manuscript after publication. A placeholder BibTeX entry is provided below and should be updated with the final publication metadata.

```bibtex
@article{FreqMambaGAN,
  title   = {FreqMambaGAN: A Frequency-Decoupled Mamba-Enhanced CycleGAN for Underwater Image Enhancement},
  author  = {Ye, Baojiang and Wang, Haifeng and Wang, Wenbin and Wang, Tianyi},
  journal = {Journal of Marine Science and Engineering},
  year    = {2025},
  note    = {Manuscript under peer review}
}
```

## License

This repository is released under the license provided in the `LICENSE` file.

## Contact

For academic questions, please refer to the corresponding author information provided in the manuscript.
