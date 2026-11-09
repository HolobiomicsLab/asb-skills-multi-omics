---
name: cross-layer-expression-heatmap-generation
description: Use when you have simulated (or real) count matrices for several omic
  layers plus group/time-point design settings, and you need per-group visualizations
  whose rows/columns align across layers to inspect regulatory patterns.
when_to_use_negative:
- Input is raw read-level sequencing data rather than a count matrix — MOSim and this
  visualization operate at the count matrix level only
- Only a single omic layer is available — a cross-layer aligned heatmap requires at
  least two comparable omic layers
- The goal is spatially resolved regulatory analysis — MOSim is not prepared to simulate
  GRNs with spatial resolution
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: MOSim
  role: Generates the multi-layer count matrices and active regulatory relationships
    that feed the cross-layer heatmaps
  repo: https://github.com/ConesaLab/MOSim
- name: R
  role: Implementation environment; log transformation, per-layer scaling, and heatmap
    rendering are done in R
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/421834
    title: 'MOSim: bulk and single-cell multi-layer regulatory network simulator'
  source_content_hash: sha256:d170a85adfb5b62ba170384d129c3d96ba2ce3ecc611a0320a6d33bb42283404
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cross-layer-expression-heatmap-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Renders aligned cross-layer expression heatmaps from multi-omics simulation output (e.g., MOSim RNA-seq, miRNA-seq and TF count matrices) after log(x + 1) transformation and per-layer scaling, so that simulated feature profiles can be compared consistently across omic layers and experimental groups.

## When to use

Use when you have simulated (or real) count matrices for several omic layers plus group/time-point design settings, and you need per-group visualizations whose rows/columns align across layers to inspect regulatory patterns.

## When NOT to use

- Input is raw read-level sequencing data rather than a count matrix — MOSim and this visualization operate at the count matrix level only
- Only a single omic layer is available — a cross-layer aligned heatmap requires at least two comparable omic layers
- The goal is spatially resolved regulatory analysis — MOSim is not prepared to simulate GRNs with spatial resolution

## Inputs

- Simulated count matrix for RNA-seq layer (from MOSim mosim() output)
- Simulated count matrix for miRNA-seq layer
- Simulated count matrix (or profiles) for TF layer
- Design metadata: experimental groups, biological replicates, time points
- Active regulatory relationships between regulators and genes (for annotating activator/repressor edges)

## Outputs

- Per-group cross-layer expression heatmap (e.g., Figure 4C style) with aligned features across omic layers
- Log-transformed and per-layer-scaled expression matrices
- Optional per-group GRN visualizations highlighting activator and repressor edges

## How to apply

Extract the simulated count matrices for each omics layer (RNA-seq, miRNA-seq, TF) from the MOSim output object along with the design metadata (2 experimental groups, 3 biological replicates, 6 time points in the reference case). Apply a log(x + 1) transformation to each layer, then scale per layer so features from different omics are comparable on one color scale. Render a cross-layer expression heatmap per experimental group, ordered to match the simulation settings (e.g., time points, DEG subsets such as the first 100 differentially expressed genes). Optionally pair the heatmaps with per-group GRN visualizations that highlight simulated activator versus repressor edges, then validate that heatmap patterns reflect known regulatory sign by computing Pearson's correlation for simulated gene-regulator pairs against ~20,000 random feature pairs with no simulated regulatory effects.

## Related tools

- **MOSim** (Generates the multi-layer count matrices and active regulatory relationships that feed the cross-layer heatmaps) — https://github.com/ConesaLab/MOSim
- **R** (Implementation environment; log transformation, per-layer scaling, and heatmap rendering are done in R)

## Examples

```
if (!requireNamespace("BiocManager", quietly = TRUE)) install.packages("BiocManager"); BiocManager::install("MOSim")
```

## Evaluation signals

- Heatmaps are rendered separately for each experimental group and their feature ordering aligns with the simulation settings (groups, replicates, time points)
- Per-layer scaling makes color intensities comparable across RNA-seq, miRNA-seq and TF panels after log(x + 1) transformation
- Cross-layer heatmap profiles show distinct patterns consistent with simulated activator and repressor regulatory effects (Figure 4C behavior)
- Pearson's correlation for simulated gene-regulator pairs differs from the null distribution over ~20,000 random feature pairs with no simulated regulatory effects
- PCA or clustering (e.g., K-means on simulated feature profiles) confirms the displayed structure aligns with the intended simulation settings

## Limitations

- MOSim operates at the count matrix level rather than simulating read data, so heatmaps reflect counts only
- Only layers supported by MOSim (RNA-seq, ATAC-seq, miRNA-seq, ChIP-seq, Methyl-seq, TF, and scRNA-seq/scATAC-seq for single-cell) can be visualized; proteomics and metabolomics layers are not supported
- No capability to simulate or visualize GRNs with spatial resolution

## Evidence

- [methods] Validation of displayed patterns: [evidence span withheld — anchor status: not_found]
