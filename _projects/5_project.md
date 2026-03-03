---
layout: page
title: Domain-Adversarial Transformer for R-Peak Detection from EEG
img: assets/img/thesis_cover.png
importance: 1
category: research
---

<div class="row mt-3">
    <div class="col-sm-8">
        <p>
            My MSc thesis, carried out at the <strong>University of Pisa</strong> (Department of Information Engineering)
            under the supervision of <strong>Prof. Mario G.C.A. Cimino</strong> and <strong>Guido Gagliardi</strong>,
            addresses the problem of detecting R-peaks — the dominant cardiac event in the ECG — directly
            from multi-channel EEG recordings, <em>without any simultaneous ECG reference</em>.
        </p>
        <p>
            While traditionally treated as an artifact removal problem, we reframe cardiac event localization
            as a regression task over the <strong>Distance Transform</strong> of R-peak positions,
            enabling dense, continuous supervision and robust cross-subject generalization.
        </p>
    </div>
    <div class="col-sm-4 mt-3 mt-sm-0">
        <a href="https://github.com/ivanbrillo/EEG-Rpeaks" target="_blank" class="btn btn-sm z-depth-1 w-100" style="background-color: #24292e; color: white;">
            <i class="fab fa-github"></i> &nbsp; View on GitHub (FinalModels branch)
        </a>
    </div>
</div>

---

## Problem Formulation

EEG signals are routinely contaminated by cardiac artifacts originating from the high-energy P-QRS-T waveforms of the heart. Rather than removing these artifacts to clean the EEG, we exploit them: we aim to **precisely localize R-peaks in time directly from EEG**, even when cardiac interference is not visually apparent. This is motivated by research into the functional coupling between the central and autonomic nervous systems (brain-heart interplay), where accurate R-peak timing is a prerequisite for downstream analysis.

The key challenge is **cross-subject generalization**: cardiac artifact morphology varies substantially across individuals due to differences in electrode impedance, head anatomy, and heart position. A model trained on one subject's EEG may fail entirely on another's.

---

## Dataset and Preprocessing

The study uses simultaneous 128-channel EEG and single-lead ECG recordings from 54 subjects (University of Padova). EEG signals are band-pass filtered in the [5, 22.5] Hz range — the band that maximises SNR for QRS complex detection — segmented into 10-second non-overlapping windows, and Z-scaled per channel. Ground-truth R-peak positions are extracted from the ECG using Pan-Tompkins and validated by experts. Cross-subject generalization is assessed using **Leave-One-Out Cross-Validation (LOOCV)**.

---

## Architecture Benchmarking

We systematically benchmarked six architectures from related domains, none of which were originally designed for this exact task:

- **RPNet** — a U-Net with multi-scale Inception-Residual blocks, optimized with Wing Loss for precise peak alignment
- **SeizureTransformer** — a hybrid CNN–Transformer U-Net combining local convolutions with global self-attention
- **LaBraM** — a large EEG foundation model (5.8M parameters) pretrained on 2,500+ hours of EEG data, fine-tuned with a learnable EEG adapter
- **Seq2Seq Transformer** — an autoregressive encoder-decoder model framing the problem as EEG-to-DT translation
- **EEGDfus** — a conditional diffusion model generating the distance transform conditioned on the raw EEG
- **EEGNet** — a compact depthwise-separable CNN used as a lightweight baseline

RPNet and SeizureTransformer achieved the strongest validation F1-scores (~63–64%), with SeizureTransformer also showing competitive timing accuracy (MAE ≈ 0.14 s).

---

## Generalization and Pretraining Strategies

Building on the two best-performing architectures, we investigated several strategies to improve cross-subject robustness:

**CLIP-style Contrastive Pretraining.** We aligned EEG and ECG representations in a shared latent space using a contrastive objective, pairing a trainable EEG encoder (SeizureTransformer) with a frozen β-VAE ECG encoder. A subject-masking strategy prevents biologically related pairs from being treated as negatives. CLIP pretraining with the ECG2DT encoder improved validation F1 by 3.3% and reduced MAE by 1.6% relative to training from scratch.

**Attention-based Skip Connections.** We replaced standard additive skip connections in the U-Net decoder with Efficient Channel Attention (ECA) modules, yielding the best overall validation performance (F1: 64.40%, MAE: 0.1347 s).

**Multi-scale Feature Extraction.** We evaluated Inception, SE-Inception, and Selective Kernel (SK) modules in the encoder. The SK module — which adaptively selects receptive field sizes based on input content — achieved the best results (F1: 64.85%, MAE: 0.1353 s).

**Domain Generalization.** We investigated domain-adversarial training to explicitly suppress subject-specific features from the learned representations, encouraging the encoder to learn cardiac signatures that generalize across individuals.

---

## Status

This work is the subject of my ongoing MSc thesis (Academic Year 2025/2026). Sections on domain generalization, spatial-temporal transformers, and extended MetaFormer architectures are currently under development and evaluation.