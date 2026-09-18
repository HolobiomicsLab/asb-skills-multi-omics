---
name: ari-benchmark-computation
description: Use when benchmarking spatial transcriptomics domain-detection methods
  on Visium slices that have manual layer annotations, and you need per-slice ARI
  values against ground truth plus median/mean summaries to compare methods.
when_to_use_negative:
- Slices lack manual layer annotations or any ground-truth domain labels, so ARI cannot
  be computed against a reference
- Data are not spatially resolved (e.g. dissociated single-cell RNA-seq), where spatial
  adjacency views and spatial domain detection do not apply
- The goal is method development on tumor/histology-integrated tasks (e.g. colorectal
  cancer liver metastasis invasion identification) rather than the DLPFC layer-annotation
  benchmark
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MCGAE
  role: 'Primary method: multi-view contrastive graph autoencoder whose unified multimodal
    embedding is clustered and scored by ARI against manual DLPFC layer annotations'
  repo: github.com/yiwen-yang/MCGAE
- name: STAGATE
  role: 'Baseline competitor: graph attention autoencoder run on the same 12 slices
    to produce per-slice ARI for comparison'
- name: BayesSpace
  role: 'Baseline competitor: Bayesian spatial method run on the same 12 slices to
    produce per-slice ARI for comparison'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:d81fd86937d8e0eb23b7507314916ea960dc33d4be02b73f5cd04ea83da3da50
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# ari-benchmark-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the unsupervised spatial domain detection benchmark from the MCGAE paper (Briefings in Bioinformatics, 2024): per-slice Adjusted Rand Index (ARI) computation for MCGAE vs STAGATE and BayesSpace on the 12 LIBD human DLPFC 10x Visium slices, assembled into a 12-row comparison table with median and mean per method.

## When to use

Use when benchmarking spatial transcriptomics domain-detection methods on Visium slices that have manual layer annotations, and you need per-slice ARI values against ground truth plus median/mean summaries to compare methods.

## When NOT to use

- Slices lack manual layer annotations or any ground-truth domain labels, so ARI cannot be computed against a reference
- Data are not spatially resolved (e.g. dissociated single-cell RNA-seq), where spatial adjacency views and spatial domain detection do not apply
- The goal is method development on tumor/histology-integrated tasks (e.g. colorectal cancer liver metastasis invasion identification) rather than the DLPFC layer-annotation benchmark

## Inputs

- 12 LIBD human DLPFC 10x Visium slices (raw counts plus spot coordinates)
- Manual layer annotations (ground-truth spatial domain labels per spot)
- MCGAE source code from github.com/yiwen-yang/MCGAE

## Outputs

- 12-row CSV of per-slice ARI values with columns for MCGAE, STAGATE, and BayesSpace
- Median and mean ARI summary statistics per method (MCGAE median 0.51, mean 0.52)
- Per-slice spatial domain cluster assignments from the MCGAE multimodal embedding

## How to apply

Obtain the 12 LIBD human DLPFC 10x Visium slices with manual layer annotations and the MCGAE code from github.com/yiwen-yang/MCGAE. Preprocess each slice (quality filtering, gene expression normalization, construction of the spatial adjacency view from spot coordinates), then train the MCGAE multi-view contrastive graph autoencoder per slice to produce a unified multimodal embedding; cluster that embedding into spatial domains and compute ARI against the manual layer annotations. Run STAGATE (graph attention autoencoder) and BayesSpace (Bayesian spatial method) on the same 12 slices for a like-for-like comparison. Assemble all per-slice ARI values into a 12-row CSV (one row per slice; columns MCGAE, STAGATE, BayesSpace) and report median and mean per method; expected anchors are MCGAE mean 0.52 / median 0.51, STAGATE approx 0.41/0.44, and slice #151669 MCGAE ARI approx 0.63 vs BayesSpace 0.46, consistent with Supplementary Note 5 / Fig. 2A.

## Related tools

- **MCGAE** (Primary method: multi-view contrastive graph autoencoder whose unified multimodal embedding is clustered and scored by ARI against manual DLPFC layer annotations) — github.com/yiwen-yang/MCGAE
- **STAGATE** (Baseline competitor: graph attention autoencoder run on the same 12 slices to produce per-slice ARI for comparison)
- **BayesSpace** (Baseline competitor: Bayesian spatial method run on the same 12 slices to produce per-slice ARI for comparison)

## Evaluation signals

- The ARI CSV exists with exactly 12 slice rows (one per DLPFC slice) and columns for MCGAE, STAGATE, and BayesSpace
- MCGAE mean ARI falls in [0.45, 0.60], consistent with the reported mean 0.52 and median 0.51
- STAGATE mean/median approximate 0.41/0.44 across the 12 slices
- Slice #151669 MCGAE ARI approximates 0.63 vs BayesSpace 0.46
- The assembled table is consistent with Supplementary Note 5 / Fig. 2A of the MCGAE paper

## Limitations

- Adjacency matrix computation for spatial position information incurs O(n^2) complexity, which may exhaust computational resources with large sample counts
- MCGAE's batch-effect handling via distance between neighboring regions may not suffice for other complex datasets
- The benchmark reflects DLPFC Visium slices with layer annotations; generalization to other tissues or image-free settings must be verified separately

## Evidence

- [abstract] MCGAE demonstrates superior performance in spatial domain detection, data denoising, trajectory inference, and 3D feature extraction, outperforming existing methods: "MCGAE demonstrates superior performance in spatial domain detection, data denoising, trajectory inference, and 3D feature extraction, outperforming"
- [abstract] MCGAE advances the field by creating multi-view representations from gene expression and spatial adjacency matrices.: "MCGAE advances the field by creating multi-view representations from gene expression and spatial adjacency matrices."
- [intro] Non-spatial clustering overlooks spatial relationships, motivating spatial benchmarking against layer annotations.: "non-spatial clustering algorithms often overlook vital spatial relationships"
- [readme] README describes MCGAE as a framework for spatial transcriptomics analysis across platforms.: "MCGAE is a novel deep computational framework designed for comprehensive analysis of spatial transcriptomics data across platforms, integrating morphological images."
