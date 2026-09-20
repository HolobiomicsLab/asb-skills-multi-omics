---
name: bulk-multi-omics-count-simulation
description: Use when synthetic bulk multi-omics count matrices with known ground-truth
  regulator-gene interactions are needed for benchmarking multi-omics integration
  or GRN inference pipelines.
when_to_use_negative:
- Input is already a feature table or the goal is single-cell multi-omics simulation
  (use sc_mosim with SPARSim instead, which covers only scRNA-seq and scATAC-seq)
- The target omics layers are proteomics or metabolomics — MOSim only simulates gene
  regulatory relationships based on sequencing data
- Spatially resolved GRN simulation is required — MOSim is not prepared to simulate
  GRNs with spatial resolution
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: MOSim
  role: R package whose mosim() function performs the bulk multi-omics count simulation
    and reports active regulator-gene relationships
  repo: https://github.com/ConesaLab/MOSim
- name: R
  role: Execution environment; MOSim is a Bioconductor package installed via BiocManager
  repo: https://github.com/ConesaLab/MOSim
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_001
  source_papers:
  - doi: 10.1101/421834
    title: 'MOSim: bulk and single-cell multi-layer regulatory network simulator'
  source_content_hash: sha256:48a724fad61ce35618a2e88100742feb282715739af1186e586f2f12d52404b7
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
  - MORE
  - MOSim
  - NOISeq
  - R
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# bulk-multi-omics-count-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulate consistent multi-layer bulk multi-omics count data (RNA-seq, ATAC-seq, miRNA-seq, ChIP-seq, Methyl-seq, TF regulation) with the MOSim R package, using a seed dataset and regulator-gene association matrix to generate realistic count matrices and active regulatory relationships.

## When to use

Use when synthetic bulk multi-omics count matrices with known ground-truth regulator-gene interactions are needed for benchmarking multi-omics integration or GRN inference pipelines.

## When NOT to use

- Input is already a feature table or the goal is single-cell multi-omics simulation (use sc_mosim with SPARSim instead, which covers only scRNA-seq and scATAC-seq)
- The target omics layers are proteomics or metabolomics — MOSim only simulates gene regulatory relationships based on sequencing data
- Spatially resolved GRN simulation is required — MOSim is not prepared to simulate GRNs with spatial resolution

## Inputs

- Seed multi-omics count dataset (e.g. STATegra mouse RNA-seq, ATAC-seq, miRNA-seq)
- Regulator-gene association matrix linking regulators (miRNAs, TFs, chromatin peaks) with transcripts
- Experimental design specification (groups, replicates, time points, depth, DEG percentage, repressor fraction)

## Outputs

- Simulated count matrices for each bulk omic layer (negative binomial-based)
- Active regulatory relationships (activator/repressor edges) between regulatory omics and gene expression
- Assessment metrics (e.g. mosim_bulk_assessment_metrics.csv) with clustering accuracy and regulatory correlation fractions

## How to apply

Install MOSim from Bioconductor and load the STATegra mouse multi-omics seed dataset (RNA-seq, ATAC-seq, miRNA-seq) plus the regulator-gene association matrix. Call the mosim() function, which simulates bulk data via the negative binomial distribution, configuring experimental design parameters such as number of experimental groups (e.g. 2), biological replicates (e.g. 3), time points (e.g. 5-20), sequencing depth (e.g. ~30M reads), percentage of differentially expressed genes (e.g. 5-50%), and fraction of repressor regulatory effects (e.g. 40-60%). Extract the simulated count matrices per omic layer together with the active regulatory relationships between regulators and genes, which serve as ground truth. Apply log(x + 1) transformation and per-layer scaling for downstream visualization such as cross-layer heatmaps. Validate by K-means clustering of simulated temporal profiles (optimal k via Silhouette's coefficient), PCA, and Pearson's correlation of simulated regulator-gene pairs benchmarked against 20,000 random feature pairs with no modelled regulatory effect.

## Related tools

- **MOSim** (R package whose mosim() function performs the bulk multi-omics count simulation and reports active regulator-gene relationships) — https://github.com/ConesaLab/MOSim
- **R** (Execution environment; MOSim is a Bioconductor package installed via BiocManager) — https://github.com/ConesaLab/MOSim

## Examples

```
BiocManager::install("MOSim"); sim <- mosim(omics = c("RNA-seq", "miRNA-seq", "TF"), groups = 2, replicates = 3, times = 6, depth = 30000000, diff.percentage = 0.05)
```

## Evaluation signals

- <0.5% of simulated feature profiles assigned to incorrect K-means clusters (k selected via Silhouette's coefficient)
- ~99.4% of modelled activator/repressor interactions show Pearson correlation >0.9 in the bulk ATAC-seq simulation
- 0% of interactions without a modelled effect reach the correlation >0.9 threshold (benchmark against 20,000 random feature pairs)
- PCA of log(x+1)-transformed data confirms clustering aligns with the simulation settings
- Simulated significant regulatory interactions reproduce expected activator and repressor patterns across molecular layers (e.g. Figure 4C cross-layer heatmaps)

## Limitations

- Single-cell simulation is restricted to scRNA-seq and scATAC-seq only
- No capability to simulate GRNs with spatial resolution; spatial multi-omics support left to future work
- Proteomics and metabolomics layers are not supported; simulation of gene-proteome-metabolome interactions deferred to future work
- MOSim operates at the count matrix level rather than simulating read data, and users must select a preferred library preparation method via the seed dataset

## Evidence

- [abstract] MOSim, an R package that simulates both bulk (via mosim function) and single-cell (via sc_mosim function) multi-omics data: "MOSim, an R package that simulates both bulk (via mosim function) and single-cell (via sc_mosim function) multi-omics data"
- [discussion] For bulk data, the simulation is based on the negative binomial distribution: "For bulk data, the simulation is based on the negative binomial distribution"
- [methods] K-means clustering [39] was applied to the simulated feature profiles to assess the correct simulation of temporal expression patterns.: "K-means clustering [39] was applied to the simulated feature profiles to assess the correct simulation of temporal expression patterns."
- [methods] a log transformation (log(x + 1))[42] was applied to the data: "a log transformation (log(x + 1))[42] was applied to the data"
- [discussion] Using a seed dataset and a regulator-gene association matrix, MOSim generates realistic simulated count matrices for both bulk and single-cell transcriptomics data: "Using a seed dataset and a regulator-gene association matrix, MOSim generates realistic simulated count matrices for both bulk and single-cell transcriptomics data"
- [readme] MOSim is a Bioconductor R package, and we strongly recommend that it is installed from the Bioconductor repository: "MOSim is a Bioconductor R package, and we strongly recommend that it is installed from the Bioconductor repository"
