# FreqMambaGAN：一种用于水下图像增强的频率解耦 Mamba 增强 CycleGAN

<p align="center">
  <strong>面向审稿人的 FreqMambaGAN 论文配套学术仓库</strong>
</p>

<p align="center">
  <a href="LICENSE"><img alt="License: AGPL-3.0" src="https://img.shields.io/badge/License-AGPL--3.0-blue.svg"></a>
  <img alt="Task" src="https://img.shields.io/badge/Task-Underwater%20Image%20Enhancement-informational">
  <img alt="Status" src="https://img.shields.io/badge/Status-Reviewer%20Supplement-lightgrey">
</p>

## 概述

**FreqMambaGAN** 是一种用于水下图像增强的频率解耦、Mamba 增强循环对抗框架。该方法旨在处理常见的水下图像退化问题，包括颜色偏移、低对比度、散射引起的雾化效应，以及纹理和细节衰减。

该框架遵循 CycleGAN 风格的双向图像转换范式，并引入频率感知的生成器瓶颈结构。低频分量用于引导颜色与光照恢复，高频分量用于支持纹理和结构细节恢复。框架采用选择性状态空间建模，以较高计算效率增强长程依赖建模能力。此外，跳跃注意力重建和多阶段优化策略被进一步用于提升训练稳定性和视觉合理性。

本仓库主要作为**同行评审补充材料**提供。其目标是帮助审稿人理解算法流程，同时避免公开私有源代码、专利敏感工程细节、固定超参数配置、数据集路径或具体实现算子。

## 仓库内容

| 文件 | 说明 |
|---|---|
| `README.md` | 仓库级介绍、使用说明和范围声明。 |
| `FreqMambaGAN_secure_academic_pseudocode.md` | 面向审稿人的符号化伪代码，用于描述所提出方法。 |
| `LICENSE` | 仓库许可证。 |

## 方法亮点

- **循环一致的双向图像转换**：在成对和非成对训练设置下，学习退化水下图像与参考质量图像之间的映射关系。
- **频率解耦特征恢复**：将编码特征分解为低频和高频分量，从而将全局颜色/光照校正与局部纹理/细节恢复解耦。
- **Mamba 增强特征建模**：采用选择性状态空间建模抽象，以高效方式捕获长程依赖关系。
- **跳跃注意力重建**：在编码器—解码器重建过程中保留浅层空间细节。
- **多阶段优化**：结合监督预热、循环对抗学习、感知/全变分正则化、成对锚点引导和轻量级物理启发约束。
- **审稿安全披露**：展示公开的学术逻辑，同时抽象处理专利敏感的实现细节。

## 算法流程

FreqMambaGAN 的高层流程可概括如下：

```text
输入退化水下图像
        │
        ▼
公开预处理协议
        │
        ▼
编码器：由浅到深的特征提取
        │
        ▼
频率解耦 Mamba 瓶颈
        ├── 低频分支：颜色与光照建模
        └── 高频分支：纹理与结构建模
        │
        ▼
受保护的特征融合
        │
        ▼
带跳跃注意力的解码器重建
        │
        ▼
增强后的水下图像
```

详细的符号化训练和推理流程见：

> [`FreqMambaGAN_secure_academic_pseudocode.md`](FreqMambaGAN_secure_academic_pseudocode.md)

## 审稿人阅读建议

用于同行评审时，建议按以下顺序阅读：

1. 阅读论文的方法部分，理解架构设计动机。
2. 打开 `FreqMambaGAN_secure_academic_pseudocode.md`，查看符号化算法流程。
3. 将伪代码与论文中的图 2–6 以及第 2.1–2.6 节进行对照。
4. 利用流程说明和符号表，核查生成器、判别器、频率解耦瓶颈、跳跃注意力重建、损失项和多阶段训练过程的作用。

伪代码有意采用符号化学术表达，而不是可执行编程语法。这种设计可以提高审稿人的阅读清晰度，同时避免泄露专有工程细节。

## 保密性与范围说明

本仓库**不包含**以下内容：

- 可执行的训练或推理源代码；
- 数据集目录或私有数据路径；
- 已训练模型检查点；
- 固定数值超参数；
- 硬件相关加速配置；
- 私有算子或实现层面的优化规则；
- 能够直接复现专利敏感组件的源码级细节。

相应地，本仓库提供的是对公开算法逻辑的清晰、结构化学术描述，包括输入输出流程、模块关系、优化阶段以及基于验证结果的模型选择策略。

## 可复现性声明

该伪代码旨在从概念和方法层面提升所提出框架的透明度。出于知识产权保护考虑，精确的工程配置、实现相关优化和受保护的内部模块未予公开。因此，实验结果应结合论文正文共同理解；论文中报告了数据集、评价指标和比较实验结果。

## 引用

如果本工作对您的研究有帮助，请在论文正式发表后引用该文。下面提供的 BibTeX 条目仅为占位格式，应在正式发表后使用最终出版信息进行更新。

```bibtex
@article{FreqMambaGAN,
  title   = {FreqMambaGAN: A Frequency-Decoupled Mamba-Enhanced CycleGAN for Underwater Image Enhancement},
  author  = {Ye, Baojiang and Wang, Haifeng and Wang, Wenbin and Wang, Tianyi},
  journal = {Journal of Marine Science and Engineering},
  year    = {2025},
  note    = {Manuscript under peer review}
}
```

## 许可证

本仓库依据 `LICENSE` 文件中提供的许可证发布。

## 联系方式

如有学术问题，请参考论文中给出的通讯作者信息。
