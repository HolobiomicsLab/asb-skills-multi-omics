---
name: pca-clustering-alignment-verification
description: Use when validating that simulated bulk RNA-seq/ATAC-seq count matrices
  reproduce the intended temporal expression patterns and experimental design before
  downstream GRN or correlation assessments.
when_to_use_negative:
- 'Simulated single-cell data: MOSim''s single-cell assessment instead uses Spearman
  distance with K-medoids clustering and Kendall''s Tb correlation, not bulk K-means/PCA'
- 'Raw untransformed count matrices: the log(x + 1) transformation must precede clustering,
  so this skill does not apply directly to untransformed counts'
- Datasets where no experimental design (time points/groups) was imposed — there is
  no alignment target to verify against
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MOSim
  role: Generates the simulated bulk multi-omics count matrices and design whose clustering-PCA
    alignment is verified
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Implementation environment for K-means clustering, PCA, and Silhouette-based
    cluster-number selection
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/421834
    title: 'MOSim: bulk and single-cell multi-layer regulatory network simulator'
  source_content_hash: sha256:5f4db3cf5cdf8f4b5fd26f9395caa92905d8d176837f8c7c65893e9473a27027
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# PCA-clustering alignment verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A validation step for simulated (or otherwise structured) multi-omics expression data in which log(x+1)-transformed feature profiles are K-means clustered and projected by PCA to confirm that cluster membership matches the experimental design (time points, groups) used to generate the data.

## When to use

Use when validating that simulated bulk RNA-seq/ATAC-seq count matrices reproduce the intended temporal expression patterns and experimental design before downstream GRN or correlation assessments.

## When NOT to use

- Simulated single-cell data: MOSim's single-cell assessment instead uses Spearman distance with K-medoids clustering and Kendall's Tb correlation, not bulk K-means/PCA
- Raw untransformed count matrices: the log(x + 1) transformation must precede clustering, so this skill does not apply directly to untransformed counts
- Datasets where no experimental design (time points/groups) was imposed — there is no alignment target to verify against

## Inputs

- Simulated bulk RNA-seq and ATAC-seq count matrices from mosim() (negative binomial-based)
- Simulation design specification (5 time points, 2 experimental groups, 3 biological replicates, 15% DEGs)
- Regulator-gene association matrix (for the complementary correlation assessment)

## Outputs

- Cluster assignment table for simulated gene feature profiles (k=7)
- PCA score plot showing clustering alignment with time points and experimental groups
- Fraction of profiles assigned to incorrect clusters, compiled into mosim_bulk_assessment_metrics.csv

## How to apply

Apply a log(x + 1) transformation to the simulated count matrix produced by mosim() (e.g., STATegra-seeded bulk simulation with 5 time points, 2 experimental groups, 3 replicates, 15% DEGs). Run K-means clustering (k=7 for the MOSim bulk assessment) on the transformed gene feature profiles, selecting the optimal number of clusters by maximising Silhouette's coefficient while minimising intra-cluster variability. Then run PCA on the transformed data and verify that cluster assignments align with the simulation settings (samples separate by time point and experimental group in the PC space). Count the fraction of profiles assigned to incorrect clusters — the simulation is accepted if <0.5% of profiles fall in incorrect clusters. This clustering-alignment check complements the regulator-gene Pearson correlation check (99.4% of modelled activator/repressor interactions above 0.9, 0% of no-effect pairs), which can be compiled into mosim_bulk_assessment_metrics.csv.

## Related tools

- **MOSim** (Generates the simulated bulk multi-omics count matrices and design whose clustering-PCA alignment is verified) — github.com/ConesaLab/MOSim
- **R** (Implementation environment for K-means clustering, PCA, and Silhouette-based cluster-number selection)

## Examples

```
Rscript -e 'km <- kmeans(log(x + 1), centers=7); p <- prcomp(log(x + 1)); plot(p$x[,1], p$x[,2], col=km$cluster)'
```

## Evaluation signals

- Fraction of profiles assigned to incorrect clusters is <0.5% (validation threshold for accepting the bulk simulation)
- PCA projection shows clusters separating according to the 5 time points and 2 experimental groups used in the simulation
- Optimal k chosen by maximising Silhouette's coefficient while minimising intra-cluster variability is consistent with the simulated co-expression structure (k=7 in the MOSim assessment)
- Complementary metric: ~99.4% of modelled activator/repressor interactions have Pearson correlation >0.9 while 0% of no-effect random pairs reach this threshold
- All assessment metrics are recorded in mosim_bulk_assessment_metrics.csv

## Limitations

- Only applicable to bulk simulation assessment; single-cell data use a different clustering strategy (Spearman distance + K-medoids)
- MOSim operates at the count matrix level rather than simulating read data, so alignment checks cannot assess read-level artifacts
- K-means requires choosing k; an inappropriate k can misstate the fraction of profiles in incorrect clusters — use the Silhouette/intra-cluster-variability criterion

## Evidence

- [methods] Optimal cluster number selection: [evidence span withheld — anchor status: not_found]
