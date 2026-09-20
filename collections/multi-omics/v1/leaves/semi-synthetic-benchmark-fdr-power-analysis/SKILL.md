---
name: semi-synthetic-benchmark-fdr-power-analysis
description: Use when evaluating whether a variable-selection or conditional-dependency
  method controls FDR while gaining power on semi-synthetic single-cell data, especially
  when comparing hierarchical (group-level) vs. gene-level test resolutions.
when_to_use_negative:
- Input AnnData contains normalized expression rather than raw counts — VIVS explicitly
  requires raw counts in X.
- The dataset has many thousands of genes without prior filtering; the README warns
  'VIVS may not scale to many thousands of genes' and recommends selecting top genes
  first (e.g. n_top_genes=2000).
- The goal is marginal (correlation-based) association testing rather than calibrated
  conditional independence testing with FDR control.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: VIVS
  role: Trains the conditional-independence model and computes hierarchical CRT p-values
    via get_hier_importance for multi-resolution testing
  repo: github.com/YosefLab/VIVS
- name: scVI
  role: Deep generative model used as the generative component modeling normalized
    gene expression in the semi-synthetic setup
- name: scanpy
  role: Provides the AnnData object format for importing raw-count single-cell data
- name: Jax
  role: Backend for the fast, scalable VIVS implementation (parallel computing and
    just-in-time compilation); install with GPU support
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:c4683abb166974941c834046e9a586c9f301ede508f9eeea02c53855075d0321
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jax
  - Python
  - VI-VS
  - VIVS
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# semi-synthetic-benchmark-fdr-power-analysis

## Summary

Benchmark procedure for VI-VS multi-resolution conditional independence testing on semi-synthetic single-cell data: genes with known planted dependencies define ground truth, and detections are scored by FDR and power across hierarchical resolutions (e.g., 200 and 300 gene clusters plus gene level) with a group counted as a true positive if it contains at least one gene-level true positive.

## When to use

Use when evaluating whether a variable-selection or conditional-dependency method controls FDR while gaining power on semi-synthetic single-cell data, especially when comparing hierarchical (group-level) vs. gene-level test resolutions.

## When NOT to use

- Input AnnData contains normalized expression rather than raw counts — VIVS explicitly requires raw counts in X.
- The dataset has many thousands of genes without prior filtering; the README warns 'VIVS may not scale to many thousands of genes' and recommends selecting top genes first (e.g. n_top_genes=2000).
- The goal is marginal (correlation-based) association testing rather than calibrated conditional independence testing with FDR control.

## Inputs

- Semi-synthetic PBMC single-cell dataset as a scanpy AnnData object with raw counts in X and response variable(s) in obsm
- Gene-level ground truth labels for planted conditional dependencies used to define true positives
- List of cluster resolutions for hierarchical testing, e.g. n_clusters_list=[200, 300] plus gene level
- Number of Monte Carlo samples K for CRT p-value computation

## Outputs

- Per-resolution FDR/power table (e.g. fdr_power_multiresolution.csv)
- List of detected gene groups/clusters per resolution, including clusters detected only at coarser resolutions
- Benjamini-Hochberg-corrected CRT p-values per feature/group at each resolution

## How to apply

Load the semi-synthetic PBMC dataset as a scanpy AnnData object containing raw counts, filter to the top 2000 genes with the vivs select_genes function, then initialize and train a VIVS model (generative model: scVI latent variable model; predictive model with xy_linear=False and xy_model_kwargs={'n_hidden': 8}) via model.train_all(). Cluster genes with fast agglomerative hierarchical clustering on the gene-by-gene empirical correlation matrix computed from scVI-normalized expression. Call model.get_hier_importance with n_clusters_list=[200, 300] plus the individual gene-level resolution, computing conditional randomization test (CRT) p-values with K Monte Carlo samples per feature/group and correcting them with the Benjamini-Hochberg procedure. Score each resolution by computing FDR and power, treating a selected group as a true positive if it contains at least one gene-level true positive, then write the per-resolution FDR/power table (e.g., fdr_power_multiresolution.csv) and record detected gene groups such as clusters found only at coarser resolution.

## Related tools

- **VIVS** (Trains the conditional-independence model and computes hierarchical CRT p-values via get_hier_importance for multi-resolution testing) — github.com/YosefLab/VIVS
- **scVI** (Deep generative model used as the generative component modeling normalized gene expression in the semi-synthetic setup)
- **scanpy** (Provides the AnnData object format for importing raw-count single-cell data)
- **Jax** (Backend for the fast, scalable VIVS implementation (parallel computing and just-in-time compilation); install with GPU support)

## Examples

```
from vivs import select_genes, VIVS; adata = select_genes(adata, n_top_genes=2000); model = VIVS(adata, feature_obsm_key='my_obsm_key', xy_linear=False, xy_model_kwargs={'n_hidden': 8}); model.train_all(); res = model.get_hier_importance(n_clusters_list=[200, 300])
```

## Evaluation signals

- Empirical FDR at each resolution stays at or below the nominal target level (calibrated FDR control).
- Power (proportion of gene-level true positives recovered) increases or is maintained at coarser resolutions relative to gene-level testing.
- Detections are consistent across resolutions; coarser-resolution detections should be traceable to gene-level true positives (a selected group counts as a true positive only if it contains at least one gene-level true positive).
- The output table fdr_power_multiresolution.csv contains one FDR and one power value per tested resolution (gene level, 200 clusters, 300 clusters).
- Corrected p-values are produced with the Benjamini-Hochberg procedure applied to CRT p-values at each resolution.

## Limitations

- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries.
- VIVS may not scale to many thousands of genes, so gene filtering (e.g. top 2000 via select_genes) is recommended beforehand.
- VIVS only supports assays where X corresponds to gene expression counts, and the AnnData must contain raw counts, not normalized data.
- The semi-synthetic ground-truth definition (group-level true positive = contains at least one gene-level true positive) is a simulation-specific scoring convention, not a general-purpose validation criterion.

## Evidence

- [abstract] It uses deep generative models to identify conditionally dependent features, with false discovery rate control.: "It uses deep generative models to identify conditionally dependent features, with false discovery rate control."
- [intro] We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate.: "We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate."
- [methods] adata = select_genes(adata, n_top_genes=2000): "adata = select_genes(adata, n_top_genes=2000)"
- [methods] res = model.get_hier_importance(n_clusters_list=[100, 200]): "res = model.get_hier_importance(n_clusters_list=[100, 200])"
- [discussion] To address this risk, we proposed a multiresolution testing procedure.: "To address this risk, we proposed a multiresolution testing procedure."
- [readme] Right now, VIVS only supports assays where $X$ corresponds to gene expression counts. In particular, make sure than the anndata contains raw counts and not normalized data.: "make sure than the anndata contains raw counts and not normalized data"
- [readme] VIVS may not scale to many thousands of genes. In such a case, it is recommended to filter the genes before running VIVS: "VIVS may not scale to many thousands of genes. In such a case, it is recommended to filter the genes before running VIVS"
