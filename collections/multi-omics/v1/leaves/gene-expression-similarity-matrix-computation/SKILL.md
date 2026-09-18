---
name: gene-expression-similarity-matrix-computation
description: Use when you have a normalized spatial transcriptomics expression matrix
  with spot coordinates and need multiple graph views (distinct similarity metrics)
  for multi-view graph autoencoder clustering.
when_to_use_negative:
- The dataset is very large and only one graph view is needed — the article notes
  adjacency matrix computation for spatial position information incurs O(n^2) computational
  complexity, which may rapidly exhaust computational resources
- The input is a plain feature table without spot spatial coordinates — the spatial
  adjacency views in this module require spot positions
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Scanpy
  role: Loads and normalizes the spatial transcriptomics expression matrix and spot
    coordinates used to build the similarity/adjacency views
- name: MCGAE
  role: Reference multi-view contrastive graph autoencoder implementation whose modular
    modeling approach exposes these similarity-metric choices
  repo: github.com/yiwen-yang/MCGAE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:df3b50503b323dfca190bc3fb81fad31078d6dbe1a26a0b76642eb74d96ebdaa
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
  - Scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gene-expression-similarity-matrix-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs multiple spot-spot similarity/adjacency views from a normalized spatial transcriptomics expression matrix (e.g. DLPFC slice #151669 in AnnData) using diverse similarity metrics, as in MCGAE's multi-view graph construction module.

## When to use

Use when you have a normalized spatial transcriptomics expression matrix with spot coordinates and need multiple graph views (distinct similarity metrics) for multi-view graph autoencoder clustering.

## When NOT to use

- The dataset is very large and only one graph view is needed — the article notes adjacency matrix computation for spatial position information incurs O(n^2) computational complexity, which may rapidly exhaust computational resources
- The input is a plain feature table without spot spatial coordinates — the spatial adjacency views in this module require spot positions

## Inputs

- Normalized gene expression matrix for a spatial transcriptomics slice (AnnData/Scanpy object, e.g. DLPFC slice #151669)
- Spot coordinates (spatial positions for each spot)
- Choice of >=2 spot-spot similarity metrics (e.g. kNN graph plus an alternative neighborhood/similarity criterion)

## Outputs

- multi_view_data.npz containing >=2 enhanced expression views of shape (n_spots, p)
- >=2 spatial adjacency matrices of shape (n_spots, n_spots) derived from spot coordinates
- Recorded per-view and per-adjacency array shapes for validation

## How to apply

Load the normalized expression matrix and spot coordinates with Scanpy/AnnData (e.g. DLPFC slice #151669), then build several spot-spot similarity views rather than committing to a single predefined metric, since MCGAE's authors note that relying on one specific similarity metric creates challenges due to variation in graph structures. Form a first enhanced expression view by augmenting expression profiles with the spot spatial coordinates, and a second enhanced view by applying a distinct spot-spot similarity metric to the expression profiles. In parallel, build adjacency matrices from the coordinates using a k-nearest-neighbor graph and an alternative neighborhood/similarity criterion. Package the >=2 expression views and >=2 adjacency matrices into multi_view_data.npz, recording per-view and per-adjacency array shapes.

## Related tools

- **Scanpy** (Loads and normalizes the spatial transcriptomics expression matrix and spot coordinates used to build the similarity/adjacency views)
- **MCGAE** (Reference multi-view contrastive graph autoencoder implementation whose modular modeling approach exposes these similarity-metric choices) — github.com/yiwen-yang/MCGAE

## Examples

```
import scanpy as sc; from sklearn.neighbors import kneighbors_graph; adata = sc.read_h5ad('151669.h5ad'); A_knn = kneighbors_graph(adata.obsm['spatial'], n_neighbors=6, mode='connectivity')
```

## Evaluation signals

- multi_view_data.npz exists and contains at least 2 expression views and 2 adjacency matrices
- View arrays have shape (n_spots, p); adjacency arrays have shape (n_spots, n_spots)
- The generation script runs without error end-to-end
- Each adjacency matrix is derived from a distinct similarity/neighborhood criterion (kNN vs. an alternative), not duplicated copies
- Expression views retain all n_spots spots and p genes from the normalized AnnData matrix

## Limitations

- Adjacency matrix computation for spatial position information has O(n^2) complexity, limiting scalability to large sample counts; the authors suggest subgraph sampling or deep multichannel GCN strategies for large-scale data
- Metric selection is challenging because different similarity metrics yield variation in graph structures, so results depend on the chosen metrics

## Evidence

- [discussion] the computation of the adjacency matrix for spatial position information incurs a computational complexity of $O\left({n}^2\right)$, which may rapidly exhaust computational resources: [evidence span withheld — anchor status: not_found]
