---
name: spatial-domain-identification
description: Use when clustering spots from 10x Visium or similar SRT data into spatial
  domains, especially when gene expression, spot coordinates, and H&E image features
  are available and a benchmark ARI against manual layer annotations is required.
when_to_use_negative:
- When only expression data with no spatial coordinates or image is available and
  a non-spatial method like Seurat's linear dimensionality reduction suffices
- When very large spot counts make the O(n^2) spatial adjacency matrix computation
  infeasible without subgraph sampling
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MCGAE
  role: Multi-view contrastive graph autoencoder that builds multi-view representations
    from gene expression and spatial adjacency matrices, fuses modalities via self-attention,
    and clusters the unified embedding into spatial domains
  repo: github.com/yiwen-yang/MCGAE
- name: ResNet50
  role: Pre-trained network used by MCGAE to extract morphological image features
    per spot for the multimodal integration
- name: STAGATE
  role: Graph attention autoencoder baseline run on the same 12 DLPFC slices for per-slice
    ARI comparison
- name: BayesSpace
  role: Bayesian spatial method baseline run on the same 12 DLPFC slices for per-slice
    ARI comparison
merged_aliases:
- spatial-domain-identification-srt
merged_alias_records:
- alias: spatial-domain-identification-srt
  slug: spatial-domain-identification-srt
  jaccard_score: 0.75
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:801426cb2ac02684af5a67c848c3a450d4584f7b1542275949d14a5a423c9e99
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: thin
yes_rate: 0.0
n_claims: 0
---

# spatial-domain-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply MCGAE, a multi-view contrastive graph autoencoder that fuses gene expression, spatial adjacency, and morphological image features via self-attention, to identify spatial domains in spatially resolved transcriptomics (SRT) data. The skill reproduces the paper's benchmarks on 12 human DLPFC 10x Visium slices (mean ARI 0.52) and tumor-invasion region detection in colorectal cancer liver metastasis (CRLM) data.

## When to use

Use when clustering spots from 10x Visium or similar SRT data into spatial domains, especially when gene expression, spot coordinates, and H&E image features are available and a benchmark ARI against manual layer annotations is required.

## When NOT to use

- When only expression data with no spatial coordinates or image is available and a non-spatial method like Seurat's linear dimensionality reduction suffices
- When very large spot counts make the O(n^2) spatial adjacency matrix computation infeasible without subgraph sampling

## Inputs

- 10x Visium count matrix (e.g., the 12 LIBD human DLPFC slices with manual layer annotations, or the CRLM 10x Visium count matrix)
- Spot spatial coordinates for constructing the spatial adjacency matrix
- H&E histology image for ResNet50-based morphological feature extraction (optional; MCGAE functions without image information)
- Manual layer/domain annotations for computing ARI in benchmarking

## Outputs

- Per-spot spatial domain labels (e.g., crlm_spatial_domains.csv covering all Visium spots with a domain field)
- Per-slice ARI table (12-row CSV with columns for MCGAE, STAGATE, BayesSpace plus median and mean per method)
- Unified multimodal spot embedding; t-SNE plot of region-Y spots resolving 3 sub-clusters (Fig. 4)

## How to apply

Build multi-view representations from the normalized gene expression matrix and the spatial adjacency matrix constructed from spot coordinates; extract per-spot morphological image features with a pre-trained ResNet50 when histology is available. Train the MCGAE multi-view contrastive graph autoencoder per slice, using its self-attention mechanism to fuse modality-specific spatial representations into a unified multimodal embedding; the method also works without image information. Cluster the unified embedding into spatial domains and, for benchmarking, compute the Adjusted Rand Index (ARI) against manual layer annotations. For multi-slice 3D reconstruction, enhance similarity between adjacent slices by calculating the adjacency distance of neighboring batches. Baseline comparisons should include STAGATE (graph attention autoencoder) and BayesSpace (Bayesian spatial method) on the same slices. For the CRLM tumor-invasion analysis, annotate the domain spanning tumor and connective tissue as region Y and verify its sub-structure via t-SNE (expect 3 main clusters, Cluster 0 dominated by tumor cells).

## Related tools

- **MCGAE** (Multi-view contrastive graph autoencoder that builds multi-view representations from gene expression and spatial adjacency matrices, fuses modalities via self-attention, and clusters the unified embedding into spatial domains) — github.com/yiwen-yang/MCGAE
- **ResNet50** (Pre-trained network used by MCGAE to extract morphological image features per spot for the multimodal integration)
- **STAGATE** (Graph attention autoencoder baseline run on the same 12 DLPFC slices for per-slice ARI comparison)
- **BayesSpace** (Bayesian spatial method baseline run on the same 12 DLPFC slices for per-slice ARI comparison)

## Examples

```
git clone https://github.com/yiwen-yang/MCGAE.git
```

## Evaluation signals

- The ARI CSV exists with exactly 12 slice rows (one row per DLPFC slice, columns for MCGAE, STAGATE, BayesSpace)
- MCGAE mean ARI lies in [0.45, 0.60], consistent with the reported mean 0.52 and median 0.51
- STAGATE mean/median approximate 0.41/0.44, and slice #151669 MCGAE ARI approximates 0.63 vs BayesSpace 0.46
- CRLM per-spot domain table covers all Visium spots and includes a domain field; region Y resolves into 3 sub-clusters via t-SNE with lymphatic structures separating from tumor domains
- The table is consistent with Supplementary Note 5 / Fig. 2A of the paper

## Limitations

- Adjacency matrix computation for spatial position information incurs O(n^2) computational complexity, which may rapidly exhaust computational resources with large sample counts
- The batch-effect strategy of considering distance between neighboring regions may not suffice for other complex datasets
- Large-scale datasets will require subgraph sampling or deep multichannel GCN strategies in future work
- No version specified for the pre-trained ResNet50 network used for image feature extraction

## Evidence
