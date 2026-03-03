---
layout: page
title: Explainable Amyloidosis Detection
img: assets/img/explainability.png
importance: 1
category: research
---

<div class="row mt-3">
    <div class="col-sm-8">
        <p>
            In collaboration with the <strong>TeCIP Institute of Scuola Superiore Sant'Anna</strong> and the
            <strong>Cardiology Hospital of Pisa</strong>, we developed an explainable classifier for
            cardiac amyloidosis from 6-lead ECGs. Amyloidosis is a rare but life-threatening condition
            caused by abnormal protein deposits in the heart, and early detection from ECG signals remains
            a challenging open problem with very few existing automated approaches.
        </p>
        <p>
            This work uses a private clinical dataset (Castiglione et al., 2025) and was carried out in
            strict collaboration with a medical team to ensure clinical validity of both the predictions
            and the model's explanations.
        </p>
    </div>
    <div class="col-sm-4 mt-3 mt-sm-0">
        <a href="https://github.com/ivanbrillo/amyloidosis-ecg" target="_blank" class="btn btn-sm z-depth-1 w-100" style="background-color: #24292e; color: white;">
            <i class="fab fa-github"></i> &nbsp; View on GitHub
        </a>
    </div>
</div>

---

## Architecture

The core model is a **mixed ResNet–Spatial Transformer network with skip connections**, designed to capture both local morphological features and global rhythm patterns across the 6-lead ECG. This architecture outperformed the few existing works in the field on our clinical dataset.

To further boost performance, we combined it with a **pure CNN** branch into an ensemble. A final **SVM classifier**, trained on the probability outputs of the two models (rebalanced using ADASYN), performs the definitive classification decision. Key metrics are **F2 score** and **AUC**, chosen to penalise false negatives in this clinical context.

<div class="row justify-content-center mt-3">
    <div class="col-sm-10">
        {% include figure.liquid loading="eager" path="assets/img/network.png" title="Network architecture: mixed ResNet–Spatial Transformer with skip connections" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Network architecture: a mixed ResNet–Spatial Transformer with skip connections, combined with a pure CNN branch into an ensemble. A downstream SVM performs the final classification.
</div>

---

## Explainability

A core contribution of this work is its interpretability component. We applied two complementary techniques to understand which parts of the ECG the model attends to:

- **Grad-CAM** — highlights the most discriminative time-steps in the convolutional feature maps
- **Attention Rollout** — propagates attention weights across the Transformer layers to produce a global attention map over the input signal

Both methods are validated qualitatively with the clinical team, who assessed whether the model's focus regions align with known electrophysiological markers of amyloidosis.

<div class="row justify-content-center mt-3">
    <div class="col-sm-10">
        {% include figure.liquid loading="eager" path="assets/img/explainability.png" title="Explainability: Grad-CAM and Attention Rollout on an amyloidosis ECG" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Explainability output for an amyloidosis case. Grad-CAM and Attention Rollout highlight the ECG regions driving the classification, validated against clinical expertise.
</div>

