---
name: spearman-distance-kmedoids-clustering
description: Use when validating that a single-cell multi-omics simulation (e.g. sc_mosim
  scRNA-seq/scATAC-seq output) reproduced the intended cell-type-specific expression
  profiles, by clustering gene profiles and measuring misassignment rates.
when_to_use_negative:
- 'Bulk RNA-seq simulation output: the article applies K-means clustering (not K-medoids
  with Spearman distance) to bulk feature profiles to assess temporal expression patterns'
- Input is raw per-cell counts without first aggregating to the average count per
  cell type and applying the log(x + 1) transformation, since the distance metric
  is defined on aggregated, transformed profiles
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: MOSim
  role: Source of the simulated single-cell count matrix (via sc_mosim) whose gene
    profiles are clustered and validated
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Implementation environment (R Bioconductor package); clustering, Silhouette,
    and PCA performed in R
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/421834
    title: 'MOSim: bulk and single-cell multi-layer regulatory network simulator'
  source_content_hash: sha256:3110b8339adb1071d936bf02ddb856f0aecb6316d045a6fd35230fe0a72d9218
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ConesaLab/MOSim
  repo_url: https://github.com/ConesaLab/MOSim
  tools:
  - MOSim
  - R
  - SPARSim
  - Seurat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# spearman-distance-kmedoids-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters simulated or real gene expression profiles across cell types using Spearman distance (1 - Spearman's correlation) with partition-around-medoids (K-medoids) clustering, after aggregating single-cell counts per cell type and log(x + 1) transformation, to verify that simulated co-expression structure matches simulation settings.

## When to use

Use when validating that a single-cell multi-omics simulation (e.g. sc_mosim scRNA-seq/scATAC-seq output) reproduced the intended cell-type-specific expression profiles, by clustering gene profiles and measuring misassignment rates.

## When NOT to use

- Bulk RNA-seq simulation output: the article applies K-means clustering (not K-medoids with Spearman distance) to bulk feature profiles to assess temporal expression patterns
- Input is raw per-cell counts without first aggregating to the average count per cell type and applying the log(x + 1) transformation, since the distance metric is defined on aggregated, transformed profiles

## Inputs

- Simulated single-cell count matrix from sc_mosim (scRNA-seq counts, SPARSim-based)
- Per-cell-type grouping/cell type labels of the simulated cells
- Simulation design parameters (number of cell types, e.g. 6) used to set or check K

## Outputs

- K-medoids cluster assignment for each gene expression profile across cell types
- Silhouette coefficient and intra-cluster variability values used to select K
- PCA plot confirming cluster alignment with simulation settings
- Fraction of gene expression profiles in incorrect clusters (benchmark: 3.3%)
- sc_mosim_assessment_metrics.csv

## How to apply

First aggregate the simulated single-cell count matrix to obtain the average count per cell type, then apply a log transformation (log(x + 1)). Compute the Spearman distance, defined as 1 - Spearman's correlation, between gene expression profiles across cell types, and cluster these profiles using partition around medoids (K-medoids). Choose the optimal number of clusters by combining maximisation of Silhouette's coefficient with minimisation of intra-cluster variability (the paper used K=10 for 6 simulated cell types), and use PCA to confirm that clustering aligns with the simulation settings. Judge correctness by the fraction of gene expression profiles assigned to incorrect clusters; the article reports 3.3% of profiles in incorrect clusters as the expected benchmark.

## Related tools

- **MOSim** (Source of the simulated single-cell count matrix (via sc_mosim) whose gene profiles are clustered and validated) — github.com/ConesaLab/MOSim
- **R** (Implementation environment (R Bioconductor package); clustering, Silhouette, and PCA performed in R)

## Examples

```
R: cluster_stats <- pam(daisy(log(t(aggregate_counts(sim_scRNA) + 1)), diss=FALSE), k=10, diss=FALSE)  # after computing 1 - cor(avg_counts, method='spearman')
```

## Evaluation signals

- Fraction of gene expression profiles in incorrect clusters matches the reported benchmark of ~3.3%
- Silhouette's coefficient is maximised and intra-cluster variability minimised at the chosen K (K=10 for 6 simulated cell types)
- PCA of the log(x + 1)-transformed per-cell-type averages shows clusters aligned with the simulation settings (cell types separate as designed)
- Distance matrix uses Spearman distance, i.e. 1 - Spearman's correlation between aggregated profiles, verified against the clustering call

## Limitations

- Validation is meaningful only against known simulation ground truth; with real data the correct cluster assignment of each profile is unknown
- The optimal number of clusters must be re-derived per dataset (Silhouette + intra-cluster variability); K=10 is specific to the pbmc.multiome-based 6-cell-type simulation
- sc_mosim single-cell simulation is restricted to scRNA-seq and scATAC-seq only

## Evidence

- [methods] Clustering method: [evidence span withheld — anchor status: not_found]
- [methods] Cluster number selection: [evidence span withheld — anchor status: not_found]
