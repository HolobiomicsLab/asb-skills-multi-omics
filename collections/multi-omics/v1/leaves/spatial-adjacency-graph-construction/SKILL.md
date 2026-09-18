---
name: spatial-adjacency-graph-construction
description: Use when you have spatial transcriptomics data (Visium spots or Slide-seq
  beads) with normalized expression plus spatial coordinates and need adjacency matrices
  and >=2 expression views for a multi-view graph model.
when_to_use_negative:
- Datasets so large that O(n^2) adjacency computation would exhaust computational
  resources (the paper flags this scalability limit and suggests subgraph sampling
  or deep multichannel GCN strategies instead)
- Non-spatial single-cell expression data without spot/bead coordinates, since adjacency
  matrices cannot be built from coordinates
- Datasets where a single predefined similarity metric suffices and multi-view diversity
  adds no value
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MCGAE
  role: Framework whose multi-view construction module consumes the adjacency matrices
    and enhanced expression views; supports user-selected enhancement methods such
    as simple autoencoders
  repo: github.com/yiwen-yang/MCGAE
- name: Scanpy/AnnData
  role: Loading the normalized expression matrix and spot coordinates before view
    and adjacency construction
merged_aliases:
- spatial-transcriptomics-graph-construction
merged_alias_records:
- alias: spatial-transcriptomics-graph-construction
  slug: spatial-transcriptomics-graph-construction
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:2c16d340b4f857f0c79d4b036dab025b65aeb2a5552a3a5f8a650b96e71e7b47
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
  - Scanpy/AnnData
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# spatial-adjacency-graph-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construction of multiple spatial adjacency matrices and multi-view representations from spot/bead coordinates and gene expression, as used by MCGAE to feed its multi-view contrastive graph autoencoder.

## When to use

Use when you have spatial transcriptomics data (Visium spots or Slide-seq beads) with normalized expression plus spatial coordinates and need adjacency matrices and >=2 expression views for a multi-view graph model.

## When NOT to use

- Datasets so large that O(n^2) adjacency computation would exhaust computational resources (the paper flags this scalability limit and suggests subgraph sampling or deep multichannel GCN strategies instead)
- Non-spatial single-cell expression data without spot/bead coordinates, since adjacency matrices cannot be built from coordinates
- Datasets where a single predefined similarity metric suffices and multi-view diversity adds no value

## Inputs

- Normalized spatial gene-expression matrix (e.g. Scanpy/AnnData object, DLPFC slice #151669 or CRLM 10x Visium count matrix)
- Spot or bead spatial coordinates (2D per section; per-section coordinates for 3D multi-slice input)
- Optional H&E histology image (10x Visium) for downstream morphological feature views
- Choice of neighborhood/similarity criteria (kNN k and an alternative similarity metric)

## Outputs

- multi_view_data.npz containing >=2 enhanced expression views with shape (n_spots, p) and >=2 spatial adjacency matrices with shape (n_spots, n_spots)
- Recorded per-view and per-adjacency array shapes
- Optional inter-batch adjacency distances linking neighboring sections for 3D reconstruction

## How to apply

Load the normalized expression matrix and spot coordinates (e.g. DLPFC slice #151669 via Scanpy/AnnData). Build at least two enhanced gene-expression views: one by augmenting expression profiles with spot spatial coordinates, another using a distinct spot-spot similarity metric applied to the expression profiles. Build at least two spatial adjacency matrices from the coordinates using different criteria, e.g. a k-nearest-neighbor graph versus an alternative neighborhood/similarity rule. For 3D reconstruction across consecutive sections (e.g. seven Slide-seq hippocampal sections), additionally compute the inter-batch adjacency distance between neighboring sections to link adjacent slices and enhance cross-section similarity. Package the views and adjacency matrices (arrays of shape (n_spots, p) and (n_spots, n_spots) respectively) into a single artifact such as multi_view_data.npz with per-view shapes recorded. Varying the similarity metrics across views is the rationale: MCGAE avoids reliance on a single predefined metric since graph structures vary with metric choice.

## Related tools

- **MCGAE** (Framework whose multi-view construction module consumes the adjacency matrices and enhanced expression views; supports user-selected enhancement methods such as simple autoencoders) — github.com/yiwen-yang/MCGAE
- **Scanpy/AnnData** (Loading the normalized expression matrix and spot coordinates before view and adjacency construction)

## Examples

```
import numpy as np; np.savez('multi_view_data.npz', expr_view1=view1, expr_view2=view2, adj_knn=A1, adj_alt=A2)
```

## Evaluation signals

- multi_view_data.npz exists and contains at least 2 expression views and at least 2 adjacency matrices
- View arrays have shape (n_spots, p) and adjacency arrays have shape (n_spots, n_spots)
- The generation script runs end-to-end without error
- The two adjacency matrices derive from different neighborhood/similarity criteria (e.g. kNN vs. an alternative rule)
- For 3D input, inter-batch adjacency edges exist between every pair of neighboring sections and downstream clustering delineates expected structures (e.g. hippocampal regions on UMAP)

## Limitations

- Adjacency matrix computation has O(n^2) complexity, which may rapidly exhaust computational resources with large sample counts
- The choice of similarity metric is challenging because graph structures vary with the metric selected
- The distance-based batch-effect strategy between neighboring regions may not suffice for other complex datasets

## Evidence

- [discussion] Scalability limit: [evidence span withheld — anchor status: not_found]
