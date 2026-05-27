## Symbol Definitions

---
| Symbol                             | Meaning                                                             |
| ---------------------------------- | ------------------------------------------------------------------- |
| (\mathcal{A})                      | degraded underwater image domain                                    |
| (\mathcal{B})                      | visually enhanced or reference-quality underwater image domain      |
| (x_A)                              | unpaired degraded underwater image sampled from (\mathcal{A})       |
| (x_B)                              | unpaired reference-style image sampled from (\mathcal{B})           |
| ((x_A^p,x_B^p))                    | paired degraded/reference sample used for supervised anchoring      |
| (\mathcal{P})                      | paired training set                                                 |
| (\mathcal{U}_A,\mathcal{U}_B)      | unpaired training sets from domains (\mathcal{A}) and (\mathcal{B}) |
| (\mathcal{V})                      | validation set for checkpoint selection                             |
| (G_{A\rightarrow B})               | forward enhancement generator                                       |
| (G_{B\rightarrow A})               | backward degradation-domain generator                               |
| (D_A,D_B)                          | domain discriminators for (\mathcal{A}) and (\mathcal{B})           |
| (\Theta_G,\Theta_D)                | trainable generator and discriminator parameters                    |
| (\Gamma_{\mathrm{enc}})            | shallow encoder parameter subset used for staged freezing           |
| (\Gamma_{\mathrm{ssm}})            | selected state-space parameter subset stabilized during fine-tuning |
| (\rho)                             | symbolic frequency partition boundary                               |
| (\mathcal{E}(\cdot))               | encoder operation                                                   |
| (\mathcal{F}_{\rho}(\cdot))        | symbolic frequency decomposition operator                           |
| (\mathcal{M}_{H}(\cdot))           | high-frequency contextual modeling branch                           |
| (\mathcal{M}_{L}(\cdot))           | low-frequency contextual modeling branch                            |
| (\mathcal{C}(\cdot))               | feature fusion operation                                            |
| (\mathcal{R}(\cdot))               | decoder reconstruction operation                                    |
| (\mathcal{S}(\cdot,\cdot))         | skip-attention fusion operation                                     |
| (\Phi(\cdot))                      | frozen perceptual feature extractor                                 |
| (\Psi_{\mathrm{dc}}(\cdot))        | dark-channel consistency prior                                      |
| (\Psi_{\mathrm{phy}}(\cdot,\cdot)) | simplified physics-inspired consistency prior                       |
| (\Lambda)                          | set of symbolic loss weights                                        |
| (\alpha_G,\alpha_D)                | generator and discriminator learning-rate symbols                   |
| (\Omega_D(\tau))                   | symbolic discriminator update schedule                              |
| (\mathcal{Q}(\cdot))               | validation-based model selection criterion                          |
| (\varepsilon)                      | minimum improvement tolerance for checkpoint updating               |
| (\tau)                             | symbolic training step index                                        |

---

## Algorithm A. Confidential High-Level Pseudocode of FreqMambaGAN

```text
Input:
    Paired dataset 𝒫 = {(x_A^p, x_B^p)}
    Unpaired datasets 𝒰_A = {x_A}, 𝒰_B = {x_B}
    Validation set 𝒱
    Frequency partition symbol ρ
    Loss-weight set Λ
    Generator and discriminator learning symbols α_G, α_D
    Training schedules 𝒯_sup, 𝒯_cyc^sta, 𝒯_cyc^ada, 𝒯_phy
    Discriminator update schedule Ω_D(τ)

Output:
    Trained forward enhancement generator G*_{A→B}
    Enhanced image x̂_B = G*_{A→B}(x_A)


Procedure GeneratorForward(x; G, ρ):

    h, s ← 𝔈(x; Θ_G)
        ⟂ Encode the input image and preserve shallow spatial features s.

    h_L, h_H ← 𝔉_ρ(h)
        ⟂ Symbolically separate low-frequency appearance features
          and high-frequency structural features.

    z_H ← 𝓜_H(h_H; Θ_G)
        ⟂ Model high-frequency texture, edge, and structural dependencies.

    z_L ← 𝓜_L(h_L; Θ_G)
        ⟂ Model low-frequency color, illumination, and haze-related context.

    z ← 𝓒(z_H, z_L, h; Θ_G)
        ⟂ Fuse frequency-specific representations with residual stabilization.

    y ← 𝓡(z, 𝓢(s); Θ_G)
        ⟂ Reconstruct the image with skip-attention-guided spatial preservation.

    return y


Procedure DiscriminatorScore(x; D):

    r ← D(x; Θ_D)
        ⟂ Produce a patch-level realism score map without probability normalization.

    return r


Initialize:
    Generators G_{A→B}, G_{B→A}
    Discriminators D_A, D_B
    Best validation score q* ← −∞


Stage 𝒮_sup: Supervised Warm-Up

    Freeze D_A and D_B.

    For each τ ∈ 𝒯_sup do:

        Sample paired data (x_A^p, x_B^p) from 𝒫.

        x̂_B^p ← GeneratorForward(x_A^p; G_{A→B}, ρ)

        𝓛_sup ← ‖x̂_B^p − x_B^p‖_abs

        Update trainable parameters of G_{A→B}
            by descending ∇_{Θ_G} 𝓛_sup with learning symbol α_G.

        q ← 𝒬(G_{A→B}; 𝒱)

        If q > q* + ε then:
            q* ← q
            Store current G_{A→B} as the best supervised checkpoint.


Stage 𝒮_cyc^sta: Stabilized Cycle-Adversarial Learning

    Freeze Γ_enc in G_{A→B}.
    Activate trainable parameters of G_{B→A}, remaining G_{A→B}, D_A, and D_B.

    For each τ ∈ 𝒯_cyc^sta do:

        Sample x_A from 𝒰_A and x_B from 𝒰_B.
        Sample paired anchor (x_A^p, x_B^p) from 𝒫.

        x̂_B ← GeneratorForward(x_A; G_{A→B}, ρ)
        x̃_A ← GeneratorForward(x̂_B; G_{B→A}, ρ)

        x̂_A ← GeneratorForward(x_B; G_{B→A}, ρ)
        x̃_B ← GeneratorForward(x̂_A; G_{A→B}, ρ)

        x̂_B^p ← GeneratorForward(x_A^p; G_{A→B}, ρ)

        𝓛_adv^G ← adversarial generator loss using D_A and D_B
        𝓛_cyc ← ‖x̃_A − x_A‖_abs + ‖x̃_B − x_B‖_abs
        𝓛_id ← identity-preserving loss for both translation directions
        𝓛_TV ← total-variation regularization on x̂_B
        𝓛_perc ← ‖Φ(x̂_B) − Φ(x_A)‖
        𝓛_anc ← ‖x̂_B^p − x_B^p‖_abs

        𝓛_G^cyc ←
              λ_adv 𝓛_adv^G
            + λ_cyc 𝓛_cyc
            + λ_id 𝓛_id
            + λ_TV 𝓛_TV
            + λ_perc 𝓛_perc
            + λ_anc 𝓛_anc

        Update trainable generator parameters
            by descending ∇_{Θ_G} 𝓛_G^cyc with learning symbol α_G.

        If Ω_D(τ) is satisfied then:

            r_B_real ← DiscriminatorScore(x_B; D_B)
            r_B_fake ← DiscriminatorScore(stop_gradient(x̂_B); D_B)

            r_A_real ← DiscriminatorScore(x_A; D_A)
            r_A_fake ← DiscriminatorScore(stop_gradient(x̂_A); D_A)

            𝓛_D ← least-squares discriminator loss over real and generated score maps

            Update D_A and D_B
                by descending ∇_{Θ_D} 𝓛_D with learning symbol α_D.

        q ← 𝒬(G_{A→B}; 𝒱)

        If q > q* + ε then:
            q* ← q
            Store current G_{A→B} as the best cycle-adversarial checkpoint.


Stage 𝒮_cyc^ada: End-to-End Cycle-Adversarial Adaptation

    Release Γ_enc according to the predefined trainability schedule.
    Keep G_{A→B}, G_{B→A}, D_A, and D_B trainable.

    For each τ ∈ 𝒯_cyc^ada do:

        Repeat the bidirectional translation, cycle reconstruction,
        forward anchor regularization, generator update,
        scheduled discriminator update, and validation selection
        defined in Stage 𝒮_cyc^sta.


Stage 𝒮_phy: Physics-Guided Fine-Tuning

    Freeze D_A and D_B as fixed adversarial scoring networks.
    Freeze Γ_ssm to stabilize the learned state-space dynamics.
    Keep the remaining generator parameters trainable.

    For each τ ∈ 𝒯_phy do:

        Sample x_A from 𝒰_A and x_B from 𝒰_B.
        Sample paired anchor (x_A^p, x_B^p) from 𝒫.

        x̂_B ← GeneratorForward(x_A; G_{A→B}, ρ)
        x̃_A ← GeneratorForward(x̂_B; G_{B→A}, ρ)

        x̂_A ← GeneratorForward(x_B; G_{B→A}, ρ)
        x̃_B ← GeneratorForward(x̂_A; G_{A→B}, ρ)

        x̂_B^p ← GeneratorForward(x_A^p; G_{A→B}, ρ)

        𝓛_adv^G ← fixed-score adversarial generator loss using frozen D_A and D_B
        𝓛_cyc ← ‖x̃_A − x_A‖_abs + ‖x̃_B − x_B‖_abs
        𝓛_id ← identity-preserving loss for both translation directions
        𝓛_TV ← total-variation regularization on x̂_B
        𝓛_perc ← ‖Φ(x̂_B) − Φ(x_A)‖
        𝓛_anc ← ‖x̂_B^p − x_B^p‖_abs

        𝓛_dc ← Ψ_dc(x̂_B)
            ⟂ Dark-channel consistency regularization.

        𝓛_phy ← Ψ_phy(x_A, x̂_B)
            ⟂ Simplified image-formation consistency regularization.

        𝓛_G^phy ←
              λ_adv^phy 𝓛_adv^G
            + λ_cyc^phy 𝓛_cyc
            + λ_id^phy 𝓛_id
            + λ_TV^phy 𝓛_TV
            + λ_perc^phy 𝓛_perc
            + λ_anc^phy 𝓛_anc
            + λ_dc 𝓛_dc
            + λ_phy 𝓛_phy

        Update trainable generator parameters
            by descending ∇_{Θ_G} 𝓛_G^phy with learning symbol α_G.

        q ← 𝒬(G_{A→B}; 𝒱)

        If q > q* + ε then:
            q* ← q
            Store current G_{A→B} as the best physics-guided checkpoint.


Inference:

    Load G*_{A→B} selected by 𝒬.

    For each degraded underwater image x_A do:
        x̂_B ← GeneratorForward(x_A; G*_{A→B}, ρ)

    return x̂_B
```

---

## Minimal Explanation for Manuscript

The above pseudocode summarizes the complete execution flow of FreqMambaGAN from input sampling to final enhanced-image generation. The forward generator first extracts hierarchical features, performs symbolic frequency decomposition, processes low-frequency and high-frequency components with separate contextual modeling branches, and reconstructs the enhanced image through fusion, decoding, and skip-attention preservation. The discriminator provides patch-level adversarial supervision while maintaining global contextual awareness.

The training procedure is organized into supervised warm-up, stabilized cycle-adversarial learning, end-to-end cycle-adversarial adaptation, and physics-guided fine-tuning. The supervised warm-up initializes the forward enhancement mapping with paired data. The cycle-adversarial stages then exploit unpaired degraded/reference-style domains through adversarial, cycle-consistency, identity, perceptual, total-variation, and anchor constraints. Finally, the physics-guided fine-tuning stage uses dark-channel and image-formation consistency priors as auxiliary regularization terms, while frozen discriminators provide stable adversarial feedback. This algorithmic summary clarifies the relationship among the generator, discriminator, frequency decomposition, skip-attention reconstruction, loss constraints, and stage-wise optimization strategy without exposing implementation-specific low-level operations.

---

