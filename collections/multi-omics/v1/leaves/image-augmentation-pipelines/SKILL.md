---
name: image-augmentation-pipelines
description: Use when you have Visium H&E slides with a spot coordinate table and
  need per-spot morphological image embeddings (e.g. Z-morph) to integrate with spatial
  gene expression.
when_to_use_negative:
- No H&E image is available for the spatial transcriptomics dataset — MCGAE is explicitly
  stated to function effectively even in the absence of image information, so skip
  this module.
- Input is already a spot-level feature/embedding table; re-cropping tiles and re-extracting
  ResNet50 features would duplicate work.
- Very large sample counts where the article notes O(n^2) adjacency and scalability
  concerns — the article suggests subgraph sampling strategies rather than heavier
  image processing.
edam_operation: http://edamontology.org/operation_3935
tools:
- name: PyTorch
  role: Deep-learning framework used to run ResNet50 and train the MLP mapping PCA
    features to Z^morph
  repo: https://github.com/yiwen-yang/MCGAE
- name: torchvision
  role: Provides the transforms augmentation pipeline applied to spot tiles and the
    pre-trained ResNet50
  repo: https://github.com/yiwen-yang/MCGAE
- name: ResNet50
  role: Pre-trained CNN backbone (classification head removed) producing 2048-d features
    per spot tile
- name: scikit-learn
  role: PCA implementation used to retain the first 50 principal components of the
    2048-d features
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:e172c51635293277c934b4cd2d0422c2b99467e2393c095acde60ab1f6863e5a
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# image-augmentation-pipelines

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds the histological image embedding module of MCGAE: H&E spot tiles are augmented with torchvision.transforms, encoded by a pre-trained ResNet50 (classification head removed), PCA-compressed to 50 components, and mapped by an MLP into the spot-like morphological representation Z^morph.

## When to use

Use when you have Visium H&E slides with a spot coordinate table and need per-spot morphological image embeddings (e.g. Z-morph) to integrate with spatial gene expression.

## When NOT to use

- No H&E image is available for the spatial transcriptomics dataset — MCGAE is explicitly stated to function effectively even in the absence of image information, so skip this module.
- Input is already a spot-level feature/embedding table; re-cropping tiles and re-extracting ResNet50 features would duplicate work.
- Very large sample counts where the article notes O(n^2) adjacency and scalability concerns — the article suggests subgraph sampling strategies rather than heavier image processing.

## Inputs

- Visium H&E-stained full-resolution slide image
- Spot coordinate table (spot centers) from the Visium spaceranger output
- Pre-trained ResNet50 weights (torchvision)

## Outputs

- 2048-d ResNet50 feature vector per spot (intermediate)
- 50-d PCA-reduced spot image features (intermediate)
- (n_spots x 50) morphological embedding matrix Z^morph, e.g. morph_embeddings.csv

## How to apply

Load the Visium H&E image and spot coordinate table and crop a tile centered on each spot (Human Breast Cancer slide in the reference workflow) to build the spot-tile dataset. Apply the torchvision.transforms augmentation pipeline to each spot tile and pass it through a pre-trained ResNet50 with the classification head removed to obtain 2048-dimensional feature vectors per spot. Reduce the 2048-d features with PCA, retaining the first 50 principal components as latent characteristics, then train an MLP that maps the 50-d PCA features into the spot-like morphological representation Z^morph. Save the resulting (n_spots x 50) matrix (e.g. morph_embeddings.csv) so it can be fused with expression and spatial-position views inside the MCGAE autoencoder via the self-attention integration step.

## Related tools

- **PyTorch** (Deep-learning framework used to run ResNet50 and train the MLP mapping PCA features to Z^morph) — https://github.com/yiwen-yang/MCGAE
- **torchvision** (Provides the transforms augmentation pipeline applied to spot tiles and the pre-trained ResNet50) — https://github.com/yiwen-yang/MCGAE
- **ResNet50** (Pre-trained CNN backbone (classification head removed) producing 2048-d features per spot tile)
- **scikit-learn** (PCA implementation used to retain the first 50 principal components of the 2048-d features)

## Examples

```
import torchvision.transforms as T; transform = T.Compose([T.ToTensor()]); model = torchvision.models.resnet50(pretrained=True)
```

## Evaluation signals

- Each spot tile is centered on the spot coordinates from the coordinate table — verify tile-to-spot alignment before feature extraction.
- ResNet50 output per spot must be 2048-dimensional (classification head removed).
- PCA-reduced features must be exactly 50-dimensional; the final embedding matrix must have shape (n_spots x 50).
- morph_embeddings.csv rows must be in one-to-one correspondence with the spot coordinate table (no dropped or duplicated spots).
- Downstream MCGAE runs with image view enabled should show improved spatial domain detection versus expression-only, per the article's reported benchmarking.

## Limitations

- The article does not specify which version/weights of the pre-trained ResNet50 were used for image feature extraction.
- The article notes most existing methods (and by implication the image pathway) must be used carefully so spatial image information is efficiently utilized; MCGAE is designed to remain effective without image data.
- Tile augmentation choices from torchvision.transforms are dataset-dependent and are not given as explicit hyperparameters in the README or article text.

## Evidence

- [methods] workflow PCA-50: [evidence span withheld — anchor status: not_found]
- [results] finding PCA-50 MLP: [evidence span withheld — anchor status: not_found]
