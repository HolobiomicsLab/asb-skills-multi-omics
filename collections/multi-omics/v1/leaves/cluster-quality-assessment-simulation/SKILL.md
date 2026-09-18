---
name: cluster-quality-assessment-simulation
description: Use when you need to verify that a simulator (e.g., sc_mosim) reproduced
  intended cell-type expression profiles and gene-peak regulatory ground truth before
  benchmarking clustering or GRN-inference methods on the simulated data.
when_to_use_negative:
- Input is bulk RNA-seq simulation output — the article's bulk assessment uses K-means
  clustering and Pearson's correlation, not K-medoids with Spearman distance
- Input is raw per-cell counts that have not been aggregated to average count per
  cell type with log(x + 1) transformation
- The data are real experimental (not simulated) single-cell data — this skill validates
  simulation ground truth, not biological clustering quality
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_3308
tools:
- name: MOSim
  role: Generates the simulated scRNA-seq/scATAC-seq count matrices and the gene-peak
    regulatory ground truth that this skill validates
  repo: github.com/ConesaLab/MOSim
- name: SPARSim
  role: Underlying count-data simulator leveraged by sc_mosim for the single-cell
    simulation being assessed
- name: Seurat
  role: Source of the human pbmc.multiome 10x Genomics seed dataset (via SeuratData)
    used as simulation input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/421834
    title: 'MOSim: bulk and single-cell multi-layer regulatory network simulator'
  source_content_hash: sha256:c725238db5f6fb19564f38b16779c5e71cd0277fdd2d2f92643677f161d345f1
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
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# cluster-quality-assessment-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A validation workflow for multi-omics simulators (e.g., MOSim's sc_mosim) that checks whether simulated single-cell count matrices reproduce the intended cell-type-specific expression profiles, using cell-type aggregation, log(x+1) transformation, K-medoids clustering with Spearman distance, PCA, and Kendall's Tb correlation of regulatory vs. non-regulatory feature pairs.

## When to use

Use when you need to verify that a simulator (e.g., sc_mosim) reproduced intended cell-type expression profiles and gene-peak regulatory ground truth before benchmarking clustering or GRN-inference methods on the simulated data.

## When NOT to use

- Input is bulk RNA-seq simulation output — the article's bulk assessment uses K-means clustering and Pearson's correlation, not K-medoids with Spearman distance
- Input is raw per-cell counts that have not been aggregated to average count per cell type with log(x + 1) transformation
- The data are real experimental (not simulated) single-cell data — this skill validates simulation ground truth, not biological clustering quality

## Inputs

- Simulated scRNA-seq count matrix from MOSim sc_mosim (6 cell types, 2 groups, 3 replicates)
- Simulated scATAC-seq peak count matrix from MOSim sc_mosim
- Simulated gene-peak regulatory interaction list (activator/repressor vs. no-effect pairs)
- Set of ~20,000 random non-regulatory feature pairs for the correlation baseline

## Outputs

- Cell-type-cluster assignment of gene expression profiles (K-medoids, Spearman distance)
- PCA plot confirming cluster alignment with simulation settings
- Kendall's Tb correlation distributions for regulatory vs. random feature pairs
- sc_mosim_assessment_metrics.csv with misclustering rate and correlation-threshold metrics

## How to apply

After running sc_mosim to generate scRNA-seq and scATAC-seq counts for 6 cell types, 2 groups, and 3 biological replicates (30% up- and 20% down-regulated DEGs), aggregate the simulated count matrix to the average count per cell type and apply a log(x + 1) transformation. Cluster gene expression profiles across cell types using partition around medoids (K-medoids, K=10) with Spearman distance (1 - Spearman's correlation); select the optimal cluster number by combining maximisation of Silhouette's coefficient with minimisation of intra-cluster variability. Use PCA to confirm that clustering aligns with the simulation settings (i.e., cells of the same simulated type co-cluster). For regulatory ground-truth validation, compute Kendall's Tb correlation for simulated gene-peak regulatory interactions and compare against 20,000 random feature pairs with no simulated regulatory effects. Judge success against the paper's reported metrics: 3.3% of profiles in incorrect clusters, 79.5% of modelled activator/repressor interactions with absolute correlation > 0.7, and no-effect interactions centered at 0.32 absolute correlation, writing results to sc_mosim_assessment_metrics.csv.

## Related tools

- **MOSim** (Generates the simulated scRNA-seq/scATAC-seq count matrices and the gene-peak regulatory ground truth that this skill validates) — github.com/ConesaLab/MOSim
- **SPARSim** (Underlying count-data simulator leveraged by sc_mosim for the single-cell simulation being assessed)
- **Seurat** (Source of the human pbmc.multiome 10x Genomics seed dataset (via SeuratData) used as simulation input)

## Examples

```
devtools::install_github("ConesaLab/MOSim")
```

## Evaluation signals

- Misclustering rate: ~3.3% of simulated profiles fall in incorrect clusters (paper-reported benchmark)
- 79.5% of interactions with modelled activator or repressor effects have absolute Kendall's Tb correlation values exceeding 0.7
- 'No effect' random interactions show a broader correlation distribution centered at 0.32 absolute correlation — a clear separation from modelled interactions is expected
- PCA of the log(x + 1)-transformed, cell-type-averaged profiles must align clusters with the simulated cell types
- Optimal K confirmed by maximisation of Silhouette's coefficient combined with minimisation of intra-cluster variability

## Limitations

- Single-cell simulation is restricted to scRNA-seq and scATAC-seq, as these are presently the only two commercially available sequencing techniques
- MOSim operates at the count matrix level rather than simulating read data
- MOSim software version is not stated in the assessment section, complicating exact reproduction

## Evidence

- [methods] K-medoids with Spearman distance on cell-type-averaged profiles: [evidence span withheld — anchor status: not_found]
- [methods] Kendall's Tb correlation for single-cell regulatory validation: [evidence span withheld — anchor status: not_found]
- [readme] sc_mosim functionality available via GitHub developer install: [evidence span withheld — anchor status: external_doc]
