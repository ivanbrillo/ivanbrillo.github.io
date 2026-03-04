---
layout: page
title: Vision and Voice Assistant on Raspberry
img: assets/img/visionchat_architecture.png
importance: 3
category: AI projects
---

<div class="row mt-3">
  <div class="col-sm-8">
    <p>
      An open-source <strong>embedded AI assistant</strong> running on a Raspberry Pi 3B+, combining
      <strong>real-time object detection, speech interaction, and LLM-powered reasoning</strong>
      to let users query and receive proactive alerts about their surroundings through natural language.
    </p>
  </div>
  <div class="col-sm-4 mt-3 mt-sm-0">
    <a href="https://github.com/andreabochicchio02/VisionChat" target="_blank" class="btn btn-sm z-depth-1 w-100" style="background-color: #24292e; color: white;">
      <i class="fab fa-github"></i> &nbsp; View on GitHub
    </a>
  </div>
</div>

## Overview

VisionChat is a prototype intelligent vision system built on a **Raspberry Pi 3 B+** equipped with a camera module and microphone. The system continuously performs real-time object detection and allows users to interact with it entirely through spoken natural language — asking what objects are visible, where they are located, or requesting proactive notifications when specific objects or motion appear in view.

The project was developed as part of the Industrial Application Project for the Master's degree in Artificial Intelligence and Data Engineering at the **University of Pisa** (Academic Year 2025/2026), by Andrea Bochicchio, Ivan Brillo, and Filippo Gambelli.

<div class="row justify-content-center mt-3">
  <div class="col-sm-10">
    {% include figure.liquid loading="eager" path="assets/img/visionchat_architecture.png" title="VisionChat system architecture — Raspberry Pi 3B+ handling local perception and STT/TTS, offloading LLM reasoning to a cloud-connected host via Ollama REST API" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

## Features

**Real-Time Object Detection.** The system uses SSD MobileNet V3 Large integrated with OpenCV DNN and Picamera2 to perform continuous inference on the Raspberry Pi's CPU. The model is trained on MS COCO (80 object categories) and achieves an average inference time of 384 ms per frame — fast enough to support interactive use on constrained hardware, and significantly outperforming YOLO-based alternatives on the same device.

**Natural Language Interaction.** Users speak queries in either English or Italian; a Vosk small acoustic model transcribes speech in real time with endpoint detection. A structured prompt fusing the transcription with the current scene description (detected objects and positions) is then sent to a cloud-hosted LLM for response generation.

**Proactive Notifications via User Profiling.** Users can ask the system to alert them when a specific object appears or when motion is detected. The LLM extracts the intent and target objects from the spoken request, stores them as notification rules, and the Analyzer module continuously matches them against live detection results to trigger spoken alerts automatically.

**Two-Stage LLM Prompting.** Every user request first goes through a lightweight classification prompt to determine intent — notification setup or general scene query — before a full conversation prompt is constructed if needed. This avoids unnecessary LLM calls for simple notification registrations and keeps latency manageable.

**Streaming TTS Response.** LLM responses are streamed back and simultaneously converted to audio via pico2wave (SVOX Pico), allowing users to hear the answer as it is generated rather than waiting for the full response — partially masking the average 7.3 s LLM request latency.

**Hybrid Edge–Cloud Architecture.** Perception, STT, TTS and alert logic all run locally on the Raspberry Pi, preserving responsiveness and reducing network dependency. Only LLM inference is offloaded to a host PC running Llama 3.2 3B via Ollama, exposed through a RESTful API over the local network.

## Technical Stack

The object detection pipeline uses **SSD MobileNet V3 Large** (5.4M parameters) running on the Raspberry Pi CPU via OpenCV DNN. YOLO-based alternatives were evaluated and rejected due to their significantly higher inference latency (YOLOv5-S: 612 ms, YOLOv7-T: 568 ms vs. SSD-MNv3L: 384 ms) and CPU saturation, which left insufficient headroom for the other modules.

Speech recognition is handled by **Vosk small models** (English: `vosk-model-small-en-us-0.15`, Italian: `vosk-model-small-it-0.22`), selected for their low memory footprint and true streaming capability. Larger Vosk models and OpenAI's Whisper-Tiny were evaluated but ruled out due to excessive size or multi-second decoding latency.

For language understanding, **Llama 3.2 3B** served via **Ollama** was selected after benchmarking against Qwen 2.5 3B and Llama 3.1 8B. The 3B model achieved the lowest generation latency (983 ms/word) while maintaining strong instruction-following performance — with the 8B variant offering no meaningful accuracy gain at significantly higher cost.

## Performance

The vision pipeline achieves an average throughput of **4.44 FPS** on the Raspberry Pi, sufficient for detecting objects in static or slowly changing environments. Frame capture and encoding account for only 55 ms and 26 ms respectively, confirming that the dominant bottleneck is inference (384 ms average). Motion detection adds just 1.4 ms per frame, making it effectively free in the pipeline.

LLM interaction is the system's primary bottleneck, with an average total request time of **7.3 seconds** and an average time to first token of 3.4 seconds. Response streaming substantially mitigates perceived latency, allowing users to read and hear the answer progressively.

## Future Work

Planned improvements include upgrading to a newer Raspberry Pi with a hardware accelerator (e.g., the Hailo-8L AI Kit) to improve vision throughput, and migrating LLM inference to a GPU-equipped machine or a cloud API to reduce response latency. The current work demonstrates that advanced AI capabilities can be democratized on low-cost embedded hardware through careful model selection and a hybrid architecture.