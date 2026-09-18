---
name: regulator-gene-association-configuration
description: Use when building synthetic multi-omics benchmarks of multi-layer GRN
  inference or integration pipelines, and you must define which regulators (miRNAs,
  TFs, ATAC/ChIP/Methyl signals) regulate which transcripts.
when_to_use_negative:
- You need read-level (FASTQ) simulation — MOSim operates at the count matrix level
  rather than simulating read data
- You need proteomics, metabolomics, or spatially resolved GRN layers — the simulator
  is limited to sequencing-based gene regulatory relationships
- You need single-cell multi-omics beyond scRNA-seq and scATAC-seq — single-cell simulation
  is restricted to these two techniques
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MOSim
  role: R Bioconductor package whose mosim() function consumes the seed dataset and
    regulator-gene association matrix to produce bulk multi-omics counts and active
    regulatory edges
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Language/environment in which MOSim is implemented and invoked (BiocManager
    or devtools installation)
  repo: github.com/ConesaLab/MOSim
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/421834
    title: 'MOSim: bulk and single-cell multi-layer regulatory network simulator'
  source_content_hash: sha256:e813903fb6c2379a4e7b0216f85dede682ca21ace83980a89e21a53193b0de4d
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# regulator-gene-association-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure MOSim simulations by supplying a seed dataset and a regulator-to-gene association matrix (e.g., miRNA/TF-to-gene links) so that the simulator generates count matrices for RNA-seq, miRNA-seq, ATAC-seq, ChIP-seq and Methyl-seq layers together with the active regulatory relationships between regulators and genes.

## When to use

Use when building synthetic multi-omics benchmarks of multi-layer GRN inference or integration pipelines, and you must define which regulators (miRNAs, TFs, ATAC/ChIP/Methyl signals) regulate which transcripts.

## When NOT to use

- You need read-level (FASTQ) simulation — MOSim operates at the count matrix level rather than simulating read data
- You need proteomics, metabolomics, or spatially resolved GRN layers — the simulator is limited to sequencing-based gene regulatory relationships
- You need single-cell multi-omics beyond scRNA-seq and scATAC-seq — single-cell simulation is restricted to these two techniques

## Inputs

- Seed bulk multi-omics dataset (e.g., STATegra mouse multi-omics data) supplying realistic expression baselines
- Regulator-gene association matrix linking miRNAs and/or TFs (and optionally ATAC-seq/ChIP-seq/Methyl-seq features) to target transcripts
- Experimental design parameters: number of groups, replicates, time points, depth, DEG fraction, repressor fraction

## Outputs

- Simulated count matrices for each configured omic layer (e.g., RNA-seq, miRNA-seq, TF)
- Active regulatory relationships between regulatory omics and gene expression, tagged as activator or repressor effects
- Per-group GRN visualizations and cross-layer expression heatmaps (log(x + 1)-transformed, per-layer scaled)

## How to apply

Provide a seed dataset (e.g., the STATegra mouse multi-omics data) plus a regulator-gene association matrix as inputs to the mosim() function from the R Bioconductor package MOSim. Specify the omic layers to simulate (RNA-seq, miRNA-seq, TF; optionally ATAC-seq, ChIP-seq, Methyl-seq) and the experimental design — number of experimental groups, biological replicates, time points, sequencing depth (~30M reads), fraction of differentially expressed genes (e.g., 5%), and the fraction of miRNA regulatory effects set as repressors (e.g., 40%) versus activators. Bulk data are simulated under a negative binomial model, while sc_mosim leverages SPARSim for single-cell counts. Extract the per-omic simulated count matrices and the reported active regulatory relationships between regulators and genes; visualize per-group GRNs highlighting activator and repressor edges, and apply a log(x + 1) transformation with per-layer scaling for cross-layer heatmaps. The rationale is that specifying the association matrix lets users tailor which regulator-transcript links can become active, enabling controlled benchmarking of multi-layer GRN inference.

## Related tools

- **MOSim** (R Bioconductor package whose mosim() function consumes the seed dataset and regulator-gene association matrix to produce bulk multi-omics counts and active regulatory edges) — github.com/ConesaLab/MOSim
- **R** (Language/environment in which MOSim is implemented and invoked (BiocManager or devtools installation)) — github.com/ConesaLab/MOSim

## Examples

```
if (!requireNamespace("BiocManager", quietly = TRUE)) install.packages("BiocManager"); BiocManager::install("MOSim")
```

## Evaluation signals

- mosim() returns, for each configured omic layer, a quantification matrix plus the active regulatory relationships between regulators and genes
- Pearson's correlation computed on simulated gene-regulator pairs in the bulk data is compared against 20,000 random feature pairs with no simulated regulatory effects; simulated pairs must show consistent activator/repressor patterns
- Cross-layer heatmaps (after log(x + 1) transformation and per-layer scaling) show distinct feature profiles per experimental group that reflect both activator and repressor effects
- Simulated bulk counts follow the negative binomial model with the requested design (groups, replicates, time points, DEG fraction, e.g., 5% DE genes and 40% repressor miRNA effects)

## Limitations

- MOSim operates at the count matrix level rather than simulating read data
- Single-cell simulation is restricted to scRNA-seq and scATAC-seq, as these are presently the only two commercially available sequencing techniques
- Both bulk and single-cell modules are designed to simulate gene regulatory relationships based on sequencing data, limiting applicability to other omics layers like proteomics and metabolomics
- MOSim is not prepared to simulate GRN with spatial resolution, which could be inferred from spatial multi-omics data

## Evidence

- [abstract] MOSim enables users to generate quantification matrices for each simulated omics data type: [evidence span withheld — anchor status: not_found]
