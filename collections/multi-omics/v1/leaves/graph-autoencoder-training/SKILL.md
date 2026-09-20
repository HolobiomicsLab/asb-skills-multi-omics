---
name: graph-autoencoder-training
description: Use when spot/bead-level spatial transcriptomics data (e.g. 10x Visium,
  Slide-seq) with spatial coordinates need unsupervised spatial domain detection,
  3D multi-slice reconstruction, or multimodal integration with H&E image features.
when_to_use_negative:
- Very large samples where the O(n^2) spatial adjacency computation would exhaust
  computational resources — the article recommends deferring to future subgraph sampling
  or deep multichannel GCN strategies
- Datasets whose cross-slice batch effects are not adequately handled by the distance-between-neighboring-regions
  adjacency strategy used in the 3D Slide-seq example
- Already-clustered or feature-reduced inputs where no raw spatial coordinates exist
  to construct the adjacency matrix
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: MCGAE
  role: 'Multi-view contrastive graph autoencoder being trained: builds spatial-adjacency
    views, learns modality-specific representations, fuses them via self-attention'
  repo: https://github.com/yiwen-yang/MCGAE
- name: STAGATE
  role: Graph attention autoencoder baseline against which the trained MCGAE embeddings
    are benchmarked by per-slice ARI
- name: BayesSpace
  role: Bayesian spatial baseline method for per-slice ARI comparison in the DLPFC
    benchmark
- name: ResNet50
  role: Pre-trained network used to extract morphological image features per spot
    to enrich the multimodal embedding
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:75afe53794c049b207c68df441d5f769fe9481bcdfcb989f63ea26775613c2f5
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
  - BayesSpace
  - MCGAE
  - ResNet50
  - STAGATE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# graph-autoencoder-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train the MCGAE multi-view contrastive graph autoencoder to build modality-specific spatial representations from gene expression and spatial adjacency matrices, then fuse them via a self-attention mechanism into a unified embedding for spatial domain identification on spatially resolved transcriptomics (SRT) data.

## When to use

Use when spot/bead-level spatial transcriptomics data (e.g. 10x Visium, Slide-seq) with spatial coordinates need unsupervised spatial domain detection, 3D multi-slice reconstruction, or multimodal integration with H&E image features.

## When NOT to use

- Very large samples where the O(n^2) spatial adjacency computation would exhaust computational resources — the article recommends deferring to future subgraph sampling or deep multichannel GCN strategies
- Datasets whose cross-slice batch effects are not adequately handled by the distance-between-neighboring-regions adjacency strategy used in the 3D Slide-seq example
- Already-clustered or feature-reduced inputs where no raw spatial coordinates exist to construct the adjacency matrix

## Inputs

- Spot-level gene expression count matrix (e.g. 10x Visium or Slide-seq) with quality filtering and normalization applied
- Spatial coordinates of spots/beads for building the intra-section adjacency graph
- H&E histology image aligned to spots (optional, for ResNet50 morphological feature extraction)
- Manual layer/region annotations (e.g. DLPFC layer labels) when computing ARI
- Inter-batch adjacency distances between consecutive sections for multi-slice 3D reconstruction

## Outputs

- Unified multimodal spot embedding (plus modality-specific embeddings)
- Per-spot spatial domain label table (e.g. crlm_spatial_domains.csv with coordinates and domain field)
- Per-slice ARI table comparing MCGAE against baselines (e.g. 12-row CSV: MCGAE, STAGATE, BayesSpace)
- 2D UMAP/t-SNE projections overlaid with domain labels

## How to apply

Build multi-view representations from the normalized gene expression matrix and a spatial adjacency matrix constructed from spot/bead coordinates; optionally extract morphological image features per spot with a pre-trained ResNet50. Train the MCGAE contrastive graph autoencoder per slice (or across slices with inter-batch adjacency distance for 3D reconstruction) to obtain modality-specific embeddings, then fuse them through the self-attention mechanism into a unified multimodal embedding that is robust to noise. Cluster the unified embedding to assign each spot a spatial domain label and quantify agreement with manual annotations using the Adjusted Rand Index (ARI); project embeddings with UMAP/t-SNE for visual validation. MCGAE functions effectively even in the absence of image information, so the image view can be dropped when histology is unavailable.

## Related tools

- **MCGAE** (Multi-view contrastive graph autoencoder being trained: builds spatial-adjacency views, learns modality-specific representations, fuses them via self-attention) — https://github.com/yiwen-yang/MCGAE
- **STAGATE** (Graph attention autoencoder baseline against which the trained MCGAE embeddings are benchmarked by per-slice ARI)
- **BayesSpace** (Bayesian spatial baseline method for per-slice ARI comparison in the DLPFC benchmark)
- **ResNet50** (Pre-trained network used to extract morphological image features per spot to enrich the multimodal embedding)

## Examples

```
git clone https://github.com/yiwen-yang/MCGAE  # then train per slice: preprocess Visium counts + spot coordinates, train the MCGAE autoencoder, cluster embeddings, compute ARI
```

## Evaluation signals

- On the 12 LIBD human DLPFC 10x Visium slices, mean MCGAE ARI falls within [0.45, 0.60] (reported mean 0.52, median 0.51), with STAGATE mean/median ≈ 0.41/0.44
- Slice #151669 DLPFC MCGAE ARI ≈ 0.63 vs BayesSpace ≈ 0.46, matching Supplementary Note 5 / Fig. 2A
- CRLM domain-label output table covers all Visium spots with a domain field present, and tumor-invasion region Y resolves into 3 sub-clusters via t-SNE (Fig. 4)
- 3D Slide-seq UMAP of the unified embedding delineates cord-like hippocampal structures across all seven sections with cross-section edges included
- Adjacent-section contrast: including inter-batch adjacency edges visibly improves region separation on the UMAP compared with intra-section-only graphs

## Limitations

- Adjacency matrix computation for spatial position information has O(n^2) complexity, which may rapidly exhaust computational resources with large sample counts
- The 3D batch-effect handling via distance between neighboring regions may not suffice for other complex datasets
- Scaling to large datasets requires future work on subgraph sampling or deep multichannel GCN strategies
- The pre-trained ResNet50 used for image feature extraction has no version specified

## Evidence
