# Exploring Methods for PCB Defect Detection and Classification

A comparative study of four computer vision and deep learning approaches for automated defect detection and classification in PCB (Printed Circuit Board) component images.

---

## Project Overview

This project explores and compares four distinct methods for detecting and classifying defects in zoomed-in PCB component images. The dataset consists of **207 defective PCB images** (105 CVAT-annotated, 102 unannotated) and **1000+ good PCB images**. All images are cropped to specific component regions rather than full-board photographs.

The four methods explored are:

| # | Method | Output | Labels Required |
|---|--------|--------|-----------------|
| 1 | Image Subtraction (Classical CV) | Binary + heatmap | None (image pairs only) |
| 2 | Anomaly Detection (WideResNet-50 + FAISS) | Binary + anomaly score | None |
| 3 | Binary Classification (EfficientNet-B0) | Binary + probability | Binary labels |
| 4 | Multi-class Object Detection (YOLOv8s) | Bounding box + class label | CVAT bounding boxes |

---

## Defect Categories

The dataset covers 15 defect categories:
`component_damage` · `component_crack` · `component_liftup` · `component_missing` · `component_shift` · `component_tombstone` · `component_no_solder` · `wrong_polarity` · `wrong_component` · `component_solder_short` · `solder_spread` · `dry_solder` · `pad_damage` · `led_damage` · `solder_ball` 

---



## Methods Summary

### 1. Image Subtraction
Classical computer vision approach applied to **solder spread** and **component liftup**. Uses template matching for alignment, per-channel histogram matching to normalise lighting, and SSIM + morphological contour filtering to detect defect regions. No training required.

### 2. Anomaly Detection
PatchCore-style approach using a **WideResNet-50** backbone (pretrained on ImageNet) to extract 2048-dimensional feature vectors from good PCB patches. Features are stored in a **FAISS** memory bank and anomaly scores are computed as the nearest-neighbour Euclidean distance at inference time.

- Memory bank: 1061 good patches · shape (1061, 2048) · 8.7 MB
- AUC-ROC: **0.8620** · Defect Recall: **0.84** · Best F1: **0.5811**

### 3. Binary Classification
**EfficientNet-B0** fine-tuned using a two-phase training strategy (frozen backbone warmup → full fine-tuning). Handles severe class imbalance (5:1 ratio) via WeightedRandomSampler, class-weighted CrossEntropyLoss, and F1-based model saving.

- AUC-ROC: **0.9790** · Defect Recall: **0.96** · Best Val Defect F1: **0.5725**

### 4. YOLOv8s Multi-class Detection
End-to-end object detection using **YOLOv8s** for simultaneous localisation and 15-class classification. Includes a two-stage pipeline (YOLO detect → EfficientNet-B0 crop classifier) and Grad-CAM explainability. Augmented training set: 70 → 420 images (×5 Albumentations pipeline).

- mAP@0.5: **0.4970** · mAP@0.5:0.95: **0.2480** · Inference: **45.3 ms/image**

---

## Results at a Glance

| Method | Primary Metric | Value | Defect Recall | Localisation |
|--------|---------------|-------|---------------|--------------|
| Image Subtraction | Qualitative | — | — | Heatmap |
| Anomaly Detection | AUC-ROC | 0.8620 | 0.84 | ✗ |
| Binary Classification | AUC-ROC | 0.9790 | 0.96 | ✗ |
| YOLOv8s | mAP@0.5 | 0.4970 | 0.50 | Bounding box |

---


---

## Dataset

The dataset is **not included** in this repository. It consists of:
- **1000+** good PCB component-region images
- **105** defective images with CVAT bounding-box annotations (COCO JSON format)
- **102** defective images without annotations

All images are zoomed-in crops of specific PCB component areas, not full-board photographs.

---



