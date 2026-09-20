---
name: multi-view-feature-augmentation
description: Use when preparing spatial transcriptomics data (AnnData with normalized
  expression matrix plus spot coordinates) for a multi-view graph contrastive model,
  and you need >=2 expression views and >=2 adjacency matrices from different similarity
  criteria.
when_to_use_negative:
- When the aim is single-view downstream clustering with a non-spatial method (e.g.,
  Seurat-style linear dimensionality reduction on expression alone), since those overlook
  spatial relationships by design
- 'When spot count is very large: adjacency matrix computation is O(n^2) and may rapidly
  exhaust computational resources; subgraph sampling is needed instead'
- When no spatial coordinates are available, since adjacency views cannot be built
  from spot positions
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Scanpy/AnnData
  role: Loads the normalized DLPFC slice expression matrix and spot coordinates used
    to build the enhanced views and adjacency matrices
- name: MCGAE
  role: Downstream multi-view contrastive graph autoencoder that consumes the packaged
    views and adjacency matrices
  repo: github.com/yiwen-yang/MCGAE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:1a56b4eab1e01c7cda026a6d9ff912b50c057f93286f1b9aa2814ed10d5e4728
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/yiwen-yang/MCGAE
  tools:
  - MCGAE
  - Scanpy/AnnData
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# multi-view-feature-augmentation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs enhanced multi-view inputs for MCGAE by augmenting the normalized gene expression matrix (e.g., DLPFC slice #151669) into multiple expression views and building multiple spatial adjacency matrices from spot coordinates using diverse similarity metrics, packaged as multi_view_data.npz.

## When to use

Use when preparing spatial transcriptomics data (AnnData with normalized expression matrix plus spot coordinates) for a multi-view graph contrastive model, and you need >=2 expression views and >=2 adjacency matrices from different similarity criteria.

## When NOT to use

- When the aim is single-view downstream clustering with a non-spatial method (e.g., Seurat-style linear dimensionality reduction on expression alone), since those overlook spatial relationships by design
- When spot count is very large: adjacency matrix computation is O(n^2) and may rapidly exhaust computational resources; subgraph sampling is needed instead
- When no spatial coordinates are available, since adjacency views cannot be built from spot positions

## Inputs

- Normalized gene expression matrix from an SRT slice (e.g., DLPFC slice #151669) in AnnData/Scanpy format
- Spot spatial coordinates (2D x/y positions per spot)
- Optional histological image for later morphological feature integration (ResNet50)

## Outputs

- multi_view_data.npz containing >=2 enhanced gene-expression views of shape (n_spots, p)
- >=2 spatial adjacency matrices of shape (n_spots, n_spots) derived from different similarity/neighborhood criteria
- Recorded per-view and per-adjacency array shapes

## How to apply

Load the normalized expression matrix and spot coordinates with Scanpy/AnnData. Form a first enhanced gene-expression view by augmenting expression profiles with the spot spatial coordinates, and a second enhanced view using a distinct spot-spot similarity metric applied to expression profiles; MCGAE's modular design lets users choose enhancement methods such as a simple autoencoder. Build a first spatial adjacency matrix from spot coordinates with a k-nearest-neighbor graph, and a second adjacency matrix using an alternative neighborhood/similarity criterion — diversifying metrics matters because GNN methods relying on a single predefined similarity metric suffer from variation in graph structures. Package the >=2 expression views and >=2 adjacency matrices into multi_view_data.npz, recording per-view and per-adjacency array shapes. MCGAE later integrates these views with morphological image features (pre-trained ResNet50) via a self-attention mechanism into a unified embedding.

## Related tools

- **Scanpy/AnnData** (Loads the normalized DLPFC slice expression matrix and spot coordinates used to build the enhanced views and adjacency matrices)
- **MCGAE** (Downstream multi-view contrastive graph autoencoder that consumes the packaged views and adjacency matrices) — github.com/yiwen-yang/MCGAE

## Examples

```
import numpy as np, scanpy as sc; adata = sc.read_h5ad('151669.h5ad'); np.savez('multi_view_data.npz', view1=expr_aug_spatial, view2=expr_aug_similarity, adj_knn=A_knn, adj_alt=A_alt)
```

## Evaluation signals

- multi_view_data.npz exists and contains at least 2 expression views and 2 adjacency matrices
- Each view array has shape (n_spots, p); each adjacency array has shape (n_spots, n_spots)
- The generation script runs end-to-end without error
- The two adjacency matrices are derived from distinct neighborhood/similarity criteria (e.g., KNN vs an alternative metric), not duplicates
- Input AnnData is normalized before view construction, matching the loaded normalized DLPFC slice

## Limitations

- Adjacency matrix computation has O(n^2) complexity, which may rapidly exhaust computational resources as sample counts grow; large-scale datasets require subgraph sampling or deep multichannel GCN strategies
- The batch-effect handling strategy (distance between neighboring regions) may not suffice for other complex datasets
- No version specified for the pre-trained ResNet50 network used for image feature extraction

## Evidence
