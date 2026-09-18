---
name: visium-spatial-transcriptomics-processing
description: Use when input is a 10x Visium count matrix with spatial coordinates
  (and optionally an H&E histology image) and the goal is spatial domain identification
  or benchmarking ARI against methods like SpaGCN and STAGATE.
when_to_use_negative:
- When no spatial coordinates are available and only expression data exist — non-spatial
  algorithms like Seurat's linear dimensionality reduction may suffice
- 'When the dataset is very large: adjacency matrix computation has O(n^2) complexity
  and ''may rapidly exhaust computational resources'''
- When the sole aim is simple expression-based clustering without spatial domain structure
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MCGAE
  role: 'Main framework: multi-view contrastive graph autoencoder integrating gene
    expression, spatial positions, and image features for spatial domain identification'
  repo: github.com/yiwen-yang/MCGAE
- name: ResNet50
  role: Pre-trained network used to extract morphological image features from the
    H&E histology image
- name: SpaGCN
  role: Baseline integrating image data into a neighborhood graph using graph convolutional
    networks, run for ARI comparison
- name: STAGATE
  role: Graph/attention-based baseline for spatial domain identification, run for
    ARI comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:d851b26cc553074c56b51370f4fb31084ff15af5de63d0688d597b54e699d9e1
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
  - MCGAE
  - ResNet50
  - STAGATE
  - SpaGCN
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# visium-spatial-transcriptomics-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Process a 10x Visium spatial transcriptomics section (e.g., Human Breast Cancer) by building multi-view representations from gene expression and the spatial adjacency matrix, optionally fusing ResNet50-derived H&E morphological features via MCGAE's self-attention mechanism to improve spatial domain clustering (ARI).

## When to use

Use when input is a 10x Visium count matrix with spatial coordinates (and optionally an H&E histology image) and the goal is spatial domain identification or benchmarking ARI against methods like SpaGCN and STAGATE.

## When NOT to use

- When no spatial coordinates are available and only expression data exist — non-spatial algorithms like Seurat's linear dimensionality reduction may suffice
- When the dataset is very large: adjacency matrix computation has O(n^2) complexity and 'may rapidly exhaust computational resources'
- When the sole aim is simple expression-based clustering without spatial domain structure

## Inputs

- 10x Visium gene expression count matrix (e.g., Human Breast Cancer section)
- Spatial coordinates of spots
- Spatial adjacency matrix derived from spot positions
- H&E stained histology image (optional, for morphological feature extraction with ResNet50)
- Ground-truth spatial domain annotations (for ARI evaluation)

## Outputs

- Spatial domain cluster assignments per spot
- Multimodal unified embedding of spots
- ARI comparison table across methods (e.g., breast_cancer_ari_comparison.csv: MCGAE-without-image ~0.53, MCGAE-with-image ~0.64, SpaGCN ~0.56, STAGATE ~0.57)
- Reconstructed 3D spatial domains when processing multiple adjacent slices

## How to apply

Load the 10x Visium count matrix, spatial coordinates, and (if available) the H&E image, extracting morphological image features with a pre-trained ResNet50 network. Build multi-view representations from gene expression and the spatial adjacency matrix to train MCGAE; when image data are present, fuse gene expression, spatial positions, and morphological features through the self-attention mechanism into a unified multimodal embedding. Cluster the resulting embedding to assign spatial domains, and compute the Adjusted Rand Index (ARI) against ground-truth annotations. On the Human Breast Cancer Visium dataset, expect the image-free ARI near 0.53 and the image-integrated ARI near 0.64 (~20.75% relative lift), compared with SpaGCN (~0.56) and STAGATE (~0.57); tabulate results (e.g., breast_cancer_ari_comparison.csv) and check the with-image ARI falls within [0.60, 0.68]. The method also supports 3D reconstruction across adjacent slices by calculating the adjacency distance of neighboring batches.

## Related tools

- **MCGAE** (Main framework: multi-view contrastive graph autoencoder integrating gene expression, spatial positions, and image features for spatial domain identification) — github.com/yiwen-yang/MCGAE
- **ResNet50** (Pre-trained network used to extract morphological image features from the H&E histology image)
- **SpaGCN** (Baseline integrating image data into a neighborhood graph using graph convolutional networks, run for ARI comparison)
- **STAGATE** (Graph/attention-based baseline for spatial domain identification, run for ARI comparison)

## Evaluation signals

- MCGAE-with-image ARI on the Human Breast Cancer Visium section falls within [0.60, 0.68] (expected ~0.64)
- Relative ARI lift over the image-free condition (~0.53) reproduces the ~20.75% improvement
- ARI comparison table includes SpaGCN (~0.56) and STAGATE (~0.57) run on the same breast cancer section
- Multi-view representations are built from both gene expression and the spatial adjacency matrix, not expression alone
- When images are integrated, morphological features from ResNet50 enter the embedding via the self-attention mechanism

## Limitations

- Adjacency matrix computation for spatial position information incurs O(n^2) computational complexity, which may rapidly exhaust computational resources with large sample counts
- Batch effect handling via distance between neighboring regions 'may not suffice for other complex datasets'
- Large-scale datasets need future strategies such as subgraph sampling or deep multichannel GCN approaches
- No version is specified for the pre-trained ResNet50 network used for image feature extraction

## Evidence

- [results] Integrating H&E stained images with MCGAE on the Human Breast Cancer dataset increased the average ARI from 0.53 to 0.64, a 20.75% improvement: [evidence span withheld — anchor status: not_found]
- [discussion] O(n^2) adjacency matrix complexity limitation: [evidence span withheld — anchor status: not_found]
