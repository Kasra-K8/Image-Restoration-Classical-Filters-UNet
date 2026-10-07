# Image Restoration: Classical Filters and Edge-Aware U-Net

This repository contains a series of image-restoration experiments completed as part of a graduate **Digital Image Processing** course during my M.Sc. studies in Biomedical Engineering at K. N. Toosi University of Technology.

The project investigates restoration of images degraded by **linear motion blur and additive Gaussian noise** using both classical image-processing algorithms and deep learning.

Four restoration approaches are explored:

- Wiener filtering
- Richardson–Lucy deconvolution
- Constrained Least Squares (CLS) restoration
- A custom edge-aware U-Net

The project provides a direct comparison between traditional inverse-filtering techniques and a learned convolutional restoration model.

---

## Project Overview

The project is divided into four tasks:

| Task | Method | Objective |
|---|---|---|
| Task 1 | Image Degradation Model | Simulate motion blur and Gaussian noise |
| Task 2 | Wiener & Richardson–Lucy | Classical image restoration |
| Task 3 | Constrained Least Squares | Frequency-domain regularized restoration |
| Task 4 | Custom U-Net | Deep-learning image restoration |

---

# Task 1 — Image Degradation Model

A controlled image-degradation process was created to simulate common imaging artifacts.

The degradation pipeline consists of:

```text
Original Image
      ↓
Grayscale Conversion
      ↓
70° Linear Motion Blur
      ↓
Gaussian Noise
      ↓
Degraded Image
```

## Motion Blur

A custom **31 × 31 Point Spread Function (PSF)** is generated to represent linear motion blur at an angle of:

```text
70 degrees
```

The PSF is normalized and convolved with the original image.

## Gaussian Noise

Additive Gaussian noise is subsequently introduced using:

```text
Variance = 0.05
```

The resulting image therefore contains both structured motion blur and random sensor-like noise.

---

# Task 2 — Classical Image Restoration

Two well-known image-restoration algorithms were evaluated:

- Wiener Filter
- Richardson–Lucy Deconvolution

The goal was to determine the parameter configuration producing the highest Peak Signal-to-Noise Ratio (PSNR).

---

## Wiener Filter

The Wiener filter attempts to balance inverse filtering with suppression of amplified noise.

Several balance parameters were evaluated:

```text
0.001
0.01
0.05
0.1
0.5
1
10
20
```

The best-performing configuration was:

```text
Balance = 20
PSNR = 20.73 dB
```

---

## Richardson–Lucy Deconvolution

Richardson–Lucy is an iterative deconvolution algorithm.

The following iteration counts were evaluated:

```text
5
15
30
50
```

The best configuration was:

```text
Iterations = 5
PSNR = 16.60 dB
```

Increasing the number of iterations did not improve reconstruction quality because the algorithm increasingly amplified the noise present in the degraded image.

---

## Classical Restoration Comparison

| Method | Best Parameter | PSNR |
|---|---:|---:|
| **Wiener** | Balance = 20 | **20.73 dB** |
| Richardson–Lucy | 5 iterations | 16.60 dB |

Under the simulated motion-blur and Gaussian-noise conditions, Wiener filtering produced the strongest classical restoration result.

---

# Task 3 — Constrained Least Squares Restoration

A **Constrained Least Squares (CLS)** image-restoration method was implemented manually in the frequency domain.

The method combines:

- The Fourier transform of the degradation PSF
- A Laplacian regularization operator
- A regularization parameter γ

The restoration estimate is based on:

```text
F̂ = H*G / (|H|² + γ|P|²)
```

where:

- `G` represents the degraded image in the frequency domain
- `H` represents the degradation transfer function
- `P` represents the Laplacian regularization operator
- `γ` controls the strength of regularization

A logarithmic parameter sweep was performed over multiple γ values.

The selected configuration was:

```text
Gamma ≈ 31.62
PSNR = 14.26 dB
```

---

## Classical Method Comparison

| Method | PSNR |
|---|---:|
| **Wiener** | **20.73 dB** |
| Richardson–Lucy | 16.60 dB |
| CLS | 14.26 dB |

For the degradation model used in this experiment, Wiener filtering provided the highest reconstruction PSNR.

---

# Task 4 — Edge-Aware U-Net Restoration

The final task investigates whether a learned convolutional model can restore blurred and noisy images.

A custom U-Net-style architecture was implemented using **PyTorch**.

The model contains:

- Two encoder stages
- A convolutional bottleneck
- Two decoder stages
- Skip connections
- 1×1 convolutional skip-channel reduction
- Batch normalization
- 5×5 convolutional kernels
- Sigmoid output activation

---

## Architecture

A simplified representation of the network is:

```text
Input
  │
  ▼
Conv 5×5 — 16 channels
  │
  ├────────────── Skip Connection
  ▼
Max Pooling
  │
  ▼
Conv 5×5 — 32 channels
  │
  ├────────────── Skip Connection
  ▼
Max Pooling
  │
  ▼
Conv 5×5 — 64 channels
     Bottleneck
  │
  ▼
Transposed Convolution
  │
  ├── Reduced Skip Connection
  ▼
Conv 5×5 — 32 channels
  │
  ▼
Transposed Convolution
  │
  ├── Reduced Skip Connection
  ▼
Conv 5×5 — 16 channels
  │
  ▼
1×1 Output Convolution
  │
  ▼
Restored Image
```

The skip features are compressed using **1×1 convolutions** before concatenation with the decoder features.

---

# Edge-Aware Loss Function

Instead of training the model using only pixel-wise Mean Squared Error, a custom loss function was implemented:

```text
Loss = MSE(image) + λ × MSE(edges)
```

with:

```text
λ = 0.10
```

Image edges are extracted using horizontal and vertical **Sobel filters**.

This encourages the model to preserve edges and structural details in addition to minimizing pixel reconstruction error.

---

# Training Dataset

The training data was generated from four source images:

```text
Barbara
Chelsea
Lighthouse
Monarch
```

Each image was:

1. Converted to grayscale
2. Degraded using the same motion-blur PSF
3. Corrupted with Gaussian noise
4. Divided into 64 × 64 image patches
5. Augmented using flips and rotations

The augmented patches were used to train the U-Net.

Training settings:

```text
Patch size: 64 × 64
Batch size: 32
Optimizer: Adam
Learning rate: 1e-3
Epochs: 10
```

---

# Training Behavior

The edge-aware training loss decreased consistently:

```text
Epoch 1:  0.0374
Epoch 2:  0.0336
Epoch 3:  0.0326
Epoch 4:  0.0320
Epoch 5:  0.0312
Epoch 6:  0.0307
Epoch 7:  0.0305
Epoch 8:  0.0304
Epoch 9:  0.0299
Epoch 10: 0.0297
```

This indicates stable convergence during training.

---

# Generalization Experiment

The trained network was evaluated on a separate image that was **not used for model training**.

The test image was degraded using the same motion-blur and Gaussian-noise process and restored using:

- Wiener filtering
- The custom U-Net

## Results

| Method | PSNR | Approx. Inference Time |
|---|---:|---:|
| **Wiener** | **21.81 dB** | **0.025 s** |
| Custom Edge-Aware U-Net | **21.70 dB** | 0.237 s |

The custom U-Net achieved reconstruction quality within approximately **0.11 dB PSNR** of the Wiener filter on the held-out image.

Wiener filtering remained substantially faster in this experiment.

These results illustrate that a small learned restoration model can approach the reconstruction quality of a strong classical filter, even when trained using a limited educational dataset.

---

# Key Findings

### Wiener filtering performed best among the classical methods

For the initial benchmark image:

```text
Wiener:           20.73 dB
Richardson-Lucy:  16.60 dB
CLS:              14.26 dB
```

### Richardson–Lucy was sensitive to noise

Additional iterations increasingly amplified the Gaussian noise, making a low iteration count preferable.

### Regularization strongly affects CLS restoration

The CLS experiment demonstrated the importance of the regularization parameter when balancing inverse filtering and smoothness constraints.

### The U-Net learned a useful restoration mapping

Despite being trained on a small image set, the custom network produced a PSNR close to the Wiener filter when evaluated on an unseen image.

### Classical methods remain highly competitive

For this controlled degradation model, Wiener filtering achieved slightly better reconstruction quality while requiring substantially less inference time.

The experiment therefore demonstrates an important practical trade-off between analytical restoration methods and learned image-restoration models.

---

# Technologies and Concepts

This project explores:

- Digital Image Processing
- Image Restoration
- Image Deconvolution
- Inverse Problems
- Motion Blur
- Gaussian Noise
- Point Spread Functions
- Fourier Transforms
- Wiener Filtering
- Richardson–Lucy Deconvolution
- Constrained Least Squares
- Laplacian Regularization
- Peak Signal-to-Noise Ratio
- Convolutional Neural Networks
- U-Net
- PyTorch
- Sobel Edge Detection
- Edge-Aware Loss Functions
- Image Augmentation

---

# Repository Structure

```text
.
├── Image_Restoration_Experiments.ipynb
├── README.md
├── requirements.txt
├── .gitignore
└── report/
    └── Image_Restoration_Report_FA.pdf
```

If redistribution of the source images is permitted, they can optionally be organized under an `images/` directory.

---

# Installation

Install the required packages using:

```bash
pip install -r requirements.txt
```

The main dependencies are:

```text
numpy
matplotlib
scipy
scikit-image
torch
```

---

# Running the Notebook

The notebook performs the tasks sequentially because later experiments use variables generated in previous sections.

The general execution order is:

```text
Task 1 — Generate degraded image and PSF
             ↓
Task 2 — Wiener and Richardson–Lucy restoration
             ↓
Task 3 — CLS restoration
             ↓
Task 4 — Train and evaluate custom U-Net
```

Run the notebook from top to bottom after configuring the required source-image paths.

---

# Academic Context

**Course:** Digital Image Processing  
**Project Type:** Graduate Computational Assignment  
**Program:** M.Sc. Biomedical Engineering  
**University:** K. N. Toosi University of Technology  
**Author:** Kasra Attar Kashani

The complete Persian-language project report is available in the `report/` directory.

---

# Disclaimer

This repository was created for educational and academic purposes.

The experiments use a small benchmark image collection and controlled synthetic degradation. The reported results should therefore be interpreted as demonstrations of image-restoration techniques rather than large-scale evaluations of general-purpose restoration systems.
