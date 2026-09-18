---
name: gene-regulatory-network-visualization
description: Use when simulated or real multi-omics count matrices plus regulator-gene
  associations must be inspected as multi-layered GRNs with distinct activator vs.
  repressor edge types.
when_to_use_negative:
- Input is a real (not simulated) regulatory network where activator/repressor edge
  ground truth is unknown, since the validation via simulated-vs-random Pearson correlations
  relies on known simulated edges
- Data layers outside MOSim's scope such as proteomics, metabolomics, or spatially
  resolved GRNs, which the simulator explicitly does not support
- Single-cell datasets with omics other than scRNA-seq and scATAC-seq, since sc_mosim
  currently supports only those two layers
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MOSim
  role: R Bioconductor package that simulates the bulk multi-omics layers, returns
    active regulator-gene relationships, and supplies the count matrices and DEGs
    used for GRN and heatmap rendering
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Environment in which MOSim is installed via BiocManager and in which the GRN
    plots and heatmaps are produced
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/421834
    title: 'MOSim: bulk and single-cell multi-layer regulatory network simulator'
  source_content_hash: sha256:c7d8608fca815abb76f8957f5f039ecb02712300821ec7cfa4ba52dbe314b31c
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
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# gene-regulatory-network-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A method for rendering per-experimental-group gene regulatory network (GRN) visualizations from MOSim simulations, highlighting simulated activator and repressor regulatory edges across RNA-seq, miRNA-seq and TF layers, paired with cross-layer expression heatmaps.

## When to use

Use when simulated or real multi-omics count matrices plus regulator-gene associations must be inspected as multi-layered GRNs with distinct activator vs. repressor edge types.

## When NOT to use

- Input is a real (not simulated) regulatory network where activator/repressor edge ground truth is unknown, since the validation via simulated-vs-random Pearson correlations relies on known simulated edges
- Data layers outside MOSim's scope such as proteomics, metabolomics, or spatially resolved GRNs, which the simulator explicitly does not support
- Single-cell datasets with omics other than scRNA-seq and scATAC-seq, since sc_mosim currently supports only those two layers

## Inputs

- MOSim simulated count matrices for RNA-seq, miRNA-seq and TF omic layers
- Active regulatory relationships (regulator-to-gene edges) extracted from the mosim simulation object
- Seed dataset: STATegra mouse multi-omics data
- miRNA/TF-to-gene regulator association matrix

## Outputs

- Per-group GRN visualizations highlighting simulated activator and repressor edges (first 100 DEGs)
- Cross-layer expression heatmaps (log(x + 1) transformed, per-layer scaled) per experimental group
- Pearson correlation statistics for simulated vs. random gene-regulator pairs

## How to apply

Run MOSim in R with bulk layers (RNA-seq, miRNA-seq, TF), e.g. 2 experimental groups, 3 biological replicates, 6 time points, ~30M reads depth, 5% differentially expressed genes and 40% of miRNA regulatory effects configured as repressors, using the STATegra mouse multi-omics dataset as seed data. Extract the simulated count matrices for each omic layer along with the active regulatory relationships between regulators and genes returned by the simulation. Select the first 100 differentially expressed genes and render one GRN visualization per experimental group, visually distinguishing simulated activator edges from repressor edges. Apply a log(x + 1) transformation and per-layer scaling to the simulated profiles, then draw cross-layer heatmaps aligned with the simulation settings. Validate the rendered networks by computing Pearson's correlation for simulated gene-regulator pairs in the bulk data and comparing against 20,000 random feature pairs with no simulated regulatory effects, confirming that activator and repressor patterns are distinguishable.

## Related tools

- **MOSim** (R Bioconductor package that simulates the bulk multi-omics layers, returns active regulator-gene relationships, and supplies the count matrices and DEGs used for GRN and heatmap rendering) — github.com/ConesaLab/MOSim
- **R** (Environment in which MOSim is installed via BiocManager and in which the GRN plots and heatmaps are produced)

## Examples

```
if (!requireNamespace("BiocManager", quietly = TRUE)) install.packages("BiocManager"); BiocManager::install("MOSim")
```

## Evaluation signals

- Cross-layer heatmaps per experimental group show distinct feature profiles that reflect both activator and repressor regulatory effects (expected to reproduce the Figure 4C pattern)
- Pearson's correlation for simulated gene-regulator pairs in bulk data is clearly separated from the distribution of 20,000 random feature pairs with no simulated regulatory effects
- PCA and K-means clustering of simulated feature profiles align with the simulation settings (groups, time points)
- Rendered GRN plots contain edge sets matching the active regulatory relationships returned by mosim, with the configured ~40% repressor fraction among miRNA effects
- Count matrices for each omic layer are present with the requested design: 2 groups, 3 replicates, 6 time points, ~5% DEGs

## Limitations

- MOSim operates at the count matrix level rather than simulating read data
- Simulation of gene regulatory relationships is based on sequencing data, limiting applicability to other omics layers like proteomics and metabolomics
- MOSim is not prepared to simulate GRNs with spatial resolution
- Single-cell simulation is restricted to scRNA-seq and scATAC-seq

## Evidence

- [methods] Pearson validation: [evidence span withheld — anchor status: not_found]
