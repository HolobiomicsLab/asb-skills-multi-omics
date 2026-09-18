---
name: mlp-training
description: Use when H&E-derived spot tile features must be converted into a low-dimensional
  spot-level embedding that can be fused with gene expression and spatial position
  views in a multimodal spatial transcriptomics model.
when_to_use_negative:
- When no H&E image or spot-tile data are available — the article notes MCGAE is capable
  of functioning effectively even in the absence of image information, so the image
  branch can be omitted rather than forced.
- When a morphological feature table (e.g., already-PCA-reduced embeddings) already
  exists; re-running ResNet50 and PCA would duplicate work.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: PyTorch
  role: Framework used to define and train the MLP that maps 50-d PCA features into
    the spot-like morphological representation Z^morph
  repo: github.com/yiwen-yang/MCGAE
- name: torchvision ResNet50
  role: Pre-trained backbone (classification head removed) producing the 2048-d per-spot
    feature vectors that feed the PCA+MLP pipeline
  repo: github.com/yiwen-yang/MCGAE
- name: scikit-learn
  role: Used to fit PCA retaining the first 50 principal components of the 2048-d
    ResNet50 features before MLP training
  repo: github.com/yiwen-yang/MCGAE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:65e720fa5289ec0e7c198a769014a0a4d00df9fe09740a0eda73df6758bb6ae1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/yiwen-yang/MCGAE
  repo_url: https://github.com/yiwen-yang/MCGAE
  tools:
  - PyTorch
  - ResNet50
  - scikit-learn
  - torchvision
  - torchvision ResNet50
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mlp-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

In MCGAE, a multilayer perceptron (MLP) is trained to transform 50-dimensional PCA-compressed ResNet50 image features into a spot-like morphological representation (Z^morph) for each spatial transcriptomics spot, enabling morphological information to enter the multimodal contrastive integration.

## When to use

Use when H&E-derived spot tile features must be converted into a low-dimensional spot-level embedding that can be fused with gene expression and spatial position views in a multimodal spatial transcriptomics model.

## When NOT to use

- When no H&E image or spot-tile data are available — the article notes MCGAE is capable of functioning effectively even in the absence of image information, so the image branch can be omitted rather than forced.
- When a morphological feature table (e.g., already-PCA-reduced embeddings) already exists; re-running ResNet50 and PCA would duplicate work.

## Inputs

- H&E-stained Visium slide image with spot coordinate table (e.g., Human Breast Cancer)
- Spot-centered image tiles (spot-tile dataset)
- 2048-dimensional per-spot ResNet50 feature vectors (classification head removed)
- 50-dimensional per-spot PCA-reduced feature matrix

## Outputs

- Spot-like morphological representation Z^morph per spatial spot
- (n_spots x 50) morphological embedding matrix, e.g. morph_embeddings.csv

## How to apply

First crop a tile centered on each Visium spot from the H&E slide (e.g., Human Breast Cancer) and pass tiles through a pre-trained ResNet50 with the classification head removed to obtain 2048-dimensional feature vectors per spot. Apply PCA (scikit-learn) to the 2048-d features and retain the first 50 principal components as latent characteristics. Train an MLP (PyTorch) that maps the 50-d PCA features into the spot-like morphological representation space Z^morph. Save the resulting (n_spots x 50) embedding matrix, e.g. to morph_embeddings.csv, for downstream fusion via MCGAE's self-attention mechanism. The rationale is that a learned nonlinear map compresses morphology into a compact spot-aligned view without requiring the full 2048-d backbone output.

## Related tools

- **PyTorch** (Framework used to define and train the MLP that maps 50-d PCA features into the spot-like morphological representation Z^morph) — github.com/yiwen-yang/MCGAE
- **torchvision ResNet50** (Pre-trained backbone (classification head removed) producing the 2048-d per-spot feature vectors that feed the PCA+MLP pipeline) — github.com/yiwen-yang/MCGAE
- **scikit-learn** (Used to fit PCA retaining the first 50 principal components of the 2048-d ResNet50 features before MLP training) — github.com/yiwen-yang/MCGAE

## Examples

```
from sklearn.decomposition import PCA; Z50 = PCA(n_components=50).fit_transform(resnet50_feats)  # 2048-d ResNet50 features -> 50-d input for MLP
```

## Evaluation signals

- The PCA stage must yield exactly 50 components per spot, matching the article's 'first 50 principal components'.
- The saved embedding matrix must have shape (n_spots x 50), with rows aligned 1:1 to Visium spot barcodes.
- Each spot's input to the MLP must derive from its own cropped tile passed through ResNet50 without the classification head (2048-d output).
- The learned Z^morph must be spot-like — i.e., dimensioned per spatial spot so it can be integrated with gene expression and spatial position views via self-attention.
- The output file (e.g., morph_embeddings.csv) should load cleanly and contain numeric embeddings for every spot in the coordinate table.

## Limitations

- No version is specified for the pre-trained ResNet50 network used for image feature extraction, which may affect reproducibility.
- The pipeline depends on image quality and spot-coordinate accuracy of the Visium slide; tile cropping errors propagate into the embeddings.
- The article notes MCGAE's adjacency matrix computation has O(n^2) complexity, which limits overall scalability of the framework that consumes these embeddings.

## Evidence

- [methods] Spot-level image features are reduced by principal component analysis to the first 50 principal components as latent characteristics, and these are then mapped into a spot-like representation space via an MLP.: [evidence span withheld — anchor status: not_found]
- [methods] Save the resulting (n_spots x 50) embedding matrix to morph_embeddings.csv.: [evidence span withheld — anchor status: not_found]
