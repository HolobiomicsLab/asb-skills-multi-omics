---
name: histological-image-tile-segmentation
description: Use when a spatial transcriptomics workflow has a paired H&E image and
  spot coordinate table and needs per-spot morphological image embeddings to fuse
  with expression and spatial-position views.
when_to_use_negative:
- No H&E image or spot coordinate table is available for the dataset — MCGAE 'is capable
  of functioning effectively even in the absence of image information', so the expression/spatial
  views alone suffice
- Input is already a per-spot image feature matrix (e.g., morph_embeddings.csv), not
  raw image tiles — skip straight to the PCA/MLP or fusion stage
- The task is expression-only clustering where spatial image information is not needed
  for the cellular landscape description
tools:
- name: ResNet50
  role: Pre-trained convolutional backbone (classification head removed) that encodes
    each H&E spot tile into a 2048-d feature vector
- name: torchvision
  role: Provides the transforms augmentation pipeline applied to spot tiles before
    ResNet50 encoding
- name: PyTorch
  role: Deep-learning framework for running ResNet50 inference and training the MLP
    that maps PCA features into Z^morph
- name: scikit-learn
  role: Performs PCA on the 2048-d ResNet50 features, retaining the first 50 principal
    components
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:a421ccbb4bf07ad85bab409175f0d1a807bec2022562657b4e57e43e8de59fc4
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# histological-image-tile-segmentation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts spot-level morphological features from Visium H&E-stained slides by cropping a tile centered on each spot, encoding tiles with a pre-trained ResNet50, and reducing the 2048-d features via PCA (first 50 PCs) into a spot-like morphological representation (Z^morph) used in MCGAE's multimodal integration.

## When to use

Use when a spatial transcriptomics workflow has a paired H&E image and spot coordinate table and needs per-spot morphological image embeddings to fuse with expression and spatial-position views.

## When NOT to use

- No H&E image or spot coordinate table is available for the dataset — MCGAE 'is capable of functioning effectively even in the absence of image information', so the expression/spatial views alone suffice
- Input is already a per-spot image feature matrix (e.g., morph_embeddings.csv), not raw image tiles — skip straight to the PCA/MLP or fusion stage
- The task is expression-only clustering where spatial image information is not needed for the cellular landscape description

## Inputs

- Visium H&E-stained histology image (e.g., Human Breast Cancer slide)
- Spot coordinate table mapping spot IDs to image coordinates
- Pre-trained ResNet50 weights (classification head removed)

## Outputs

- Spot-tile dataset (one image tile per spatial transcriptomics spot)
- 2048-dimensional ResNet50 feature vector per spot
- 50-dimensional PCA-reduced morphological feature matrix per spot
- Spot-like morphological representation Z^morph saved as morph_embeddings.csv with shape (n_spots x 50)

## How to apply

Load the Visium H&E image and the spot coordinate table, then crop a tile centered on each spot (e.g., from the Human Breast Cancer slide) to build a spot-tile dataset. Apply the torchvision.transforms augmentation pipeline to the spot tiles and pass each tile through a pre-trained ResNet50 with the classification head removed to obtain 2048-dimensional feature vectors per spot. Apply PCA (scikit-learn) to the 2048-d features and retain the first 50 principal components as latent morphological characteristics. Train an MLP that maps the 50-d PCA features into the spot-like morphological representation Z^morph, and save the resulting (n_spots x 50) matrix to morph_embeddings.csv. The rationale is that MCGAE enriches its multi-view contrastive graph autoencoder with morphological image features, integrated with gene expression and spatial positions via a self-attention mechanism.

## Related tools

- **ResNet50** (Pre-trained convolutional backbone (classification head removed) that encodes each H&E spot tile into a 2048-d feature vector)
- **torchvision** (Provides the transforms augmentation pipeline applied to spot tiles before ResNet50 encoding)
- **PyTorch** (Deep-learning framework for running ResNet50 inference and training the MLP that maps PCA features into Z^morph)
- **scikit-learn** (Performs PCA on the 2048-d ResNet50 features, retaining the first 50 principal components)

## Examples

```
import torchvision; resnet = torchvision.models.resnet50(pretrained=True); resnet.fc = torch.nn.Identity(); feats = resnet(transform(tile_batch))  # then sklearn PCA(n_components=50)
```

## Evaluation signals

- Each spatial spot yields exactly one 2048-dimensional ResNet50 feature vector (classification head removed, so no class logits are emitted)
- PCA retains exactly the first 50 principal components — the intermediate feature matrix must be (n_spots x 50)
- The final morph_embeddings.csv matches the spot coordinate table row-for-row with shape (n_spots x 50)
- Spot tiles are centered on the coordinates in the spot table, so tile centroids align with spot positions in the H&E image
- The 50-d morphological features can be fused with expression and spatial-position views in MCGAE's self-attention-based multimodal integration without dimension mismatches

## Limitations

- No version of the pre-trained ResNet50 network is specified, which may hinder reproducibility of the image feature extraction
- Downstream, the adjacency matrix computation for spatial position information has O(n^2) complexity, so the overall pipeline may exhaust computational resources on very large samples
- The article does not specify tile size or augmentation parameters beyond the torchvision.transforms pipeline, requiring reference to the MCGAE repository code

## Evidence

- [methods] Spot-level image features are reduced by principal component analysis to the first 50 principal components as latent characteristics, and these are then mapped into a spot-like representation space via an MLP.: [evidence span withheld — anchor status: not_found]
- [methods] crop a tile centered on each spot from the Human Breast Cancer slide to build the spot-tile dataset.: [evidence span withheld — anchor status: not_found]
