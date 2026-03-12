---
layout: post
title: GAIA - Generative AI for Autonomy
date: 2025-03-12
description: Notes on Wayve's GAIA-1 and GAIA-2 world models for autonomous driving
tags: autonomous-driving generative-ai world-models
categories: ml
related_posts: true
featured: true
thumbnail: assets/img/blog/gaia/autonomous-driving.jpg
---

Based on a presentation by Xiangtian Li from Pony.ai (April 2025), this post summarizes **GAIA**—Wayve's Generative AI for Autonomy—covering both GAIA-1 and GAIA-2 world models for autonomous driving. Reference materials: [GAIA-1 introduction](https://wayve.ai/thinking/introducing-gaia1/) and [GAIA-2](https://wayve.ai/thinking/gaia-2/).

{% include figure.liquid path="assets/img/blog/gaia/autonomous-driving.jpg" caption="Autonomous driving relies on world models like GAIA to predict and simulate driving scenarios." %}

## GAIA-1 Overview

GAIA-1 is a world model that encodes video, text, and action into a unified representation for predicting and generating driving scenarios.

### Sequence Length and Vocabulary Size

As in large language models, two key concepts apply to images:

- **Sequence length:** The number of discrete tokens needed to describe the data.
- **Vocabulary size:** The number of possible values a single token can take.

**Reducing sequence length:**

- **Downsampling:** Each image is downsampled by a factor of $D=16$ in both height and width.
- **Tokenization:** Images are converted into a fixed set of $n=576$ tokens instead of raw pixels.

**Increasing vocabulary size:**

- **Semantic tokens:** DINO pre-training makes tokens more semantically rich, so each token carries more information.
- **Quantization:** A discrete image autoencoder (2D U-Net) expands the vocabulary and maintains semantic fidelity.

### Image Encoder and Decoder

- **Encoder:** Compresses images into discrete semantic tokens.
- **Decoder:** Reconstructs images from encoded tokens. Encoder and decoder are trained jointly. At inference, only the encoder is needed when features (not pixels) are used.

### Image Tokenizer Loss

The quantization loss follows [VQ-VAE](https://arxiv.org/pdf/1711.00937), with a 2D U-Net architecture that produces quantized feature maps and uses both reconstruction and inductive losses.

### Encoding Video, Text and Action

The system jointly encodes:

- **Video frames** as image tokens.
- **Text** via language model tokenization.
- **Actions** as discrete control tokens (e.g., steering, throttle, brake).

### World Model

The world model operates over these unified tokens to predict future states conditioned on past observations, actions, and optional text commands.

{% include figure.liquid path="assets/img/blog/gaia/world-model-ai.jpg" caption="World models learn to predict future states from past observations—a key capability for autonomous systems." %}

### Video Decoder

- **Architecture:** 3D U-Net with spatial and temporal attention.
- **Training:** Conditioned on image tokens from the pre-trained tokenizer; at inference, on predicted tokens from the world model.
- **Multi-task training:** Image and video generation, autoregressive decoding, and video interpolation, with conditioning dropout ($p=0.15$) for robustness.

### GAIA-1 Capabilities

1. **Long driving scenario generation** – Extended, coherent driving videos.
2. **Multiple plausible futures** – Diverse rollout samples.
3. **Controllable generation** – Steering via text or action conditioning.

---

## What's New in GAIA-2

GAIA-2 expands capabilities along several axes:

1. **High-resolution, multi-camera video:** Up to five temporally and spatially consistent camera streams at 448×960 resolution.
2. **Rich structured conditioning:** Control over ego-vehicle kinematics, geography, time of day, weather, road layout (lanes, crossings, traffic lights, intersections), and dynamic agents (location, orientation, dimensions).
3. **External latent embedding conditioning:** Integration with CLIP and proprietary driving model embeddings for semantic control and interoperability.
4. **Multiple generation modes:** Generation from scratch, autoregressive prediction, content inpainting, and scene editing (e.g., change weather).

{% include figure.liquid path="assets/img/blog/gaia/self-driving-sensors.jpg" caption="GAIA-2 supports multi-camera video generation with spatiotemporal consistency across viewpoints." %}

### Model Architecture

- **Video tokenizer encoder:** Compresses high-resolution video into a compact, continuous latent space while preserving semantic and temporal structure.
- **Latent world model:** Predicts future latent states conditioned on past latents, actions, and conditioning inputs.
- **Video tokenizer decoder:** Decodes predicted latent states back into pixel space to generate driving video.

### Video Tokenizer

The encoder–decoder video tokenizer uses a continuous latent space instead of discrete tokens, improving expressiveness and gradient flow. It uses reconstruction and perceptual losses to retain semantic and temporal fidelity.

### Latent World Model

The latent world model uses flow-matching for its time distribution, conditioning on past latents, actions, and a rich set of structured and embedding-based conditionings.

{% include figure.liquid path="assets/img/blog/gaia/road-scenario.jpg" caption="Driving scenarios generated by GAIA span diverse road layouts, weather, and traffic conditions." %}

---

## Advancing from GAIA-1 to GAIA-2

GAIA-2 shifts from discrete tokens to continuous latent space, supports multi-camera high-resolution output, and adds fine-grained conditioning and multiple generation modes. These changes improve controllability, realism, and integration with other autonomy components.

---

## References

- [Introducing GAIA-1](https://wayve.ai/thinking/introducing-gaia1/) – Wayve
- [GAIA-2](https://wayve.ai/thinking/gaia-2/) – Wayve
- [Neural Discrete Representation Learning (VQ-VAE)](https://arxiv.org/pdf/1711.00937)
