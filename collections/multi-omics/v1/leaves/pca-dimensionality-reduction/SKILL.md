---
name: pca-dimensionality-reduction
description: Use when spot-level image (or other) features are high-dimensional vectors
  (e.g., 2048-d CNN outputs per spatial transcriptomics spot) and a low-dimensional
  latent space is needed before feeding an MLP or fusion model.
when_to_use_negative:
- Input features are already low-dimensional (e.g., a 50-d embedding or precomputed
  spot-like representation) — additional PCA is unnecessary
- No image modality is available and MCGAE is run in its expression-only mode, so
  there are no ResNet50 features to reduce
- Features come from gene expression rather than image tiles, where MCGAE instead
  builds multi-view representations from expression and spatial adjacency matrices
tools:
- name: scikit-learn
  role: Provides the PCA implementation used to retain the first 50 principal components
    of the 2048-d ResNet50 spot features
  repo: github.com/yiwen-yang/MCGAE
- name: ResNet50
  role: Pre-trained backbone (classification head removed) producing the 2048-d per-spot
    image features that PCA reduces
  repo: github.com/yiwen-yang/MCGAE
- name: PyTorch / torchvision
  role: Loads the pre-trained ResNet50 and applies the torchvision.transforms augmentation
    pipeline to spot tiles before feature extraction
  repo: github.com/yiwen-yang/MCGAE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:f393826fb2cb927222e5a433e4fa5a927a37a56f6cfe93f2009a3735b221d2dd
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
  - PyTorch / torchvision
  - ResNet50
  - scikit-learn
  - torchvision
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# pca-dimensionality-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reduces high-dimensional per-spot feature vectors (e.g., 2048-d ResNet50 image features extracted from H&E spot tiles in the MCGAE pipeline) to a compact 50-principal-component latent representation before downstream multimodal integration.

## When to use

Use when spot-level image (or other) features are high-dimensional vectors (e.g., 2048-d CNN outputs per spatial transcriptomics spot) and a low-dimensional latent space is needed before feeding an MLP or fusion model.

## When NOT to use

- Input features are already low-dimensional (e.g., a 50-d embedding or precomputed spot-like representation) — additional PCA is unnecessary
- No image modality is available and MCGAE is run in its expression-only mode, so there are no ResNet50 features to reduce
- Features come from gene expression rather than image tiles, where MCGAE instead builds multi-view representations from expression and spatial adjacency matrices

## Inputs

- Spot-tile dataset: H&E image tiles cropped from a Visium slide, one tile centered on each spot
- 2048-dimensional per-spot ResNet50 feature matrix (classification head removed)
- Spot coordinate table mapping tiles to spatial transcriptomics spots

## Outputs

- (n_spots x 50) PCA-reduced latent feature matrix (e.g., morph_embeddings.csv)
- Fitted PCA model exposing retained principal components as latent characteristics

## How to apply

Crop an H&E tile centered on each Visium spot and pass it through a pre-trained ResNet50 with the classification head removed to obtain 2048-dimensional feature vectors per spot. Apply PCA (scikit-learn) to the 2048-d features and retain the first 50 principal components as the spot's latent morphological characteristics. Rationale: PCA compresses redundant CNN features into a compact, information-dense space that an MLP can then map into the spot-like morphological representation Z^morph used by MCGAE's multimodal fusion. Save the resulting (n_spots x 50) matrix (e.g., morph_embeddings.csv) so the reduced features can be joined back to spot coordinates for integration with gene expression and spatial adjacency views.

## Related tools

- **scikit-learn** (Provides the PCA implementation used to retain the first 50 principal components of the 2048-d ResNet50 spot features) — github.com/yiwen-yang/MCGAE
- **ResNet50** (Pre-trained backbone (classification head removed) producing the 2048-d per-spot image features that PCA reduces) — github.com/yiwen-yang/MCGAE
- **PyTorch / torchvision** (Loads the pre-trained ResNet50 and applies the torchvision.transforms augmentation pipeline to spot tiles before feature extraction) — github.com/yiwen-yang/MCGAE

## Examples

```
from sklearn.decomposition import PCA; Z_pca = PCA(n_components=50).fit_transform(resnet50_feats)  # 2048-d -> 50-d per spot
```

## Evaluation signals

- Output matrix shape is exactly (n_spots x 50), matching the spot count of the Visium coordinate table
- Retained components are the first 50 principal components in descending explained-variance order
- Row order/IDs of the reduced matrix align with the spot coordinate table so features can be joined to Z^morph
- Resulting embeddings, after MLP mapping, produce spot-like morphological representations consistent with the MCGAE workflow and saved as morph_embeddings.csv
- PCA is fit on the 2048-d ResNet50 features (not on raw pixels or post-MLP features)

## Limitations

- The article specifies no version for the pre-trained ResNet50 network used for image feature extraction, affecting reproducibility of the 2048-d inputs to PCA
- Choosing to retain exactly 50 components is a fixed design choice; the article does not report a sensitivity analysis over the component count
- PCA is applied per dataset/slide; components are not transferable across samples without refitting

## Evidence

- [methods] Spot-level image features are reduced by principal component analysis to the first 50 principal components as latent characteristics, and these are then mapped into a spot-like representation space via an MLP.: [evidence span withheld — anchor status: not_found]
