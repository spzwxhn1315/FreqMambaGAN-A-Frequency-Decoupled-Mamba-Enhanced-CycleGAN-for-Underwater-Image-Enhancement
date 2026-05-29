# FreqMambaGAN Secure Academic Pseudocode

This document provides a reviewer-facing pseudocode description of FreqMambaGAN. It is intended to clarify the end-to-end algorithmic workflow without exposing implementation-specific source code, private operators, fixed numerical settings, dataset paths, or patent-sensitive engineering details. All tunable quantities are expressed symbolically.

## Confidentiality and Scope Statement

The pseudocode below describes only the public academic logic of the method: bidirectional domain translation, frequency-decoupled feature restoration, selective state-space feature modeling, skip-attention reconstruction, multi-phase optimization, and validation-based checkpoint selection. Proprietary low-level implementations, engineering optimizations, private iteration rules, hardware-specific acceleration details, and reproducible source-level module internals are intentionally abstracted as generic mathematical operators.

## Symbol Definition Table

| Symbol                            | Academic meaning                                             |
| --------------------------------- | ------------------------------------------------------------ |
| $A$                               | Degraded underwater image domain                             |
| $B$                               | Clear or reference-quality underwater image domain           |
| $x_A$                             | Sample drawn from domain $A$                                 |
| $x_B$                             | Sample drawn from domain $B$                                 |
| $\hat{x}_B$                       | Enhanced output generated from $x_A$                         |
| $\hat{x}_A$                       | Degraded-domain output generated from $x_B$                  |
| $\widetilde{x}_A$                 | Cycle-reconstructed sample in domain $A$                     |
| $\widetilde{x}_B$                 | Cycle-reconstructed sample in domain $B$                     |
| $\mathcal{D}_{AB}^{p}$            | Paired degraded/reference training set                       |
| $\mathcal{D}_{A}^{u}$             | Unpaired degraded-domain training set                        |
| $\mathcal{D}_{B}^{u}$             | Unpaired clear-domain training set                           |
| $\mathcal{V}_{AB}^{p}$            | Paired validation set                                        |
| $\mathcal{V}_{A}^{u}$             | Unpaired degraded-domain validation set                      |
| $G_{A\rightarrow B}$              | Forward enhancement generator                                |
| $G_{B\rightarrow A}$              | Reverse degradation-style generator                          |
| $D_A$                             | Discriminator for domain $A$                                 |
| $D_B$                             | Discriminator for domain $B$                                 |
| $\Theta_G$                        | Generator parameter set                                      |
| $\Theta_D$                        | Discriminator parameter set                                  |
| $\mathcal{E}$                     | Encoder abstraction                                          |
| $\mathcal{B}$                     | Protected frequency-decoupled Mamba bottleneck abstraction   |
| $\mathcal{R}$                     | Decoder or reconstruction abstraction                        |
| $\mathcal{A}_{s}$                 | Skip-attention operator                                      |
| $\mathcal{A}_{f}$                 | Frequency-branch fusion operator                             |
| $\mathcal{F}$                     | Centered orthonormal spectral transform                      |
| $\mathcal{F}^{-}$                 | Inverse centered orthonormal spectral transform              |
| $\mathcal{M}_{\delta}$            | Symbolic low-frequency selection mask controlled by threshold $\delta$ |
| $\overline{\mathcal{M}}_{\delta}$ | Complementary high-frequency selection mask                  |
| $\mathcal{S}_{g}$                 | Generic global selective state-space modeling operator       |
| $\mathcal{S}_{l}$                 | Generic local patch-wise selective state-space modeling operator |
| $\mathcal{P}$                     | Generic non-overlapping patch partition and restoration operator |
| $\Phi$                            | Current training phase                                       |
| $\mathcal{T}_{\Phi}$              | Symbolic iteration schedule of phase $\Phi$                  |
| $\alpha_G$                        | Generator learning-rate symbol                               |
| $\alpha_D$                        | Discriminator learning-rate symbol                           |
| $\omega$                          | Generic trainable parameter symbol                           |
| $\beta_r$                         | Soft real-label symbol in adversarial training               |
| $\beta_f$                         | Soft fake-label symbol in adversarial training               |
| $\kappa_D$                        | Symbolic discriminator update policy                         |
| $\lambda_{adv}^{\Phi}$            | Adversarial-loss weight in phase $\Phi$                      |
| $\lambda_{cyc}^{\Phi}$            | Cycle-consistency-loss weight in phase $\Phi$                |
| $\lambda_{id}^{\Phi}$             | Identity-loss weight in phase $\Phi$                         |
| $\lambda_{tv}^{\Phi}$             | Total-variation-loss weight in phase $\Phi$                  |
| $\lambda_{per}^{\Phi}$            | Perceptual-loss weight in phase $\Phi$                       |
| $\lambda_{anc}^{\Phi}$            | Paired-anchor-loss weight in phase $\Phi$                    |
| $\lambda_{dark}^{\Phi}$           | Dark-channel-consistency weight in phase $\Phi$              |
| $\lambda_{phy}^{\Phi}$            | Physics-guided-consistency weight in phase $\Phi$            |
| $\mathcal{L}_{adv}^{G}$           | Generator-side least-squares adversarial loss                |
| $\mathcal{L}_{adv}^{D}$           | Discriminator-side least-squares adversarial loss            |
| $\mathcal{L}_{cyc}$               | Bidirectional cycle-consistency loss                         |
| $\mathcal{L}_{id}$                | Bidirectional identity loss                                  |
| $\mathcal{L}_{tv}$                | Forward-direction total-variation regularization             |
| $\mathcal{L}_{per}$               | Forward-direction perceptual regularization                  |
| $\mathcal{L}_{anc}$               | Forward-direction paired absolute-error anchor loss          |
| $\mathcal{L}_{dark}$              | Dark-channel consistency regularization                      |
| $\mathcal{L}_{phy}$               | Simplified physics-inspired consistency regularization       |
| $S_q$                             | Validation image-quality score                               |
| $S_{phy}$                         | Validation physical-consistency score                        |
| $S_{save}$                        | Checkpoint-selection score                                   |
| $\epsilon$                        | Minimum symbolic improvement threshold for model saving      |

## Algorithm A: Reviewer-Facing Inference Workflow

```text
Input:
    Degraded underwater image x_A
    Forward generator G_{A→B} with parameters Θ_G

Output:
    Enhanced image \hat{x}_B

Procedure:
    Prepare x_A through the same public preprocessing protocol used in training.

    Encode shallow-to-deep spatial features:
        h = E(x_A)

    Apply the protected frequency-decoupled Mamba bottleneck:
        z = B(h)

    Reconstruct the enhanced image with decoder and skip-attention fusion:
        \hat{x}_B = R(z, A_s)

    Return \hat{x}_B.
```

## Algorithm B: Protected Frequency-Decoupled Mamba Bottleneck

```text
Input:
    Encoded feature tensor h
    Symbolic frequency threshold δ
    Generic global and local state-space operators S_g and S_l

Output:
    Frequency-aware bottleneck feature z

Procedure:
    Map h into a centered spectral representation:
        H = F(h)

    Apply a symbolic low-frequency mask in the centered spectral domain:
        H_low = M_δ(H)

    Recover the low-frequency feature in the spatial feature domain:
        h_low = F^-(H_low)

    Compute the high-frequency feature by residual subtraction:
        h_high = h ⊖ h_low

    Model high-frequency structural information with a generic global selective state-space operator:
        z_high = S_g(h_high)

    Model low-frequency color and illumination information with a generic local patch-wise selective state-space operator:
        z_low = P^-( S_l( P(h_low) ) )

    Fuse the frequency-specific features using a protected attention-style fusion rule:
        z_fuse = A_f(z_high, z_low)

    Preserve bottleneck-level residual information:
        z = z_fuse ⊕ h

    Return z.
```

The exact spectral mask construction, patch granularity, internal selective-scan realization, branch depth, and fusion implementation are deliberately not expanded. These components are represented only as public, high-level academic operators.

## Algorithm C: Secure Multi-Phase Training Procedure

```text
Input:
    Paired training set D_AB^p
    Unpaired training sets D_A^u and D_B^u
    Validation sets V_AB^p and V_A^u
    Generators G_{A→B}, G_{B→A}
    Discriminators D_A, D_B
    Symbolic phase schedules T_Φ
    Symbolic loss weights λ_*^Φ
    Symbolic optimization settings α_G, α_D, κ_D, ε

Output:
    Best validation-selected forward generator G_{A→B}^*

Procedure:
    Initialize G_{A→B}, G_{B→A}, D_A, and D_B.
    Initialize symbolic optimizers for Θ_G and Θ_D.
    Initialize best-score memory S_save^*.

    For each training phase Φ in {I, II-a, II-b, III}:

        Configure the trainable modules for phase Φ:
            Phase I:
                Train only the forward enhancement mapping G_{A→B}.
                Use paired degraded/reference samples.

            Phase II-a:
                Train the cycle-adversarial framework on unpaired samples.
                Keep selected shallow forward-generator feature extractors fixed for stabilization.
                Use paired samples only as an auxiliary forward anchor.

            Phase II-b:
                Continue cycle-adversarial learning with the full forward generator unfrozen.
                Use paired samples only as an auxiliary forward anchor.

            Phase III:
                Fine-tune the generators under fixed discriminator scoring.
                Keep selected state-space stability parameters fixed.
                Add physics-inspired consistency regularization only to the forward enhancement direction.

        For each symbolic iteration τ within T_Φ:

            If Φ is Phase I:
                Sample a paired mini-batch (x_A^p, x_B^p) from D_AB^p.
                Generate \hat{x}_B = G_{A→B}(x_A^p).
                Compute supervised absolute reconstruction loss:
                    L_G^Φ = L_abs(\hat{x}_B, x_B^p)
                Update Θ_G of G_{A→B} only.

            Otherwise:
                Sample unpaired mini-batches x_A^u from D_A^u and x_B^u from D_B^u.
                Optionally sample paired anchor mini-batch (x_A^p, x_B^p) from D_AB^p.

                Forward translation:
                    \hat{x}_B = G_{A→B}(x_A^u)

                Reverse translation:
                    \hat{x}_A = G_{B→A}(x_B^u)

                Cycle reconstruction:
                    \widetilde{x}_A = G_{B→A}(\hat{x}_B)
                    \widetilde{x}_B = G_{A→B}(\hat{x}_A)

                Identity mapping:
                    x_B^{id} = G_{A→B}(x_B^u)
                    x_A^{id} = G_{B→A}(x_A^u)

                Compute generator-side losses:
                    L_adv^G from D_B(\hat{x}_B) and D_A(\hat{x}_A)
                    L_cyc from (\widetilde{x}_A, x_A^u) and (\widetilde{x}_B, x_B^u)
                    L_id  from (x_B^{id}, x_B^u) and (x_A^{id}, x_A^u)
                    L_tv  from \hat{x}_B only
                    L_per from (\hat{x}_B, x_A^u) only
                    L_anc from (G_{A→B}(x_A^p), x_B^p) only when the paired anchor is used

                Combine the phase-dependent generator objective:
                    L_G^Φ = λ_adv^Φ L_adv^G
                            ⊕ λ_cyc^Φ L_cyc
                            ⊕ λ_id^Φ  L_id
                            ⊕ λ_tv^Φ  L_tv
                            ⊕ λ_per^Φ L_per
                            ⊕ λ_anc^Φ L_anc

                If Φ is Phase III:
                    Compute L_dark from \hat{x}_B.
                    Compute L_phy from (\hat{x}_B, x_A^u).
                    Extend the generator objective:
                        L_G^Φ = L_G^Φ ⊕ λ_dark^Φ L_dark ⊕ λ_phy^Φ L_phy

                Update generator parameters Θ_G using L_G^Φ.

                If Φ permits discriminator learning and the symbolic update policy κ_D is satisfied:
                    Compute discriminator-side least-squares adversarial loss:
                        L_adv^D = L_DA ⊕ L_DB
                    Update Θ_D using L_adv^D.
                Otherwise:
                    Keep discriminator parameters unchanged.

        After each validation interval of phase Φ:
            Evaluate paired validation quality:
                S_q = η_P Ψ_P(PSNR) ⊕ η_S Ψ_S(SSIM) ⊕ η_E Ψ_E(E_abs)

            If Φ is Phase III:
                Evaluate physics-oriented validation consistency:
                    S_phy = Ψ_phy(L_dark ⊕ L_phy)
                Compute final checkpoint score:
                    S_save = η_q S_q ⊕ η_φ S_phy
            Otherwise:
                Use image-quality score as the checkpoint score:
                    S_save = S_q

            If S_save improves over S_save^* by at least ε:
                Save the public forward enhancement generator as G_{A→B}^*.
                Update S_save^*.

    Return G_{A→B}^*.
```

## Public Loss Definitions Used in the Pseudocode

The following definitions are written at the level required for academic review and are not intended to reproduce the exact engineering implementation.

### Bidirectional Translation

$$
\hat{x}_B = G_{A\rightarrow B}(x_A),
\qquad
\hat{x}_A = G_{B\rightarrow A}(x_B)
$$

$$
\widetilde{x}_A = G_{B\rightarrow A}(\hat{x}_B),
\qquad
\widetilde{x}_B = G_{A\rightarrow B}(\hat{x}_A)
$$

### Generator-Side Objective

For the cycle-adversarial phases, the symbolic generator objective is:

$$
\mathcal{L}_{G}^{\Phi} = \lambda_{adv}^{\Phi}\mathcal{L}_{adv}^{G} + \lambda_{cyc}^{\Phi}\mathcal{L}_{cyc} + \lambda_{id}^{\Phi}\mathcal{L}_{id} + \lambda_{tv}^{\Phi}\mathcal{L}_{tv} + \lambda_{per}^{\Phi}\mathcal{L}_{per} + \lambda_{anc}^{\Phi}\mathcal{L}_{anc}.
$$

For the physics-guided fine-tuning phase, the objective is extended as:

$$
\mathcal{L}_{G}^{\Phi} \leftarrow \mathcal{L}_{G}^{\Phi} + \lambda_{dark}^{\Phi}\mathcal{L}_{dark} + \lambda_{phy}^{\Phi}\mathcal{L}_{phy}.
$$

Here, $\mathcal{L}_{tv}$, $\mathcal{L}_{per}$, $\mathcal{L}_{anc}$, $\mathcal{L}_{dark}$, and $\mathcal{L}_{phy}$ are applied to the forward enhancement direction $A\rightarrow B$ rather than symmetrically to both mappings.

### Checkpoint Selection

The symbolic image-quality score is expressed as:

$$
S_q = \eta_P\Psi_P(\mathrm{PSNR}) + \eta_S\Psi_S(\mathrm{SSIM}) + \eta_E\Psi_E(E_{\mathrm{abs}}).
$$

The symbolic physics-consistency score is expressed as:

$$
S_{phy}=\Psi_{phy}\bigl(\mathcal{L}_{dark}+\mathcal{L}_{phy}\bigr).
$$

The final model-selection score is:

$$
S_{save}=S_q
$$

for the non-physics-guided phases, and

$$
S_{save}=\eta_q S_q+\eta_{\phi}S_{phy}
$$

for the physics-guided fine-tuning phase.

## Reviewer-Oriented Logic Summary

FreqMambaGAN first learns a stable forward enhancement mapping from paired degraded/reference data. It then performs unpaired bidirectional domain translation with adversarial, cycle-consistency, identity, perceptual, smoothness, and paired-anchor constraints. The generator bottleneck separates encoded features into low-frequency and high-frequency components, processes them through generic selective state-space branches, and fuses them before decoder reconstruction with skip-attention. In the final fine-tuning phase, physics-inspired regularization constrains the enhanced output without exposing or relying on private implementation details. The best forward generator is selected by a validation score that combines perceptual fidelity, structural preservation, pixel-level error, and, where applicable, physical consistency.

## Reproducibility and Non-Disclosure Note

This file is designed for manuscript review, not for source-level reproduction. It provides enough algorithmic structure for readers to understand the proposed framework while deliberately omitting fixed numerical settings, private function names, device-specific logic, dataset paths, exact branch depth, exact patch granularity, precise mask constants, and proprietary optimization details.
