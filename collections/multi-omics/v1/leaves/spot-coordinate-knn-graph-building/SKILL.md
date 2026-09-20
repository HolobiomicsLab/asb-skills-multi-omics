---
name: spot-coordinate-knn-graph-building
description: 'Use when preparing spatial transcriptomics data (e.g., DLPFC slice #151669
  spot coordinates in AnnData) for a graph-based method that requires multiple adjacency-matrix
  views of spot neighborhoods, such as MCGAE.'
when_to_use_negative:
- The dataset lacks per-spot spatial coordinates, so no spatial adjacency can be computed
  from positions
- The number of spots is very large — the adjacency computation incurs O(n^2) complexity
  and may exhaust computational resources; consider subgraph sampling instead
- Only a single predefined similarity graph is needed and multi-view contrastive modeling
  (MCGAE) is not the downstream method
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Scanpy
  role: Loads the normalized DLPFC slice expression matrix and spot coordinates held
    in AnnData format prior to adjacency-matrix construction
- name: STMGCN
  role: Cited prior approach that employs GNNs with contrastive learning and multiple
    adjacency matrices — the design pattern this skill instantiates
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:9f39c0f2c8c5d9033ccb1570655114db3675db50649dce51d8186a90d2db5dff
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/yiwen-yang/MCGAE
  tools:
  - STMGCN
  - Scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# spot-coordinate-knn-graph-building

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs multiple spatial adjacency-matrix views from spot coordinates (e.g., a k-nearest-neighbor graph plus an alternative neighborhood/similarity criterion) as required by MCGAE's multi-view contrastive graph autoencoder, which creates multi-view representations from gene expression and spatial adjacency matrices.

## When to use

Use when preparing spatial transcriptomics data (e.g., DLPFC slice #151669 spot coordinates in AnnData) for a graph-based method that requires multiple adjacency-matrix views of spot neighborhoods, such as MCGAE.

## When NOT to use

- The dataset lacks per-spot spatial coordinates, so no spatial adjacency can be computed from positions
- The number of spots is very large — the adjacency computation incurs O(n^2) complexity and may exhaust computational resources; consider subgraph sampling instead
- Only a single predefined similarity graph is needed and multi-view contrastive modeling (MCGAE) is not the downstream method

## Inputs

- Normalized spatial transcriptomics expression matrix (AnnData/Scanpy object, e.g., DLPFC slice #151669)
- Spot spatial coordinates (e.g., adata.obsm['spatial'])

## Outputs

- multi_view_data.npz containing >=2 enhanced gene-expression views, each of shape (n_spots, p)
- >=2 spatial adjacency matrices, each of shape (n_spots, n_spots) (k-NN graph and an alternative neighborhood/similarity criterion)
- Recorded per-view and per-adjacency array shapes

## How to apply

Load the normalized expression matrix and spot coordinates with Scanpy/AnnData (e.g., DLPFC slice #151669). Build a first spatial adjacency matrix from the spot coordinates using a k-nearest-neighbor graph, then build a second spatial adjacency matrix using an alternative neighborhood/similarity criterion so the downstream model sees diverse graph structures — the article motivates this by noting GNN methods 'rely on predefined specific similarity metrics, creating challenges in selecting the most effective metric.' MCGAE is modular, so the enhancement/similarity methods are user-selectable choices. Package the resulting adjacency matrices (each of shape (n_spots, n_spots)) together with the >=2 enhanced expression views (each of shape (n_spots, p)) into a multi_view_data.npz artifact, recording per-view and per-adjacency array shapes. Be aware the adjacency computation from spatial positions carries O(n^2) cost.

## Related tools

- **Scanpy** (Loads the normalized DLPFC slice expression matrix and spot coordinates held in AnnData format prior to adjacency-matrix construction)
- **STMGCN** (Cited prior approach that employs GNNs with contrastive learning and multiple adjacency matrices — the design pattern this skill instantiates)

## Examples

```
import scanpy as sc; from sklearn.neighbors import kneighbors_graph; adata = sc.read_h5ad('<dlpfc_151669>.h5ad'); A_knn = kneighbors_graph(adata.obsm['spatial'], n_neighbors=k, mode='connectivity')
```

## Evaluation signals

- multi_view_data.npz exists and contains at least 2 expression views and 2 adjacency matrices
- Every expression-view array has shape (n_spots, p) matching the spot count of the source AnnData object
- Every adjacency array has shape (n_spots, n_spots) and is derivable from the spot coordinates
- The adjacency-generation script runs without error end-to-end
- The two adjacency matrices come from distinct neighborhood/similarity criteria (e.g., k-NN vs. an alternative), not identical copies

## Limitations

- Adjacency-matrix computation from spatial position information incurs O(n^2) complexity, which may rapidly exhaust computational resources as sample counts grow
- For large-scale datasets, MCGAE would need subgraph sampling or deep multichannel GCN strategies rather than full dense adjacency matrices
- Choice of similarity metric for each view is a user decision (modular design); poor metric selection changes graph structure and downstream spatial-domain identification

## Evidence

- [discussion] discussion: [evidence span withheld — anchor status: not_found]
