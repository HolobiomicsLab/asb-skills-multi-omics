---
name: baseline-method-comparison-benchmarking
description: Use when a new spatial transcriptomics clustering method needs head-to-head
  evaluation against published baselines (e.g. STAGATE, BayesSpace) on a common dataset
  with ground-truth annotations such as manual DLPFC layer labels.
when_to_use_negative:
- When the input has no ground-truth spatial-domain annotations, since ARI against
  manual layer labels cannot be computed for fair comparison
- When evaluating MCGAE's tumor-invasion application (colorectal cancer liver metastases
  histology+expression integration) rather than benchmarking against STAGATE/BayesSpace
  on DLPFC
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MCGAE
  role: 'Method under evaluation: multi-view contrastive graph autoencoder whose per-slice
    ARI on the 12 DLPFC slices is compared to baselines'
  repo: github.com/yiwen-yang/MCGAE
- name: STAGATE
  role: 'Baseline method: graph attention autoencoder run on the same 12 DLPFC slices
    for per-slice ARI comparison'
- name: BayesSpace
  role: 'Baseline method: Bayesian spatial method optimizing relationships between
    adjacent points, run on the same 12 DLPFC slices'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:25c9466d859f31c895dc61a9d008b291dafd4189fde89b27fec91b8244d5ef21
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/yiwen-yang/MCGAE
  tools:
  - BayesSpace
  - MCGAE
  - STAGATE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# baseline-method-comparison-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a proposed spatial-domain detection method against established baselines by running each method on the same inputs and comparing per-sample performance metrics. In this context: run MCGAE, STAGATE, and BayesSpace side-by-side on the 12 LIBD human DLPFC 10x Visium slices and report per-slice Adjusted Rand Index (ARI) against manual layer annotations.

## When to use

Use when a new spatial transcriptomics clustering method needs head-to-head evaluation against published baselines (e.g. STAGATE, BayesSpace) on a common dataset with ground-truth annotations such as manual DLPFC layer labels.

## When NOT to use

- When the input has no ground-truth spatial-domain annotations, since ARI against manual layer labels cannot be computed for fair comparison
- When evaluating MCGAE's tumor-invasion application (colorectal cancer liver metastases histology+expression integration) rather than benchmarking against STAGATE/BayesSpace on DLPFC

## Inputs

- 12 LIBD human dorsolateral prefrontal cortex (DLPFC) 10x Visium slices with spot coordinates and manual layer annotations
- MCGAE source code (github.com/yiwen-yang/MCGAE)
- STAGATE and BayesSpace baseline implementations
- Processed spatial adjacency view built from spot coordinates per slice

## Outputs

- 12-row CSV table (one row per slice; columns MCGAE, STAGATE, BayesSpace) of per-slice ARI values
- Per-method median and mean ARI summary statistics

## How to apply

Obtain each method's code (MCGAE from github.com/yiwen-yang/MCGAE) and a common benchmark dataset — here the 12 LIBD human DLPFC 10x Visium slices with manual layer annotations. Preprocess each slice identically for all methods (quality filtering, gene-expression normalization, and construction of the spatial adjacency view from spot coordinates) so differences reflect the methods, not the pipeline. Train the multi-view contrastive graph autoencoder per slice to produce a unified multimodal embedding, cluster the embedding into spatial domains, and compute Adjusted Rand Index against the manual layer annotations; run STAGATE (graph attention autoencoder) and BayesSpace (Bayesian optimization of adjacent-point relationships) on the same 12 slices and compute their per-slice ARI. Assemble per-slice ARI values into a 12-row CSV (one row per slice, columns MCGAE/STAGATE/BayesSpace) and report median and mean per method.

## Related tools

- **MCGAE** (Method under evaluation: multi-view contrastive graph autoencoder whose per-slice ARI on the 12 DLPFC slices is compared to baselines) — github.com/yiwen-yang/MCGAE
- **STAGATE** (Baseline method: graph attention autoencoder run on the same 12 DLPFC slices for per-slice ARI comparison)
- **BayesSpace** (Baseline method: Bayesian spatial method optimizing relationships between adjacent points, run on the same 12 DLPFC slices)

## Examples

```
from sklearn.metrics import adjusted_rand_score; ari = adjusted_rand_score(manual_layer_annotations, predicted_domains)
```

## Evaluation signals

- The ARI CSV exists with exactly 12 slice rows (one row per DLPFC slice) and columns for MCGAE, STAGATE, BayesSpace
- MCGAE mean ARI falls in [0.45, 0.60], consistent with the reported mean of 0.52 and median of 0.51
- STAGATE mean/median ARI approximate 0.41/0.44
- Slice #151669 MCGAE ARI approximates 0.63 versus BayesSpace 0.46
- The assembled table is consistent with Supplementary Note 5 / Fig. 2A of the paper

## Limitations

- Adjacency matrix computation from spatial position information has O(n^2) complexity, which may exhaust computational resources with large sample counts and limits scalability of the benchmark protocol
- The batch-effect handling strategy (distance between neighboring regions) may not suffice for other complex datasets beyond the tested slices
- The 12-slice DLPFC benchmark does not by itself establish performance on datasets lacking image information at scale, although MCGAE is reported to function without image information

## Evidence

- [abstract] MCGAE demonstrates superior performance ... outperforming: "MCGAE demonstrates superior performance in spatial domain detection, data denoising, trajectory inference, and 3D feature extraction, outperforming"
- [intro] BayesSpace baseline description: "BayesSpace [15] applies Bayesian methods to optimize relationships between adjacent points"
- [intro] STAGATE baseline description: "STAGATE [16] incorporate graph-based and attention mechanisms, respectively, to enhance spatial domain identification."
