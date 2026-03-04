---
layout: page
title: NoProp Diffusion-Based Learning
img: assets/img/noprop_architecture.png
importance: 2
category: AI projects
---

<div class="row mt-3">
    <div class="col-sm-8">
        <p>
            An experimental <strong>PyTorch implementation</strong> of the <strong>NoProp</strong> algorithm,
            exploring its application in two distinct domains: image classification and time-series forecasting.
            NoProp trains neural networks <em>without any form of end-to-end gradient propagation</em>,
            replacing backpropagation with a diffusion-based local learning rule.
        </p>
        <p>
            The approach is grounded in denoising diffusion theory: each layer independently learns to
            reconstruct a clean label embedding from a noisy state, conditioned on the input data —
            enabling fully <strong>modular, parallel training</strong> of deep networks.
        </p>
    </div>
    <div class="col-sm-4 mt-3 mt-sm-0">
        <a href="https://github.com/ivanbrillo/NoProp" target="_blank" class="btn btn-sm z-depth-1 w-100" style="background-color: #24292e; color: white;">
            <i class="fab fa-github"></i> &nbsp; View on GitHub
        </a>
        <a href="https://ivanbrillo.github.io/NoProp/" target="_blank" class="btn btn-sm z-depth-1 w-100 mt-2" style="background-color: #1a1a2e; color: white;">
            📊 &nbsp; Theoretical Foundations Slides
        </a>
    </div>
</div>

> **Reference Paper:** NoProp: Training Neural Networks without Back-propagation or Forward-propagation
> *Qinyu Li, Yee Whye Teh, Razvan Pascanu (2025)* — [arXiv:2503.24322](https://arxiv.org/abs/2503.24322)

---

## Project Implementations

This repository is divided into two main experimental applications of the NoProp architecture:

**NoProp for Classification (MNIST).** A direct implementation of the paper's core concept using the MNIST dataset. The goal is to classify digits using a forward diffusion process on label embeddings. The architecture consists of a sequential **BlockNN** model where gradients are detached between steps, demonstrating how the model learns to reconstruct labels from noisy states without end-to-end backpropagation.

**NoProp for Time-Series Prediction (Autoencoder).** An extension of the NoProp algorithm applied to sequential data. The model forecasts time-series data by embedding the NoProp architecture within an **Autoencoder (AE)** framework, using diffusion-based training principles to learn latent representations of time-series windows while avoiding standard propagation through the entire temporal sequence.

---

## Model Architecture

The classification model consists of a sequence of **BlockNN** modules (T steps). Each block consists of two main components that merge information: a **CNN Image Encoder** (2× Conv2d layers, ReLU activations, MaxPool2d, producing a 32-dim feature vector), a **Label Embedding MLP** processing the noisy label state z<sub>t−1</sub> from the diffusion process, and a **Fusion MLP** that concatenates both streams and outputs the denoised label estimate ẑ<sub>0</sub>.

<div class="row justify-content-center mt-3">
    <div class="col-sm-10">
        {% include figure.liquid loading="eager" path="assets/img/noprop_architecture.png" title="NoProp BlockNN architecture — each block independently denoises label embeddings conditioned on image features" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

The key mechanic underlying the whole approach is **gradient detachment**: during training, the input to the current block z<sub>t−1</sub> is detached from the computation graph (`z[t-1].detach()`). This ensures that the optimizer updates parameters for the current block without backpropagating errors to previous blocks, making each module entirely self-contained.

---

## Training & Results (MNIST)

The model is trained on the **MNIST** dataset (60,000 train samples, 10,000 test samples) with T = 10 diffusion steps, the Adam optimizer (LR: 0.001), and a weighted MSE loss based on the Signal-to-Noise Ratio (SNR). The model achieves high accuracy rapidly:

| Epoch | Avg Loss | Train Accuracy | Test Accuracy |
| :---: | :------: | :------------: | :-----------: |
| 1 | 0.5416 | 94.31% | 94.62% |
| 2 | 0.1529 | 95.98% | 96.11% |
| 3 | 0.1094 | 97.38% | 97.52% |
| 4 | 0.0878 | 97.79% | 97.98% |
| **5** | **0.0717** | **98.15%** | **98.15%** |

---

## Usage

Clone the repository, then open `NoProp.ipynb` in Jupyter Notebook or Google Colab and run all cells to download the MNIST dataset and begin training. Prerequisites: Python 3.x, PyTorch, Torchvision, Matplotlib, NumPy, and Torchsummary.

```python
# Run inference on a single batch after training:
x_test, y_test = next(iter(test_loader))
prediction = classify_batch(x_test, blockNN)
```

**File Structure:**
- `NoProp.ipynb` — Main notebook containing the MNIST classification implementation.
- `NoProp_TimeSeries.ipynb` — Implementation of the Autoencoder for time-series forecasting.
- `data/` — Directory for datasets (auto-downloaded on first run).
- `slides/` — LaTeX slides explaining the math and diffusion process.