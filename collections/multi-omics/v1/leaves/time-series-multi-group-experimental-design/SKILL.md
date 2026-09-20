---
name: time-series-multi-group-experimental-design
description: Use when you need a synthetic multi-omics dataset with controlled experimental
  structure (e.g. 2 groups, 3 replicates, 6 time points) to benchmark multi-omics
  integration or multi-layer GRN inference methods.
when_to_use_negative:
- You need single-cell multi-omics with more than scRNA-seq and scATAC-seq — MOSim's
  single-cell module is restricted to those two assays
- You need spatially resolved GRN simulation — MOSim is not prepared to simulate GRNs
  with spatial resolution
- You need proteomics, metabolomics, or read-level (FASTQ) simulation — MOSim operates
  at the count matrix level for sequencing-based layers only
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MOSim
  role: R package used to simulate bulk multi-omics count data with customizable experimental
    groups, replicates, time series, DEG fraction, and repressor regulatory effects
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Language environment in which MOSim (a Bioconductor package) is installed
    and invoked via the mosim function
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/421834
    title: 'MOSim: bulk and single-cell multi-layer regulatory network simulator'
  source_content_hash: sha256:5e5de8b6b066090e72ebe712af2fcac93649bf955d34804cd498e300b44c30e6
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# time-series-multi-group-experimental-design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure MOSim bulk simulations with custom time-series, multi-group experimental designs — specifying number of experimental groups, biological replicates, time points, sequencing depth, percentage of differentially expressed genes, and fraction of repressor regulatory effects — to generate synthetic multi-omics count matrices for benchmarking and validation.

## When to use

Use when you need a synthetic multi-omics dataset with controlled experimental structure (e.g. 2 groups, 3 replicates, 6 time points) to benchmark multi-omics integration or multi-layer GRN inference methods.

## When NOT to use

- You need single-cell multi-omics with more than scRNA-seq and scATAC-seq — MOSim's single-cell module is restricted to those two assays
- You need spatially resolved GRN simulation — MOSim is not prepared to simulate GRNs with spatial resolution
- You need proteomics, metabolomics, or read-level (FASTQ) simulation — MOSim operates at the count matrix level for sequencing-based layers only

## Inputs

- Seed bulk multi-omics dataset (e.g. STATegra mouse multi-omics data)
- Regulator-gene association matrix (miRNA/TF-to-gene links)
- Design specification: number of groups, replicates, time points, sequencing depth, DEG percentage, repressor fraction

## Outputs

- Simulated count matrices per omic layer (RNA-seq, miRNA-seq, TF, optionally ATAC-seq/ChIP-seq/Methyl-seq)
- Active regulatory relationships between regulators and genes (activator and repressor edges)

## How to apply

Load a seed dataset (e.g. the STATegra mouse multi-omics data) and a miRNA/TF-to-gene regulator association matrix in R, then call the mosim function with the desired omic layers (RNA-seq, miRNA-seq, TF, plus optionally ATAC-seq, ChIP-seq, Methyl-seq). Set the design parameters explicitly: number of experimental groups (e.g. 2), biological replicates (e.g. 3), time points (e.g. 6), sequencing depth (~30M reads), and the fraction of differentially expressed genes (e.g. 5%). Tune regulation by specifying what share of miRNA regulatory effects act as repressors (e.g. 40%) so the simulation contains both activator and repressor edges. MOSim generates count matrices per omic layer via a negative-binomial model for bulk data, together with the active regulatory relationships between regulators and genes. The rationale is that few existing simulators expose these customization options, so explicit design control is what makes downstream benchmarking of integration and GRN-inference pipelines meaningful.

## Related tools

- **MOSim** (R package used to simulate bulk multi-omics count data with customizable experimental groups, replicates, time series, DEG fraction, and repressor regulatory effects) — github.com/ConesaLab/MOSim
- **R** (Language environment in which MOSim (a Bioconductor package) is installed and invoked via the mosim function)

## Examples

```
if (!requireNamespace("BiocManager", quietly = TRUE)) install.packages("BiocManager"); BiocManager::install("MOSim")
```

## Evaluation signals

- The simulated design matches requested settings: number of groups, replicates, time points, ~30M reads depth, and ~5% differentially expressed genes are recovered from the count matrices' metadata/structure
- Per-group cross-layer heatmaps (log(x + 1) transformed, per-layer scaled) show distinct feature profiles consistent with both activator and repressor regulatory effects
- Pearson's correlation between simulated gene-regulator pairs in bulk data separates from 20,000 random feature pairs with no simulated regulatory effects
- K-means clustering of simulated bulk feature profiles recovers the simulated temporal expression patterns
- The regulator-gene association matrix links each simulated regulator to its target genes as configured

## Limitations

- Simulation operates at the count matrix level only; read-level data is not simulated
- Single-cell simulation is restricted to scRNA-seq and scATAC-seq
- Proteomics and metabolomics layers are not supported; interactions between gene expression, proteome, and metabolome are deferred to future work
- No capability to simulate GRNs with spatial resolution
- MOSim software version is not stated in the section

## Evidence
