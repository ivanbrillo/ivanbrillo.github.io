---
layout: page
title: Plant Calcium Signals and VAE
img: assets/img/paw.png
importance: 3
category: research
---

<div class="row mt-3">
    <div class="col-sm-12">
        <p>
            This work was carried out as my <strong>Bachelor's Thesis</strong> within a research group at
            <strong>CNR (National Research Council of Italy)</strong>, in collaboration with the RFX Consortium
            (Cortese et al., 2021; Antoni, Cortese, and Navazio, 2025). The central question was how to
            non-invasively control plant behaviour — for example triggering an early immune response or
            inducing deeper root growth ahead of a drought — without any genetic modification.
        </p>
    </div>
</div>

---

## Biological Background

Most plant responses to external stimuli are encoded through **calcium (Ca²⁺) signalling cascades**. Replicating a specific Ca²⁺ signal would effectively induce the corresponding plant response, even in the absence of the original triggering event. This opens a non-invasive avenue for plant control.

It is known that **plasma-activated water (PAW)** — water exposed to a non-thermal plasma discharge — induces a characteristic Ca²⁺ transduction pattern in plants upon watering. Crucially, the shape of this signal depends on PAW production parameters such as exposure time and plasma power. The challenge is therefore an inverse problem: given a desired target Ca²⁺ signal, find the PAW parameters that produce it.

---

## Approach

To approximate this complex and noisy mapping, we trained a **beta-VAE** on a dataset of recorded Ca²⁺ signals paired with their corresponding PAW parameters. The regularised variational objective encourages the latent space to disentangle the main factors of variation in the transduced signal, making interpolation and nearest-neighbour search geometrically meaningful.

At inference time, we encode the **target Ca²⁺ signal** into the same latent space and apply **k-NN** to retrieve the closest training examples. The PAW parameters associated with those neighbours are then used as the recommended production settings — the PAW generated under those conditions, when applied to the plant, should induce a signal closely resembling the target.

<div class="row justify-content-center mt-3">
    <div class="col-sm-10">
        {% include figure.liquid loading="eager" path="assets/img/paw_latent.png" title="Beta-VAE latent space and Ca2+ signal reconstruction" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    2D projection of the beta-VAE latent space, coloured by PAW production parameters, alongside example Ca²⁺ signal reconstructions. The structured latent geometry enables k-NN retrieval of optimal PAW settings for a given target signal.
</div>
