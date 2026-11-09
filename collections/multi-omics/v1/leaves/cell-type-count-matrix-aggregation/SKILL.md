---
name: cell-type-count-matrix-aggregation
description: Use when a single-cell count matrix (e.g., scRNA-seq from MOSim sc_mosim)
  must be reduced to per-cell-type average profiles before clustering, PCA, or cross-omics
  correlation validation.
when_to_use_negative:
- Input is already a bulk or cell-type-aggregated feature table rather than a cell-level
  single-cell count matrix
- The analysis requires cell-level resolution (e.g., single-cell clustering evaluation)
  rather than collapsed per-cell-type profiles
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: MOSim
  role: sc_mosim generates the simulated scRNA-seq count matrix (via SPARSim) that
    is then aggregated per cell type
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Implementation environment for the aggregation, log(x + 1) transformation,
    and downstream K-medoids clustering/PCA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/421834
    title: 'MOSim: bulk and single-cell multi-layer regulatory network simulator'
  source_content_hash: sha256:3e27f849915b6583800436d51ea45f4eda05cbfc71ff09d0e575764c64d96b10
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ConesaLab/MOSim
  tools:
  - MOSim
  - R
  - SPARSim
  - Seurat
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Cell-type count matrix aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates a simulated (or real) single-cell count matrix to the average count per cell type and applies a log(x + 1) transformation, producing cell-type-level expression profiles suitable for downstream profile clustering (e.g., K-medoids with Spearman distance) as used to validate sc_mosim simulations.

## When to use

Use when a single-cell count matrix (e.g., scRNA-seq from MOSim sc_mosim) must be reduced to per-cell-type average profiles before clustering, PCA, or cross-omics correlation validation.

## When NOT to use

- Input is already a bulk or cell-type-aggregated feature table rather than a cell-level single-cell count matrix
- The analysis requires cell-level resolution (e.g., single-cell clustering evaluation) rather than collapsed per-cell-type profiles

## Inputs

- Simulated single-cell scRNA-seq count matrix (cells x genes) from MOSim sc_mosim / SPARSim
- Cell type labels for each simulated cell

## Outputs

- Cell-type-level average count matrix (cell types x genes)
- Log-transformed (log(x + 1)) cell-type expression profiles for clustering/PCA

## How to apply

Run the single-cell simulation (MOSim sc_mosim, SPARSim-based, e.g., 6 cell types, 2 experimental groups, 3 biological replicates) to obtain the scRNA-seq count matrix. Aggregate the simulated count matrix to obtain the average count per cell type, collapsing cell-level counts into one profile per cell type. Apply a log(x + 1) transformation to the aggregated data to stabilize variance, as specified in the MOSim methods. The resulting cell-type profiles are then inputs for clustering gene expression profiles across cell types using Spearman distance (1 - Spearman's correlation) with partition around medoids (K-medoids, K=10 in the validation), and for PCA confirmation that clustering aligns with simulation settings.

## Related tools

- **MOSim** (sc_mosim generates the simulated scRNA-seq count matrix (via SPARSim) that is then aggregated per cell type) — github.com/ConesaLab/MOSim
- **R** (Implementation environment for the aggregation, log(x + 1) transformation, and downstream K-medoids clustering/PCA)

## Examples

```
agg <- t(apply(counts, 1, function(x) tapply(x, cell_types, mean))); log_agg <- log1p(agg)
```

## Evaluation signals

- Aggregated matrix has exactly one profile per simulated cell type (e.g., 6 rows for the pbmc.multiome-based 6-cell-type simulation)
- All values are non-negative and strictly follow the log(x + 1) transformation (min value 0 after transform of zero counts)
- K-medoids clustering (K=10, Spearman distance) of the aggregated profiles misassigns only ~3.3% of profiles to incorrect clusters
- PCA on the aggregated log-transformed profiles confirms clustering alignment with the simulation settings (cell types separated as designed)

## Limitations

- MOSim operates at the count matrix level rather than simulating read data, so aggregation works on counts only
- Single-cell simulation in MOSim is restricted to scRNA-seq and scATAC-seq, so this aggregation applies to those modalities only
- Aggregation collapses within-cell-type variability; it is intended for profile-level validation, not cell-level heterogeneity analysis

## Evidence

- [methods] Aggregation of simulated single-cell count matrix to average count per cell type: "the simulated count matrix was aggregated to obtain the average count per cell type"
- [methods] Log transformation of aggregated data: "a log transformation (log(x + 1))[42] was applied to the data"
- [methods] Downstream clustering of aggregated profiles: [evidence span withheld — anchor status: not_found]
- [discussion] Single-cell simulation basis: "while for single-cell data, it leverages the well-established simulator SPARSim[11]"
