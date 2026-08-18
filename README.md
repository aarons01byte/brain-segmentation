# MedSAM Baseline for Brain Tumor Segmentation (BraTS 2025)

A prompt-based MedSAM baseline for glioma segmentation on the BraTS 2025 dataset, built as the foundation-model comparison point in a three-model study of deep-learning approaches to brain tumor segmentation.

Stanford course project.

## Scope of this repository

This repository contains the MedSAM baseline implementation, which is my contribution to the project. The Attention U-Net and nnU-Net models it is compared against were built by my teammates and are described in the paper; they are not included here. The full three-model study and results are in the paper.

## Collaboration

Project by Aaron Sequeira, Hayden Kwan, and Avery Voss. Aaron implemented the MedSAM baseline and led the research framing, writeup, and slides. Hayden built the Attention U-Net, including the attention gates and preprocessing pipeline. Avery built the nnU-Net model, training, and evaluation.

## Motivation

Accurate brain tumor segmentation supports diagnosis, treatment planning, and monitoring, but manual annotation is slow and varies between observers, with specialist agreement dropping to 77 percent for medulloblastoma and 58 percent for glioma cases. The project evaluates three leading segmentation approaches under a single unified protocol, comparing not just accuracy but the trade-offs that matter for clinical use. MedSAM serves as the annotation-light foundation-model baseline against two trained, task-specific networks.

## MedSAM approach

MedSAM is the medical extension of the Segment Anything Model, a pre-trained foundation model that segments from spatial prompts rather than learning from task data. This baseline uses it in pure inference mode, with no training, which is exactly the setting where it is most useful: fast segmentation when annotations are sparse.

The pipeline for each case:

1. Extract the central axial slice of the T1c modality.
2. Compute the minimal bounding box enclosing the ground-truth tumor and use it as the MedSAM prompt.
3. Normalize the slice, resize to 1024x1024, and convert to 3-channel RGB to match MedSAM's expected input.
4. Threshold the predicted mask and resize it back to evaluate against the ground-truth slice.

Evaluation uses Dice, F1, sensitivity, and specificity, the same metrics applied across all three models.

## Results

MedSAM baseline on the validation set:

| Metric | MedSAM |
| --- | --- |
| Dice | 0.7008 |
| F1 | 0.7008 |
| Sensitivity | 0.7664 |
| Specificity | 0.9523 |

For context, the full study compared all three models. nnU-Net performed best overall; MedSAM held its own on specificity despite doing no training:

| Metric | MedSAM | Attention U-Net | nnU-Net |
| --- | --- | --- | --- |
| Dice | 0.7008 | 0.781 | 0.869 |
| F1 | 0.7008 | 0.618 | 0.842 |
| Specificity | 0.9523 | 0.936 | 0.99 |
| Sensitivity | 0.7664 | 0.620 | 0.793 |

## What the baseline shows

- **MedSAM is a strong annotation-light baseline.** With no training and only a bounding-box prompt, it reaches high specificity (0.9523) and captures the central tumor core well, which makes it a viable fast, low-cost option where a fully supervised model is impractical.
- **Its limits are structural.** Using a single central axial slice, it misses information in adjacent slices and other modalities, and it cannot adapt to the domain since it has no learned loss. It under-segments tumors with irregular or low-contrast edges, and its Dice and F1 vary more across cases than the trained models.
- **The comparison is the point.** The trained networks outperform MedSAM on overlap metrics, but the baseline usefully bounds what a general foundation model achieves out of the box on brain-specific segmentation.

## Data

The BraTS 2025 dataset is not included in this repository and must not be committed. It is distributed through the 2025 BraTS Lighthouse Challenge under a data usage agreement that restricts redistribution, so the MRI volumes, segmentation masks, and any slices or bounding boxes derived from them stay out of version control.

Access the data through the challenge: https://www.synapse.org/brats

The dataset provides multi-parametric MRI (native T1, T1Gd, T2, FLAIR) in NIfTI format, skull-stripped and co-registered, with expert annotations of four tumor sub-regions across 1,666 patients. The study used a patient-level 70/15/15 split.

## Running it

```bash
pip install torch numpy nibabel opencv-python
```

The MedSAM checkpoint is not included; download it from the official MedSAM release and point the script at it. The script expects the BraTS NIfTI volumes locally, extracts the central T1c slice per case, generates the bounding-box prompt from the ground-truth mask, runs MedSAM inference, and reports Dice, F1, sensitivity, and specificity.

## Built with

PyTorch, MedSAM (Segment Anything Model, medical extension), NiBabel, NumPy, OpenCV.

## Citation

Sequeira, A., Kwan, H., and Voss, A. *Evaluating the Effectiveness of Deep-Learning Approaches for Brain Tumor Segmentation.* Stanford, 2025.
