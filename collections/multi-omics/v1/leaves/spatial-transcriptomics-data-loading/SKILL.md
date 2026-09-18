---
name: spatial-transcriptomics-data-loading
description: 'Use when raw spatial transcriptomics data (expression matrix plus spot
  coordinates, e.g. DLPFC slice #151669 in AnnData/h5ad) must be normalized and loaded
  before multi-view construction or spatial clustering.'
when_to_use_negative:
- Input is single-cell (non-spatial) expression data without spot/coordinate information,
  where spatial adjacency graphs cannot be built.
- Expression data are raw unnormalized counts and no normalization step will be run
  first, since enhanced views and similarity metrics assume normalized profiles.
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Scanpy/AnnData
  role: 'Loads the normalized DLPFC slice #151669 expression matrix and spot coordinates
    as AnnData objects for multi-view construction'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:dd050f35dc7700f1d6a0518e91d022ad69c10f07ef7cabe95182d130ef5300d1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/yiwen-yang/MCGAE
  tools:
  - Scanpy/AnnData
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.2857142857142857
n_claims: 7
---

# spatial-transcriptomics-data-loading

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Load normalized spatial transcriptomics expression matrices and spot coordinates (e.g. a DLPFC slice via Scanpy/AnnData) as the entry point for building MCGAE's multi-view graph inputs.

## When to use

Use when raw spatial transcriptomics data (expression matrix plus spot coordinates, e.g. DLPFC slice #151669 in AnnData/h5ad) must be normalized and loaded before multi-view construction or spatial clustering.

## When NOT to use

- Input is single-cell (non-spatial) expression data without spot/coordinate information, where spatial adjacency graphs cannot be built.
- Expression data are raw unnormalized counts and no normalization step will be run first, since enhanced views and similarity metrics assume normalized profiles.

## Inputs

- normalized spatial transcriptomics gene expression matrix (n_spots x p genes)
- spot spatial coordinates (x, y per spot)
- AnnData/h5ad object readable by Scanpy (e.g. DLPFC slice #151669)

## Outputs

- Scanpy/AnnData object holding the expression matrix and spot coordinates ready for multi-view construction
- expression and coordinate arrays validated for shape (n_spots, p) and consistent spot ordering

## How to apply

Load the normalized gene expression matrix and spot coordinates with Scanpy/AnnData (h5ad-style containers), as done for the DLPFC slice #151669 in the MCGAE workflow. Confirm that the expression matrix is spot-by-gene (n_spots, p) and that spot spatial coordinates are stored so downstream adjacency (e.g. k-NN) graphs and enhanced expression views can be derived from them. Normalization should be applied before multi-view enhancement, since MCGAE's enhanced views augment expression with spatial coordinates and with spot-spot similarity metrics. The loaded AnnData object is then packaged into multi_view_data.npz with at least 2 expression views (n_spots, p) and 2 adjacency matrices (n_spots, n_spots).

## Related tools

- **Scanpy/AnnData** (Loads the normalized DLPFC slice #151669 expression matrix and spot coordinates as AnnData objects for multi-view construction)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('151669.h5ad')
```

## Evaluation signals

- Expression array has shape (n_spots, p) and coordinate array has n_spots rows with matching spot order.
- Downstream multi_view_data.npz contains >=2 expression views of shape (n_spots, p) and >=2 adjacency matrices of shape (n_spots, n_spots).
- Generation script runs without error on the loaded slice (e.g. DLPFC #151669).
- Expression values are normalized prior to view augmentation with spatial coordinates or similarity metrics.

## Limitations

- Adjacency computation from spatial coordinates has O(n^2) complexity, limiting scalability to large spot counts.
- Effective functioning downstream assumes gene expression, spatial positions, and optionally image information can be integrated; datasets lacking spatial coordinates cannot be processed.

## Evidence

- [other] Load the normalized DLPFC slice #151669 expression matrix and spot coordinates with Scanpy/AnnData.: [evidence span withheld — anchor status: not_found]
