---
layout: page
title: Sign Language Learning Through AI
img: assets/img/signlearnai_quiz.png
importance: 5
category: AI projects
---

<div class="row mt-3">
  <div class="col-sm-8">
    <p>
      An open-source <strong>AI-powered ASL learning application</strong> that provides
      <strong>real-time gesture recognition and adaptive feedback</strong> to help deaf, hard-of-hearing,
      and hearing users learn American Sign Language finger-spelling through interactive quizzes.
    </p>
  </div>
  <div class="col-sm-4 mt-3 mt-sm-0">
    <a href="https://github.com/ivanbrillo/SignLearnAI" target="_blank" class="btn btn-sm z-depth-1 w-100" style="background-color: #24292e; color: white;">
      <i class="fab fa-github"></i> &nbsp; View on GitHub
    </a>
  </div>
</div>

## Overview

SignLearnAI is an interactive desktop application built to make American Sign Language accessible and engaging for everyone. Using a standard webcam and a machine learning pipeline based on **MediaPipe** hand landmark detection, the app provides instant feedback on ASL finger-spelling gestures — no special hardware required.

The project was developed as part of the Business and Project Management Course at the **University of Pisa**, Department of Information Engineering (Academic Year 2024/2025).

<div class="row justify-content-center mt-3">
  <div class="col-sm-10">
    {% include figure.liquid loading="eager" path="assets/img/signlearnai_quiz.png" title="SignLearnAI Live Quiz — real-time ASL gesture recognition with MediaPipe hand landmarks overlay" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

## Features

**Real-Time ASL Recognition.** A webcam feed is processed frame-by-frame using MediaPipe Hand Landmarker, extracting 21 hand keypoints (63 numerical features) per frame. A trained Random Forest classifier then predicts the performed letter with low latency, enabling smooth and responsive interaction during quizzes.

**Adaptive Learning via Weight Scoring.** The application tracks error patterns across both quiz modes (video and text) and biases letter and mode selection toward areas where the user struggles most. A balancing term prevents over-penalizing under-tested letters, and a randomness factor (ε = 0.3) ensures variety in the learning experience.

**Two Difficulty Modes.** Easy Mode displays a reference image of the target ASL sign alongside the live camera feed, while Hard Mode requires the user to recall the sign from memory alone — making it suitable for both complete beginners and advanced learners.

**Phrase Practice.** Beyond individual letters, users can practice predefined or custom phrases in a dedicated screen, building fluency and rhythm across full finger-spelled sequences.

**Error Analytics.** A statistics screen visualizes per-letter error counts for both video and text quiz modes, giving learners a clear picture of which signs need the most attention over their session.

**Open Source.** Unlike competing platforms such as Pocket Sign, SignWise, or NVIDIA's Sign AI, SignLearnAI is fully open source — making it freely auditable, extendable, and deployable in inclusive classroom settings.

## Model & Training

The classifier was trained on the **ASL Sign Language Alphabet Pictures** dataset (8,442 images, augmented to 57,607 via rotation, brightness adjustment, Gaussian blur, and zoom). MediaPipe was used for feature extraction rather than end-to-end CNN inference, keeping the model compact and fast.

Three architectures were evaluated — a dense neural network, a Random Forest, and a kNN classifier. All achieved comparable accuracy and F1 scores, but the **Random Forest** was selected for its significantly lower inference time per sample, a critical requirement for real-time feedback.

Dataset imbalance was addressed by applying **ADASYN** exclusively to the training split (75%), generating synthetic minority-class samples without contaminating the test set (25%).

## Social Impact

According to the World Health Organization, 466 million people lived with disabling hearing loss in 2019 — a figure projected to reach 900 million by 2050. SignLearnAI is designed to lower the barrier to ASL learning for both deaf students and their hearing peers, supporting smoother integration in inclusive classrooms without requiring expensive courses or specialized equipment.

## Future Work

Planned enhancements include dynamic gesture recognition for letters J and Z (which require motion), support for common ASL words beyond finger-spelling, and a mobile port for iOS and Android to extend accessibility beyond desktop environments.