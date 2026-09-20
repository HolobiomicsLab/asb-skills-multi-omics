---
name: hierarchical-gene-clustering-on-correlation-matrix
description: Use when running VI-VS/VIVS multi-resolution hypothesis testing on single-cell
  count data and you need gene groups (clusters) as testable features in addition
  to individual genes.
when_to_use_negative:
- The AnnData contains normalized rather than raw counts — VIVS requires raw counts
  in X
- The dataset has many thousands of genes without prior filtering — VIVS may not scale;
  filter with select_genes(n_top_genes=2000) first
- The goal is marginal co-expression network inference (e.g., plain GENIE3 on correlations)
  rather than group conditional-independence testing with FDR control
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: VIVS
  role: Provides the model whose get_hier_importance consumes cluster resolutions
    and computes hierarchical feature importance (CRT p-values)
  repo: github.com/YosefLab/VIVS
- name: scVI
  role: Generative latent variable model whose normalized gene expression supplies
    the correlation matrix for clustering
- name: Jax
  role: Backend used by VIVS for parallel computing and just-in-time compilation to
    speed up inference
- name: scanpy
  role: AnnData container format for the raw-count input data
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:4e20bdfeae10923f911fa0b4b55ae1760ee62ddc558f2fe3bce54a22199434b1
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Hierarchical gene clustering on correlation matrix

## Summary

Clusters genes using fast agglomerative hierarchical clustering applied to the gene-by-gene empirical correlation matrix computed from scVI-normalized expression, enabling VI-VS multi-resolution testing at multiple granularities (e.g., 200, 300 clusters, plus individual genes).

## When to use

Use when running VI-VS/VIVS multi-resolution hypothesis testing on single-cell count data and you need gene groups (clusters) as testable features in addition to individual genes.

## When NOT to use

- The AnnData contains normalized rather than raw counts — VIVS requires raw counts in X
- The dataset has many thousands of genes without prior filtering — VIVS may not scale; filter with select_genes(n_top_genes=2000) first
- The goal is marginal co-expression network inference (e.g., plain GENIE3 on correlations) rather than group conditional-independence testing with FDR control

## Inputs

- scanpy AnnData object with raw gene expression counts (X) and response(s) Y stored in obsm
- Gene-filtered AnnData (top 2000 genes via vivs select_genes)
- Trained VIVS model
- Gene-by-gene empirical correlation matrix computed from scVI-normalized expression

## Outputs

- Hierarchical clusterings of genes at each requested resolution (n_clusters_list values)
- Hierarchical importance results with CRT p-values per gene/group (Benjamini-Hochberg corrected), e.g., from get_hier_importance
- Per-resolution FDR/power table (e.g., fdr_power_multiresolution.csv) with detected gene groups

## How to apply

After training the VIVS model (generative model: scVI latent variable model; predictive model with xy_linear=False and xy_model_kwargs={'n_hidden': 8}) on raw-count AnnData filtered to the top 2000 genes via select_genes, compute the gene-by-gene empirical correlation matrix from the scVI-normalized expression. Apply fast agglomerative hierarchical clustering on this correlation matrix to partition genes into groups. Pass the desired partition sizes through model.get_hier_importance via n_clusters_list (e.g., [200, 300] plus the individual gene-level resolution); CRT p-values are then computed per feature/group with K Monte Carlo samples and corrected with Benjamini-Hochberg. Coarser resolutions test group conditional independence, so a selected group is scored as a true positive if it contains at least one gene-level true positive.

## Related tools

- **VIVS** (Provides the model whose get_hier_importance consumes cluster resolutions and computes hierarchical feature importance (CRT p-values)) — github.com/YosefLab/VIVS
- **scVI** (Generative latent variable model whose normalized gene expression supplies the correlation matrix for clustering)
- **Jax** (Backend used by VIVS for parallel computing and just-in-time compilation to speed up inference)
- **scanpy** (AnnData container format for the raw-count input data)

## Examples

```
from vivs import VIVS; model = VIVS(adata, feature_obsm_key='my_obsm_key', xy_linear=False, xy_model_kwargs={'n_hidden': 8}); model.train_all(); res = model.get_hier_importance(n_clusters_list=[200, 300])
```

## Evaluation signals

- Clusterings are derivable at every resolution listed in n_clusters_list (e.g., 200 and 300 clusters plus individual gene level)
- Per-resolution FDR remains calibrated (nominal FDR maintained) while power increases at coarser resolutions
- Detections are consistent across resolutions; groups selected at coarse resolution contain gene-level true positives (TP if a group contains at least one gene-level TP)
- Coarser-resolution testing surfaces relevant genes not detected at the gene level (e.g., clusters 5 and 6 in the semi-synthetic PBMC experiment)
- FDR/power table is written to fdr_power_multiresolution.csv alongside the detected gene-group list

## Limitations

- VIVS may not scale to many thousands of genes; gene filtering (e.g., top 2000) is recommended before clustering and testing
- In the general case, VI-VS discoveries have no guarantees to be functional; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries
- Multiresolution testing is proposed to address the risk of missing signal, but group-level detections do not pinpoint individual causal genes

## Evidence

- [methods] Multi-Resolution Hypothesis Testing: "Multi-Resolution Hypothesis Testing"
- [discussion] To address this risk, we proposed a multiresolution testing procedure.: "To address this risk, we proposed a multiresolution testing procedure."
- [readme] n_clusters_list definition: "Here, `n_clusters_list` is a list of the number of clusters to consider for the hierarchical clustering of the features."
- [methods] hierarchical importance call: "res = model.get_hier_importance(n_clusters_list=[100, 200])"
