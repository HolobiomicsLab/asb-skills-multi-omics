---
name: visium-slice-preprocessing
description: Use when raw 10x Visium spot-level count matrices with spot spatial coordinates
  must be quality-filtered, normalized, and turned into gene-expression and spatial-adjacency
  views before graph autoencoder-based spatial domain clustering.
when_to_use_negative:
- Input data has no spot-level spatial coordinates, so no spatial adjacency view can
  be constructed (MCGAE falls back on expression-only views and the spatial-domain
  task is not applicable)
- Analysis goal is standard non-spatial clustering of expression data alone (e.g.,
  Seurat-style linear dimensionality reduction), which the article identifies as overlooking
  vital spatial relationships
- Sample counts are extremely large, since the O(n^2) adjacency matrix computation
  may rapidly exhaust computational resources
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MCGAE
  role: Multi-view contrastive graph autoencoder that consumes the preprocessed expression
    and spatial adjacency views to produce the unified embedding used for spatial
    domain detection
  repo: github.com/yiwen-yang/MCGAE
- name: STAGATE
  role: Graph attention autoencoder baseline run on identically preprocessed slices
    to compute per-slice ARI benchmarks
- name: BayesSpace
  role: Bayesian spatial baseline run on identically preprocessed slices to compute
    per-slice ARI benchmarks
- name: Seurat
  role: Referenced non-spatial normalization/clustering approach whose expression-only
    linear dimensionality reduction the spatial multi-view preprocessing extends
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:5b56babddd4f61ffdc22a775dcfdbe0a3325227aefb819a732bf069e0c5f6521
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
  - STAGATE
  - Seurat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# visium-slice-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocesses 10x Visium spatial transcriptomics slices (e.g., the 12 LIBD human DLPFC slices) via spot quality filtering, gene expression normalization, and construction of a spatial adjacency view from spot coordinates, producing the multi-view inputs required for MCGAE's contrastive graph autoencoder.

## When to use

Use when raw 10x Visium spot-level count matrices with spot spatial coordinates must be quality-filtered, normalized, and turned into gene-expression and spatial-adjacency views before graph autoencoder-based spatial domain clustering.

## When NOT to use

- Input data has no spot-level spatial coordinates, so no spatial adjacency view can be constructed (MCGAE falls back on expression-only views and the spatial-domain task is not applicable)
- Analysis goal is standard non-spatial clustering of expression data alone (e.g., Seurat-style linear dimensionality reduction), which the article identifies as overlooking vital spatial relationships
- Sample counts are extremely large, since the O(n^2) adjacency matrix computation may rapidly exhaust computational resources

## Inputs

- 10x Visium spot-level raw count matrix per slice (e.g., the 12 LIBD human DLPFC slices)
- Spot spatial coordinates for each slice (used to build the spatial adjacency matrix)
- Manual layer annotations per spot (for ARI validation in benchmark settings)
- Optional H&E histology image per slice for morphological feature extraction

## Outputs

- Quality-filtered and normalized gene expression matrix per slice
- Spatial adjacency view/matrix constructed from spot coordinates
- Modality-specific spatial representations integrated into a unified spot embedding per slice
- Per-spot spatial domain cluster labels and per-slice Adjusted Rand Index against manual layer annotations

## How to apply

For each Visium slice, load the spot-level count matrix together with the spot coordinates and (if benchmarking) the manual layer annotations. Apply spot quality filtering, normalize the gene expression matrix, and build the spatial adjacency view directly from the spot coordinate pairs so that each spot is a node linked to its physical neighbors. Keep gene expression and spatial adjacency as separate views, since MCGAE 'creates multi-view representations from gene expression and spatial adjacency matrices' and then 'generates modality-specific spatial representations and integrates them into a unified embedding'; when a histology image is available, extract morphological image features (the paper uses a pre-trained ResNet50 network for image feature extraction) as an optional third view. Feed the resulting multi-view object per slice into the MCGAE training loop (github.com/yiwen-yang/MCGAE) and cluster the unified embedding into spatial domains, validating against manual annotations with the Adjusted Rand Index as in the DLPFC benchmark (MCGAE mean ARI ~0.52 vs STAGATE ~0.41).

## Related tools

- **MCGAE** (Multi-view contrastive graph autoencoder that consumes the preprocessed expression and spatial adjacency views to produce the unified embedding used for spatial domain detection) — github.com/yiwen-yang/MCGAE
- **STAGATE** (Graph attention autoencoder baseline run on identically preprocessed slices to compute per-slice ARI benchmarks)
- **BayesSpace** (Bayesian spatial baseline run on identically preprocessed slices to compute per-slice ARI benchmarks)
- **Seurat** (Referenced non-spatial normalization/clustering approach whose expression-only linear dimensionality reduction the spatial multi-view preprocessing extends)

## Evaluation signals

- The assembled per-slice ARI CSV contains exactly 12 rows (one per LIBD DLPFC slice) with columns for MCGAE, STAGATE, and BayesSpace
- MCGAE mean ARI falls within [0.45, 0.60], consistent with the reported mean of 0.52 and median of 0.51
- STAGATE per-slice mean/median ARI approximates 0.41/0.44 on the same preprocessed slices
- Slice #151669 MCGAE ARI approximates 0.63 versus BayesSpace 0.46
- Each slice yields a spatial adjacency view whose graph connects spots according to physical neighbor relationships derived from spot coordinates, consistent with Supplementary Note 5 / Fig. 2A

## Limitations

- Adjacency matrix computation from spatial position information has O(n^2) computational complexity, which may rapidly exhaust computational resources for large sample counts
- The batch effect handling strategy (distance between neighboring regions) may not suffice for other complex datasets
- The pre-trained ResNet50 network used for image feature extraction has no version specified, and the image view is optional (MCGAE functions without image information)

## Evidence

- [discussion] scalability caveat: [evidence span withheld — anchor status: not_found]
