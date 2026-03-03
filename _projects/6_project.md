---
layout: page
title: ResNetUNet vs TransUNet
img: assets/img/transunet.jpg
importance: 1
category: AI projects
---

<div class="row mt-3">
    <div class="col-sm-8">
        <p>
            We implemented and compared two state-of-the-art deep learning architectures for
            multi-class nuclei segmentation on the <strong>PanNuke dataset</strong>: a CNN-based
            <strong>ResNetUNet</strong> and a hybrid CNN-Transformer <strong>TransUNet</strong>.
            Both models are trained under identical conditions to segment <strong>6 distinct cell
            classes</strong> in histology images from 19 tissue types.
        </p>
        <p>
            The evaluation goes beyond simple accuracy, combining per-class <strong>Dice coefficients</strong>,
            paired t-tests, and <strong>False Discovery Rate (FDR) correction</strong> to rigorously
            assess whether observed differences between the two architectures are statistically meaningful.
        </p>
    </div>
    <div class="col-sm-4 mt-3 mt-sm-0">
        <a href="https://github.com/ivanbrillo/TransUNet" target="_blank" class="btn btn-sm z-depth-1 w-100" style="background-color: #24292e; color: white;">
            <i class="fab fa-github"></i> &nbsp; View on GitHub
        </a>
    </div>
</div>

---

## ResNetUNet Architecture

The **ResNetUNet** is a fully convolutional architecture combining a ResNet-inspired encoder with a progressive U-Net decoder. Skip connections fuse multi-scale features across encoder and decoder stages, enabling precise localisation of fine-grained nuclei boundaries. The model contains approximately **25M trainable parameters** and uses batch normalisation and ReLU activations throughout.

<div class="row justify-content-center mt-3">
    <div class="col-sm-10">
        {% include figure.liquid loading="eager" path="assets/img/RESNET.png" title="ResNetUNet architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    ResNetUNet architecture: a ResNet-inspired encoder with skip connections feeding into a progressive upsampling decoder, following the U-Net paradigm.
</div>

---

## TransUNet Architecture

The **TransUNet** is a hybrid CNN-Transformer model that first tokenises the image into 16×16 patches via a convolutional stem, then processes them through **7 Transformer layers with 4 attention heads** to capture global context. A CNN decoder with progressive upsampling reconstructs the segmentation mask. The model also contains approximately **25M trainable parameters**, enabling a fair architectural comparison.

<div class="row justify-content-center mt-3">
    <div class="col-sm-10">
        {% include figure.liquid loading="eager" path="assets/img/transunet.png" title="TransUNet hybrid CNN-Transformer architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    TransUNet architecture: convolutional patch embedding followed by a Transformer encoder with multi-head self-attention, decoded by a CNN upsampling path.
</div>

---

## Results

TransUNet achieved a slightly higher mean Dice score (**0.8410**) compared to ResNetUNet (**0.8390**), with a statistically significant overall improvement (paired t-test: p = 0.0126). After FDR correction, class-level analysis reveals that this gain is driven primarily by **Epithelial cell segmentation** (+0.0162), while ResNetUNet retains a small but significant advantage on **Background** segmentation.

<div class="row justify-content-center mt-3">
    <div class="col-sm-10">
        {% include figure.liquid loading="eager" path="assets/img/dice.jpg" title="Dice coefficient scores per class for ResNetUNet and TransUNet" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Dice computation explanation.
</div>