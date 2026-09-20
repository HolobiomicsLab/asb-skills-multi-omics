---
name: leiden-clustering-resolution-sweep
description: Use when you have an integrated embedding from a multi-omics method (SSGATE,
  MOFA+, Seurat v5, totalVI, scMM) on .h5ad data and need multiple Leiden clusterings
  at resolutions 0.1–1.0 to compute Purity, HS, ARI, and NMI against ground-truth
  labels.
when_to_use_negative:
- Data is spatially resolved multi-omics where spatial coordinates in anndata.obsm['spatial']
  should drive Radius-model graph construction rather than expression-profile KNN
  graphs
- The goal is a single fixed clustering rather than a method benchmark requiring resolution-independent
  comparison across methods
- The embedding comes from raw unnormalized expression rather than the SSGATE-integrated
  30-dim representation, since the paper's benchmark protocol operates on integrated
  embeddings
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCANPY
  role: Runs the Leiden clustering of integrated embeddings and supplies the preprocessing
    (log1p scaling, HVG selection, CLR transformation) on the h5ad inputs
- name: SSGATE
  role: Produces the dual-path graph attention auto-encoder integrated embedding and
    the neighborhood graph (Cal_Nbrs_Net + prune_net) that Leiden clusters
  repo: github.com/Lv-BioInfo/SSGATE
- name: Leiden algorithm
  role: Community-detection algorithm applied at resolutions 0.1 through 1.0 to partition
    the integrated embedding into clusters
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:cb0d82e917eb8d93e2b15dcb0c648b8dadb40339ec71b7479aa388cb00dbf3b3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Leiden
  - Leiden algorithm
  - MOFA
  - SCANPY
  - SSGATE
  - ScMM
  - Seurat v5
  - TotalVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# leiden-clustering-resolution-sweep

## Summary

Cluster integrated single-cell multi-omics embeddings (e.g. SSGATE's 30-dim joint representation) with the Leiden algorithm in SCANPY, sweeping the 'resolution' parameter over 0.1–1.0 to generate 10 independent clustering replicates per method/dataset for robust metric aggregation.

## When to use

Use when you have an integrated embedding from a multi-omics method (SSGATE, MOFA+, Seurat v5, totalVI, scMM) on .h5ad data and need multiple Leiden clusterings at resolutions 0.1–1.0 to compute Purity, HS, ARI, and NMI against ground-truth labels.

## When NOT to use

- Data is spatially resolved multi-omics where spatial coordinates in anndata.obsm['spatial'] should drive Radius-model graph construction rather than expression-profile KNN graphs
- The goal is a single fixed clustering rather than a method benchmark requiring resolution-independent comparison across methods
- The embedding comes from raw unnormalized expression rather than the SSGATE-integrated 30-dim representation, since the paper's benchmark protocol operates on integrated embeddings

## Inputs

- Integrated 30-dim embedding (out_dims=30) stored in AnnData, produced by SSGATE or a baseline method on BMNC or SLN111_D1 h5ad data
- Ground-truth cell-type labels for computing Purity, HS, ARI, NMI
- Preprocessed per-omics AnnData objects (log1p + HVG RNA, CLR protein) with SSGATE neighbor network in adata.uns['nbrs_net']

## Outputs

- Cluster label assignments per resolution (10 replicates per method/dataset)
- Per-resolution metric table of Purity, Homogeneity Score, ARI, NMI aggregated into benchmark_metrics.csv or noise_robustness_results.csv

## How to apply

After training SSGATE (or a baseline) on preprocessed BMNC/SLN111_D1 h5ad data (log1p-scaled RNA with HVG selection, CLR-transformed protein), take the 30-dimensional integrated embedding and run Leiden clustering in SCANPY at resolutions 0.1, 0.2, ..., 1.0, yielding 10 independent experiment replicates per method per dataset. Each replicate's cluster labels are compared to ground-truth cell labels to compute Purity (P), Homogeneity Score (HS), Adjusted Rand Index (ARI), and Normalized Mutual Information (NMI); the sweep avoids cherry-picking a single resolution and enables average rankings and Robust Rank Aggregation p-values across methods. The neighborhood graph feeding Leiden should come from SSGATE's pipeline: Cal_Nbrs_Net (KNN model, k_cutoff=15 is optimal at sigma=0.1) followed by prune_net, so edges only connect cells within the same cell population. Persist all per-resolution metrics into benchmark_metrics.csv or noise_robustness_results.csv for downstream ranking.

## Related tools

- **SCANPY** (Runs the Leiden clustering of integrated embeddings and supplies the preprocessing (log1p scaling, HVG selection, CLR transformation) on the h5ad inputs)
- **SSGATE** (Produces the dual-path graph attention auto-encoder integrated embedding and the neighborhood graph (Cal_Nbrs_Net + prune_net) that Leiden clusters) — github.com/Lv-BioInfo/SSGATE
- **Leiden algorithm** (Community-detection algorithm applied at resolutions 0.1 through 1.0 to partition the integrated embedding into clusters)

## Examples

```
for res in [0.1,0.2,0.3,0.4,0.5,0.6,0.7,0.8,0.9,1.0]: sc.tl.leiden(adata, resolution=res, key_added=f'leiden_{res}')
```

## Evaluation signals

- Exactly 10 replicates (resolutions 0.1, 0.2, ..., 1.0) are produced per method per dataset
- On BMNC with SSGATE defaults (15 neighbors, sigma=0.1, 300 epochs, out_dims=30, lr=0.001), SSGATE ranks first in ARI and NMI and second in Purity and HS, giving average ranking 1.5
- On SLN111_D1 the protocol reproduces an average ranking of 2.25 for SSGATE (tied with totalVI) and the smallest RRA p-value among the five methods
- All four metrics (P, HS, ARI, NMI) are computed for every method/dataset/resolution combination and written to benchmark_metrics.csv
- With the neighbor count swept at fixed sigma=0.1, 15 neighbors yields optimal values in all four metrics on both datasets (Table 2 invariant)

## Limitations

- Results are resolution-dependent; conclusions must be drawn from the aggregate of the 0.1–1.0 sweep (average rankings/RRA p-values) rather than any single resolution
- Clustering quality depends on the upstream neighborhood graph: hyperparameters such as k_cutoff=15, sigma=0.1, and embedding dimension 30 were set 'based on experience' as defaults rather than systematically derived, and no software versions for compared methods are reported
- The paper's validation targets specific datasets (BMNC, SLN111_D1); benchmark behavior on other tissues or sequencing technologies is not established by this protocol

## Evidence

- [methods] Cluster every integrated embedding with the Leiden algorithm, sweeping the 'resolution' parameter over 0.1, 0.2, …, 1.0 to yield 10 independent experiment replicates per method per dataset.: [evidence span withheld — anchor status: not_found]
- [readme] Leiden on pruned neighbor network stored in uns: "ensuring that each edge only connects cells within the same cell population. After pruning, the updated adjacency network will be stored in the uns attribute of the AnnData object."
- [abstract] Neighborhood graphs from expression profiles or spatial coordinates: "It can construct the neighborhood graphs based on single-cell expression profiles or spatial coordinates"
