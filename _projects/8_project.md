---
layout: page
title: Federated Learning Platform
img: assets/img/federated_architecture.png
importance: 3
category: AI projects
---

<div class="row mt-3">
    <div class="col-sm-8">
        <p>
            A distributed platform for training and monitoring <strong>TensorFlow deep learning models</strong>
            within a <strong>Horizontal Federated Learning (HFL)</strong> framework, developed as a group project
            for the <em>Distributed Systems and Middleware Technologies</em> course at the
            <strong>University of Pisa</strong> (MSc in AI and Data Engineering, AY 2024–2025).
        </p>
        <p>
            Rather than centralizing sensitive data, each node trains locally and shares only model
            weights with a central aggregator — enabling privacy-preserving collaborative learning
            across a distributed network of participants such as hospitals or autonomous systems.
        </p>
    </div>
    <div class="col-sm-4 mt-3 mt-sm-0">
        <a href="https://github.com/ivanbrillo/FederatedLearningErlang" target="_blank" class="btn btn-sm z-depth-1 w-100" style="background-color: #24292e; color: white;">
            <i class="fab fa-github"></i> &nbsp; View on GitHub
        </a>
        <p class="mt-2" style="font-size: 0.8rem; color: #888;">
            Team: Ivan Brillo, Daniel Pipitone, Andrea Bochicchio
        </p>
    </div>
</div>

---

## Federated Learning

Federated Learning is a distributed approach to machine learning that enables collaborative model training across multiple nodes without requiring data centralization. Instead of sharing raw data, participants transmit model updates — gradients or weights — to a central server for aggregation.

**Horizontal Federated Learning (HFL)** applies when participants share the same feature space but hold different data samples. Each participant trains a local model and periodically pushes updates to a central server. A compelling use case is a distributed network of hospitals collaboratively training a tumor classifier on sensitive patient data, without ever sharing raw records.

**Weights Aggregation via FedAvg.** The global model weights w<sup>(t+1)</sup> are computed as a weighted average of local weights w<sub>k</sub><sup>(t)</sup>, proportional to each node's dataset size n<sub>k</sub> over the total n. This is the standard Federated Averaging algorithm.

---

## System Architecture

The platform is composed of four tightly integrated layers: an **Erlang Master Node** as the distributed coordinator, a **Python Master** for model management and aggregation, **Erlang/Python Worker Nodes** for local training, and a **Java Spring Boot backend** bridging the administrator UI to the Erlang system via JInterface and WebSocket.

<div class="row justify-content-center mt-3">
    <div class="col-sm-10">
        {% include figure.liquid loading="eager" path="assets/img/federated_architecture.png" title="Full system architecture: Client → Java/Spring Boot → Erlang Master → Worker Nodes (Erlang + Python)" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

**Erlang Master Node.** The master acts as the central task coordinator. An OTP supervisor monitors the gen_server process, restarting it on failure (up to 10 times within 60 seconds, transient restart type). The master initializes a linked Python process for model aggregation, manages node discovery via `.erlang.hosts` whitelisting, and distributes load and train pipelines across connected nodes. Every 10 seconds it attempts to reconnect to all known hosts; `nodeup` and `nodedown` messages drive the reconnect procedure. The master also saves model weights every 3 epochs as a backup.

**Node Discovery and Fault Tolerance.** Two state lists are maintained: `currentUpNodes` for active training participants, and `previousInitializedNodes` for efficient partition recovery. When a crash or disconnection occurs during a blocking wait, the master dynamically adjusts the expected response count rather than timing out. Late responses from nodes that disconnected and reconnected mid-operation are discarded.

**Python Master via Erlport.** The Erlang master spawns a Python process via the Erlport library, which handles bidirectional inter-language communication over a port. The Python process hosts the `NetworkModel` (a `tf.keras.Model` subclass), implements the FedAvg aggregation, and handles model serialization with `pickle`. Supported commands include `get_model`, `get_weights`, `update_weights`, `save_model`, and `load_model`.

**Python Worker Nodes.** Each worker loads a local dataset partition via a `dataParser.py` stub, receives the model definition and current weights from the master, performs a local training epoch, and returns `{weights, train_accuracy, test_accuracy}` back to the Erlang node. Workers also stream real-time node metrics (CPU, RAM, response time) to the master at 1 Hz in JSON format.

---

## Java Back-End

The Spring Boot backend bridges the administrator UI and the Erlang master via JInterface. Two always-running threads implement a Consumer-Producer pattern over `BlockingQueue` objects: `WebSocketForwarder` broadcasts Erlang messages to all active WebSocket sessions, while `ErlangController` consumes UI commands and dispatches them to Erlang.

**Authentication** uses cookie-based session management with two roles: `admin` (can issue Erlang commands) and `user` (read-only spectator). A Spring Security `securityFilterChain` enforces authentication on every HTTP request, and multiple concurrent sessions per account are supported.

**WebSocket via SockJS.** SockJS wraps the WebSocket connection, leveraging the HTTP handshake to integrate Spring Security authentication before the connection is upgraded. All frontend events are processed by a `processingInput` handler that updates the live dashboard.

**Session Restoration.** A `cacheSession` queue stores all relevant UI state messages. When a new client connects mid-session, it receives the full cached state, bringing its UI to parity with long-running clients. A `synchronized` block prevents race conditions during session replay, and train logs are cleared on each new training run to minimize redundant messages.

---

## Requirements

**Functional.** The system supports multi-node HFL training with configurable epochs and accuracy thresholds, FedAvg-based weight aggregation, real-time node monitoring, simultaneous multi-experiment execution, persistent model save/load, custom TensorFlow model injection, and training interruption.

**Non-Functional.** The system minimizes transmitted data (only weights, not raw datasets), is fault-tolerant against node crashes and network partitions, enforces data privacy by keeping datasets local, and supports flexible model swapping without system restart.

---

## Results

The system was validated with a 5-node network, each holding a disjoint equal-sized partition of the MNIST dataset. The federated model was compared against a centralized baseline trained on the full dataset.

| Method | Validation Accuracy |
| :--- | :---: |
| Centralized Training | 0.9878 |
| **HFL (5 nodes, FedAvg)** | **0.9859** |

The HFL approach achieves accuracy within 0.2% of the centralized baseline, confirming that the FedAvg implementation effectively aggregates distributed knowledge without requiring any data sharing.