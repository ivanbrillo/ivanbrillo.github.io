---
layout: page
title: Tumorous Tissue Classification
img: assets/img/biorobotics_results.png
importance: 2
category: research
---

<div class="row mt-3">
    <div class="col-sm-8">
        <p>
            In collaboration with the <strong>BioRobotics Institute of Scuola Superiore Sant'Anna</strong>,
            we developed a machine learning pipeline for classifying tissue-mimicking materials using a
            custom robotic probe equipped with force and position sensors (Massari et al., 2019;
            Bernardo et al., 2025). The long-term goal is to support intraoperative biopsy decisions
            by distinguishing tumorous from healthy tissue in real time.
        </p>
        <p>
            The current work uses a custom dataset of five synthetic materials embedded in a silicone
            vessel to mimic organic tissue properties, providing a controlled testbed before transitioning
            to real biological samples.
        </p>
    </div>
    <div class="col-sm-4 mt-3 mt-sm-0">
        <a href="https://github.com/ivanbrillo/biorobotics-tissue" target="_blank" class="btn btn-sm z-depth-1 w-100" style="background-color: #24292e; color: white;">
            <i class="fab fa-github"></i> &nbsp; View on GitHub
        </a>
    </div>
</div>

---

## Experimental Setup

The robotic probe integrates force and position sensors to record interaction signals as it scans across the surface of the silicone vessel. Five different materials with distinct mechanical properties are arranged inside, each representing a class to be identified.

Signal preprocessing was critical: raw sensor outputs were highly noisy due to environmental variability (temperature, humidity, probe contact angle). We applied filtering to remove border effects at material transitions, and guided feature engineering by domain literature to extract physically meaningful descriptors characterising each material's mechanical response.

<div class="row justify-content-center mt-3">
    <div class="col-sm-10">
        {% include figure.liquid loading="eager" path="assets/img/biorobotics_probe.png" title="Robotic probe with integrated force and position sensors" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The robotic biopsy probe developed at the BioRobotics Institute, equipped with force and position sensors to record tissue interaction signals during scanning.
</div>

---

## Classification Pipeline

We evaluated classical machine learning techniques across the engineered feature space. To reduce misclassification at material boundaries, we aggregated predictions over multiple consecutive probe measurements before committing to a label. An **ensemble model** combining several classifiers provided the best prototype, achieving strong results on the held-out test set.

<div class="row justify-content-center mt-3">
    <div class="col-sm-10">
        {% include figure.liquid loading="eager" path="assets/img/biorobotics_results.png" title="Classification results on the test dataset" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Classification output on the test dataset. The ensemble model reliably distinguishes between the five tissue-mimicking materials across varying experimental conditions.
</div>

