---
name: single-cell-transcriptome-proteome-integration-workflow
description: 'Use when paired single-cell transcriptome and protein count matrices
  with reference cell labels need a joint embedding, a clustering resolution sweep,
  and a report of agreement with the supplied labels.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 5
  member_skills:
  - single-cell-multiomics-data-preprocessing
  - single-cell-count-matrix-ingestion
  - anndata-object-construction
  - neighborhood-graph-construction-with-neighbor-pruning
  - knn-neighborhood-graph-construction
  - weighted-nearest-neighbour-graph-construction
  - single-cell-multiomics-integration-training
  - triplet-loss-self-supervised-training
  - multi-omics-embedding-fusion
  - graph-attention-autoencoder-training
  - leiden-clustering-metric-evaluation
  - leiden-clustering-resolution-sweep
  - leiden-clustering-of-integrated-embeddings
  - single-cell-rna-seq-consensus-clustering
  - clustering-agreement-metric-computation--969d24d6c46ee9fc
  - clustering-agreement-metric-computation--89a468bebe16f1ba
  - adjusted-rand-index-evaluation
  member_tools:
  - MOFA
  - SCANPY
  - SSGATE
  - anndata
  - Seurat v5
  - ScMM
  - TotalVI
  - Leiden
  - Leiden algorithm
  - MOFA+
  coverage_gaps: []
  derived_from_workflows: []
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/multi-omics/v1
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# Paired Single-Cell Transcriptome and Proteome Integration

## Summary

Normalize paired RNA and protein counts, construct pruned modality graphs, train a joint representation, and assess clustering across resolutions.


## When to use

Use when paired single-cell transcriptome and protein count matrices with reference cell labels need a joint embedding, a clustering resolution sweep, and a report of agreement with the supplied labels.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — preprocess

**Goal:** paired RNA and protein counts to normalized modality matrices

**EDAM operation:** http://edamontology.org/operation_2935

**Inputs:** h5ad/rna-counts, h5ad/protein-counts · **Outputs:** h5ad/paired-normalized

**Candidate leaf skills:** `single-cell-multiomics-data-preprocessing` (primary), `single-cell-count-matrix-ingestion`, `anndata-object-construction`

**Tools (primary):** MOFA, SCANPY, SSGATE, anndata

**Other candidate tools:** MUON, NumPy, SnapATAC, pandas

**Grounding:** 2 KB(s); DOIs: 10.1093/bib/bbae450, 10.1186/s13059-021-02577-8

### Stage 2 — neighbors

**Goal:** normalized modality matrices to separate pruned cell-neighbor graphs

**EDAM operation:** http://edamontology.org/operation_3432

**Inputs:** h5ad/paired-normalized · **Outputs:** h5ad/paired-graphs

**Candidate leaf skills:** `neighborhood-graph-construction-with-neighbor-pruning` (primary), `knn-neighborhood-graph-construction`, `weighted-nearest-neighbour-graph-construction`

**Tools (primary):** SCANPY, SSGATE, Seurat v5

**Other candidate tools:** ClusterProfiler, Leiden, MOFA, Seurat, anndata, MOFA+, MUON, NumPy, pandas

**Grounding:** 2 KB(s); DOIs: 10.1093/bib/bbae450, 10.1186/s13059-021-02577-8

### Stage 3 — integrate

**Goal:** paired expression matrices and graphs to a joint cell embedding

**EDAM operation:** http://edamontology.org/operation_3935

**Inputs:** h5ad/paired-graphs · **Outputs:** h5ad/joint-embedding, model-state

**Candidate leaf skills:** `single-cell-multiomics-integration-training` (primary), `triplet-loss-self-supervised-training`, `multi-omics-embedding-fusion`, `graph-attention-autoencoder-training`, `leiden-clustering-metric-evaluation`

**Tools (primary):** MOFA, SCANPY, SSGATE, ScMM, Seurat v5, TotalVI

**Other candidate tools:** anndata, ClusterProfiler, Seurat

**Grounding:** 1 KB(s); DOIs: 10.1093/bib/bbae450

### Stage 4 — cluster

**Goal:** joint embedding to cell-cluster assignments across resolutions

**EDAM operation:** http://edamontology.org/operation_3432

**Inputs:** h5ad/joint-embedding, tsv/reference-cell-labels · **Outputs:** h5ad/clustered, tsv/cluster-assignments

**Candidate leaf skills:** `leiden-clustering-resolution-sweep` (primary), `leiden-clustering-of-integrated-embeddings`, `single-cell-rna-seq-consensus-clustering`

**Tools (primary):** Leiden, Leiden algorithm, MOFA, SCANPY, SSGATE, ScMM, Seurat v5, TotalVI

**Other candidate tools:** ClusterProfiler, Seurat, CHAI, CHAI-SNF, R, SAME-clustering, SC3, SNFtool, Spectrum

**Grounding:** 2 KB(s); DOIs: 10.1093/bib/bbae411, 10.1093/bib/bbae450

### Stage 5 — assess

**Goal:** cell clusters and reference labels to a clustering-concordance report

**EDAM operation:** http://edamontology.org/operation_0337

**Inputs:** h5ad/clustered, tsv/cluster-assignments, tsv/reference-cell-labels · **Outputs:** h5ad/assessed-integration, tsv/clustering-concordance

**Candidate leaf skills:** `clustering-agreement-metric-computation--969d24d6c46ee9fc` (primary), `clustering-agreement-metric-computation--89a468bebe16f1ba`, `adjusted-rand-index-evaluation`

**Tools (primary):** MOFA, MOFA+, SCANPY, SSGATE, ScMM, Seurat v5, TotalVI

**Other candidate tools:** SMOPCA, SpatialPCA, scMDC, scikit-learn, MOSim, R, SPARSim, Seurat

**Grounding:** 3 KB(s); DOIs: 10.1093/bib/bbae450, 10.1101/421834, 10.1186/s13059-025-03576-9

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
