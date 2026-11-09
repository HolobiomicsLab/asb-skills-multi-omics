---
name: cnn-feature-extraction
description: Use when spatial transcriptomics spots have paired H&E-stained slide
  imagery and morphological information must be converted into a per-spot feature
  matrix (e.g. morph_embeddings.csv) to enrich spatial domain identification.
when_to_use_negative:
- No paired H&E image is available for the spatial transcriptomics run — the article
  notes MCGAE is designed to function effectively even in the absence of image information.
- A spot-level image feature table already exists — re-running tile segmentation and
  CNN extraction would duplicate computation.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: PyTorch
  role: Deep learning framework used to train the MLP that maps 50-d PCA features
    into Z^morph and to run the pre-trained network
  repo: github.com/yiwen-yang/MCGAE
- name: torchvision
  role: Supplies the pre-trained ResNet50 model (classification head removed) and
    the image augmentation transforms applied to spot tiles
  repo: github.com/yiwen-yang/MCGAE
- name: ResNet50
  role: Pre-trained convolutional backbone that converts each spot tile into a 2048-d
    feature vector
- name: scikit-learn
  role: PCA implementation used to reduce the 2048-d ResNet50 features to the first
    50 principal components
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:9ff0c046da9bfadadade678d84c68d9c8045eaa09f3ffd223315ec96a2d18f82
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/yiwen-yang/MCGAE
  tools:
  - PyTorch
  - ResNet50
  - scikit-learn
  - torchvision
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# cnn-feature-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract spot-level morphological image embeddings from Visium H&E tiles using a pre-trained ResNet50 (classification head removed), reduce the 2048-d features to the first 50 principal components via PCA, and map them through an MLP into the spot-like representation Z^morph used by MCGAE's multimodal integration.

## When to use

Use when spatial transcriptomics spots have paired H&E-stained slide imagery and morphological information must be converted into a per-spot feature matrix (e.g. morph_embeddings.csv) to enrich spatial domain identification.

## When NOT to use

- No paired H&E image is available for the spatial transcriptomics run — the article notes MCGAE is designed to function effectively even in the absence of image information.
- A spot-level image feature table already exists — re-running tile segmentation and CNN extraction would duplicate computation.

## Inputs

- Visium H&E-stained histological image (e.g. Human Breast Cancer slide)
- Spot coordinate table mapping spot barcodes to pixel positions
- Spot-centered image tiles cropped from the slide
- torchvision.transforms augmentation pipeline configuration

## Outputs

- 2048-dimensional ResNet50 feature vector per spot
- 50-dimensional PCA-reduced morphological feature matrix
- Spot-like morphological representation Z^morph from the MLP
- morph_embeddings.csv: (n_spots x 50) embedding matrix

## How to apply

Load the Visium H&E image and spot coordinate table, then crop a tile centered on each spot from the slide (e.g. the Human Breast Cancer slide) to build a spot-tile dataset. Apply the torchvision.transforms augmentation pipeline to each tile and pass it through a pre-trained ResNet50 with the classification head removed, yielding 2048-dimensional feature vectors per spot. Apply PCA to the 2048-d ResNet50 features and retain the first 50 principal components as latent characteristics. Train an MLP that maps the 50-d PCA features into the spot-like morphological representation space Z^morph, and save the resulting (n_spots x 50) embedding matrix to morph_embeddings.csv. The rationale is that H&E morphology supplies a complementary view that most expression-only or graph-based methods (Seurat, STAGATE, SpaGCN) do not efficiently exploit, and that MCGAE folds into its self-attention-based multimodal integration.

## Related tools

- **PyTorch** (Deep learning framework used to train the MLP that maps 50-d PCA features into Z^morph and to run the pre-trained network) — github.com/yiwen-yang/MCGAE
- **torchvision** (Supplies the pre-trained ResNet50 model (classification head removed) and the image augmentation transforms applied to spot tiles) — github.com/yiwen-yang/MCGAE
- **ResNet50** (Pre-trained convolutional backbone that converts each spot tile into a 2048-d feature vector)
- **scikit-learn** (PCA implementation used to reduce the 2048-d ResNet50 features to the first 50 principal components)

## Examples

```
import torchvision; resnet = torchvision.models.resnet50(pretrained=True); feats = resnet(transform(tile))
```

## Evaluation signals

- ResNet50 output per spot (with classification head removed) must be a 2048-dimensional vector.
- PCA output must retain exactly the first 50 principal components per spot.
- morph_embeddings.csv must contain n_spots rows and 50 columns, matching the spot coordinate table's row count.
- One tile is cropped per spot, centered on that spot's coordinates, so the tile count must equal the spot count.
- Final MLP output must be a spot-like representation (one Z^morph vector per spot) suitable for downstream multimodal fusion.

## Limitations

- The article does not specify which pre-trained ResNet50 version is used for image feature extraction, complicating reproducibility.
- The embedding extraction is compute-intensive; the authors tested on an Intel Core i9-12900K with 64 GB RAM and an NVIDIA GeForce RTX 3060 GPU, so modest hardware may be a bottleneck.
- PCA truncation to 50 components may discard morphological detail relevant to complex tumor datasets.

## Evidence
