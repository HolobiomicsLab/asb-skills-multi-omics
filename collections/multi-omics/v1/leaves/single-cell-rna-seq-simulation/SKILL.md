---
name: single-cell-rna-seq-simulation
description: Use when benchmarking or validating single-cell methods (clustering,
  multi-omics integration, GRN inference) requires synthetic scRNA-seq count matrices
  with known cell-type labels, replicates, experimental groups, and simulated gene-peak
  regulatory interactions.
when_to_use_negative:
- The analysis needs read-level sequencing data — MOSim operates at the count matrix
  level only
- The analysis requires single-cell omics beyond scRNA-seq/scATAC-seq (e.g. proteomics,
  metabolomics, spatially resolved data) — these are unsupported
- The goal is bulk multi-omics simulation, where the mosim function (negative binomial-based)
  rather than sc_mosim applies
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MOSim
  role: R Bioconductor package providing the sc_mosim function to simulate single-cell
    multi-omics count matrices with ground-truth regulatory relationships
  repo: github.com/ConesaLab/MOSim
- name: SPARSim
  role: Internal single-cell count data simulator that sc_mosim relies on for scRNA-seq
    count generation
- name: R
  role: Implementation language; MOSim is run as an R Bioconductor package
- name: Seurat
  role: Source of the pbmc.multiome 10x Genomics seed dataset (SeuratData R package)
    used as simulation input
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1101/421834
    title: 'MOSim: bulk and single-cell multi-layer regulatory network simulator'
  source_content_hash: sha256:6de02a2f27cc21951f722b3a9bb9e4b8374ad426b57347b028c70014d2c48223
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
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# single-cell-rna-seq-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate simulated single-cell RNA-seq count matrices with known ground-truth cell types and regulatory relationships using the sc_mosim function of the MOSim R package, which internally leverages the SPARSim single-cell count simulator seeded from a real dataset such as the human pbmc.multiome 10x Genomics dataset.

## When to use

Use when benchmarking or validating single-cell methods (clustering, multi-omics integration, GRN inference) requires synthetic scRNA-seq count matrices with known cell-type labels, replicates, experimental groups, and simulated gene-peak regulatory interactions.

## When NOT to use

- The analysis needs read-level sequencing data — MOSim operates at the count matrix level only
- The analysis requires single-cell omics beyond scRNA-seq/scATAC-seq (e.g. proteomics, metabolomics, spatially resolved data) — these are unsupported
- The goal is bulk multi-omics simulation, where the mosim function (negative binomial-based) rather than sc_mosim applies

## Inputs

- Seed scRNA-seq dataset (e.g. human pbmc.multiome 10x Genomics from the SeuratData R package)
- Regulator-gene association matrix linking regulators with transcripts
- Simulation design parameters: number of cell types, experimental groups, biological replicates, and DEG proportions

## Outputs

- Simulated scRNA-seq count matrix with per-cell ground-truth cell-type labels
- Active gene-peak (gene-TF) regulatory relationship ground truth for GRN benchmarking
- Validation metrics (e.g. clustering accuracy, Kendall's Tb correlation distributions)

## How to apply

Install MOSim as a Bioconductor R package (or the developer version from GitHub via devtools::install_github("ConesaLab/MOSim"), which includes the sc_mosim functionalities). Load a seed dataset — e.g. the human pbmc.multiome 10x Genomics dataset from the SeuratData R package — plus a regulator-gene association matrix, then call sc_mosim specifying cell types, experimental groups, biological replicates, and DEG proportions (e.g. 2 groups, 3 replicates, 30% upregulated and 20% downregulated DEGs). sc_mosim internally leverages SPARSim to produce the scRNA-seq (and scATAC-seq) count matrices and reports the active regulatory relationships as ground truth. For downstream benchmarking, aggregate the simulated count matrix to the average count per cell type and apply a log(x + 1) transformation before clustering or embedding. Validate the simulation by clustering gene expression profiles across cell types with K-medoids (Spearman distance, optimal K via Silhouette's coefficient and intra-cluster variability, confirmed by PCA) and by computing Kendall's Tb correlation for simulated gene-peak regulatory interactions against random non-regulatory feature pairs.

## Related tools

- **MOSim** (R Bioconductor package providing the sc_mosim function to simulate single-cell multi-omics count matrices with ground-truth regulatory relationships) — github.com/ConesaLab/MOSim
- **SPARSim** (Internal single-cell count data simulator that sc_mosim relies on for scRNA-seq count generation)
- **R** (Implementation language; MOSim is run as an R Bioconductor package)
- **Seurat** (Source of the pbmc.multiome 10x Genomics seed dataset (SeuratData R package) used as simulation input)

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install("MOSim")
```

## Evaluation signals

- K-medoids (Spearman distance) clustering of average per-cell-type expression profiles recovers the simulated cell types, with only a small fraction of profiles (reported: 3.3%) in incorrect clusters and PCA confirming alignment with simulation settings
- Kendall's Tb correlation of simulated gene-peak regulatory interactions shows ~79.5% of modelled interactions with absolute correlation > 0.7, versus no-effect pairs centered at 0.32 absolute correlation
- A downstream single-cell clustering method (e.g. scDHA after log(x + 1) transformation) recovers the simulated cell types with high agreement (reported ARI = 0.949)
- Simulated count matrices are delivered at the count level and accept the log(x + 1) transformation as in the article's assessment workflow
- The simulation reproduces the specified design: requested cell types, experimental groups, biological replicates, and DEG proportions are reflected in the output matrices

## Limitations

- Single-cell simulation is restricted to scRNA-seq and scATAC-seq only, as these are presently the only two commercially available sequencing techniques
- MOSim operates at the count matrix level rather than simulating read data
- Proteomics and metabolomics layers, spatially resolved GRNs, and gene-expression/proteome/metabolome interactions are not supported and deferred to future work; the MOSim software version is not stated

## Evidence

- [readme] SPARSim citation requirement when using sc_mosim: [evidence span withheld — anchor status: external_doc]
