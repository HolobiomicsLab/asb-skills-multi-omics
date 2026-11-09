---
name: count-matrix-log-transformation
description: Use when simulated count matrices from MOSim (mosim) or sc_mosim must
  be transformed before clustering, PCA, or benchmarking of clustering tools, as the
  MOSim methods specify log(x + 1) on count data.
when_to_use_negative:
- Input is already normalized or log-scale data (e.g. log-CPM, TPM, or Seurat-normalized
  expression) rather than raw simulated counts
- The downstream method requires raw integer counts (e.g. count-based models that
  assume the negative binomial distribution on the original scale)
tools:
- name: MOSim
  role: Generates the simulated bulk and single-cell count matrices that the log(x
    + 1) transformation is applied to
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Language in which both MOSim simulation and the log(x + 1) transformation
    of the count matrices are performed
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1101/421834
    title: 'MOSim: bulk and single-cell multi-layer regulatory network simulator'
  source_content_hash: sha256:b9fe92f0de5c19cf085afc399a7c9bb631e91712ce0184381c59afc45892b91e
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# count-matrix-log-transformation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies a log(x + 1) transformation to simulated count matrices (bulk negative-binomial MOSim output or SPARSim-based sc_mosim scRNA-seq output) prior to downstream assessment steps such as K-means/K-medoids clustering, PCA, and clustering-method benchmarking (e.g. scDHA).

## When to use

Use when simulated count matrices from MOSim (mosim) or sc_mosim must be transformed before clustering, PCA, or benchmarking of clustering tools, as the MOSim methods specify log(x + 1) on count data.

## When NOT to use

- Input is already normalized or log-scale data (e.g. log-CPM, TPM, or Seurat-normalized expression) rather than raw simulated counts
- The downstream method requires raw integer counts (e.g. count-based models that assume the negative binomial distribution on the original scale)

## Inputs

- Simulated bulk count matrix from mosim() (negative binomial-based, e.g. RNA-seq/ATAC-seq on STATegra seed data)
- Simulated single-cell scRNA-seq count matrix from sc_mosim() (SPARSim-based, e.g. pbmc.multiome seed data)

## Outputs

- Log-transformed (log(x + 1)) count matrix ready for K-means, K-medoids, PCA, or scDHA clustering
- Downstream assessment metrics files (e.g. mosim_bulk_assessment_metrics.csv, clustering_metrics.csv)

## How to apply

After generating simulated count matrices with MOSim — bulk data from the negative binomial-based mosim() function, or single-cell data from sc_mosim() which leverages SPARSim — apply a log transformation of the form log(x + 1) to the counts. This is the transformation specified in the MOSim methods and is applied in both surfaced workflows: before K-means clustering (k selected by maximising Silhouette's coefficient and minimising intra-cluster variability) and PCA of bulk simulated feature profiles, and before running the VAE-based scDHA clustering on simulated scRNA-seq data. The +1 offset preserves exact zeros in the count matrix while stabilising the variance of highly expressed features. The transformed matrix is then the input for cluster-number selection, PCA validation, and correlation-based assessment (Pearson's for bulk, Kendall's Tb for single-cell).

## Related tools

- **MOSim** (Generates the simulated bulk and single-cell count matrices that the log(x + 1) transformation is applied to) — github.com/ConesaLab/MOSim
- **R** (Language in which both MOSim simulation and the log(x + 1) transformation of the count matrices are performed)

## Examples

```
simulated_counts <- sc_mosim(...); log_counts <- log(simulated_counts + 1)
```

## Evaluation signals

- All transformed values are >= 0 and exact zeros in the original count matrix remain zeros after log(x + 1)
- Bulk workflow: after transformation, K-means (k selected by Silhouette's coefficient) assigns <0.5% of simulated profiles to incorrect clusters and PCA aligns clustering with the simulation settings
- Single-cell workflow: scDHA clustering on the transformed counts achieves ARI 0.949 against ground-truth simulated cell-type labels
- The transformation formula matches the methods exactly (log(x + 1), i.e. natural log with a +1 pseudocount), not log2 or base-10 variants
- Regulatory-signal validation downstream succeeds: ~99.4% of modelled activator/repressor interactions show Pearson correlation >0.9 while 0% of no-effect pairs do

## Limitations

- MOSim simulation operates at the count matrix level rather than simulating read data, so the transformation applies to matrices only, not raw reads
- The log(x + 1) transform is specified as a preprocessing step for the paper's assessment workflows (K-means, K-medoids, PCA, scDHA); other downstream tools may require different normalizations

## Evidence
