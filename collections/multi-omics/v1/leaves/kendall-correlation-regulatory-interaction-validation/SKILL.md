---
name: kendall-correlation-regulatory-interaction-validation
description: Use when validating single-cell multi-omics simulations where a ground-truth
  set of gene-peak regulatory interactions must be shown to exhibit correlations stronger
  than random non-regulatory feature pairs.
when_to_use_negative:
- For bulk MOSim simulations — the article specifies Pearson's correlation was computed
  for bulk data, while Kendall's Tb is used for single-cell data
- When the goal is validating temporal expression pattern simulation, where K-means
  clustering of feature profiles (not interaction correlations) is the appropriate
  assessment
- When no ground-truth regulator-target association matrix was supplied to the simulation,
  since there are no simulated regulatory effects to validate against random pairs
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: MOSim
  role: R package providing sc_mosim to simulate paired scRNA-seq/scATAC-seq counts
    and the ground-truth gene-peak regulatory interactions being validated
  repo: github.com/ConesaLab/MOSim
- name: SPARSim
  role: Underlying count-data simulator leveraged by sc_mosim for the single-cell
    count matrices whose interactions are validated
- name: R
  role: Implementation environment for sc_mosim, Kendall's Tb computation, and the
    assessment script
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/421834
    title: 'MOSim: bulk and single-cell multi-layer regulatory network simulator'
  source_content_hash: sha256:8f18bd2703a35ebe1e7eee5976a01474fec1b175d105340efa4181deb951a72b
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
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Kendall correlation regulatory-interaction validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that simulated gene-peak regulatory interactions from MOSim's sc_mosim carry stronger signal than non-regulatory pairs by computing Kendall's Tb correlation on aggregated, log-transformed single-cell counts and benchmarking against random feature pairs.

## When to use

Use when validating single-cell multi-omics simulations where a ground-truth set of gene-peak regulatory interactions must be shown to exhibit correlations stronger than random non-regulatory feature pairs.

## When NOT to use

- For bulk MOSim simulations — the article specifies Pearson's correlation was computed for bulk data, while Kendall's Tb is used for single-cell data
- When the goal is validating temporal expression pattern simulation, where K-means clustering of feature profiles (not interaction correlations) is the appropriate assessment
- When no ground-truth regulator-target association matrix was supplied to the simulation, since there are no simulated regulatory effects to validate against random pairs

## Inputs

- Simulated scRNA-seq and scATAC-seq count matrices from MOSim's sc_mosim function (e.g. 6 cell types, 2 experimental groups, 3 biological replicates, 30% up- and 20% downregulated DEGs, SPARSim-based)
- Simulated gene-peak regulatory interaction ground-truth list (activator/repressor and 'no effect' classes)
- Random feature pairs with no simulated regulatory effect (e.g. 20,000 pairs) as negative control

## Outputs

- Kendall's Tb correlation values for simulated regulatory gene-peak pairs
- Kendall's Tb correlation distribution for random non-regulatory feature pairs
- Summary metric: fraction of modelled interactions with absolute correlation > 0.7 (expected ~79.5%) and median/center of no-effect distribution (expected ~0.32), e.g. written to sc_mosim_assessment_metrics.csv

## How to apply

After running sc_mosim to simulate paired scRNA-seq and scATAC-seq counts (e.g. from the human pbmc.multiome 10x Genomics seed dataset in SeuratData), aggregate the simulated count matrix to the average count per cell type and apply a log(x + 1) transformation. Compute Kendall's Tb correlation between each simulated regulatory gene-peak pair and compare the distribution against ~20,000 random feature pairs with no simulated regulatory effect. The validation passes when a large majority of modelled activator/repressor interactions show high absolute correlation — in the reported assessment, 79.5% of interactions with modelled activator or repressor effects exceeded 0.7 absolute correlation, while 'no effect' interactions centered at 0.32 — confirming the simulation supplies reliable ground truth for GRN inference benchmarking.

## Related tools

- **MOSim** (R package providing sc_mosim to simulate paired scRNA-seq/scATAC-seq counts and the ground-truth gene-peak regulatory interactions being validated) — github.com/ConesaLab/MOSim
- **SPARSim** (Underlying count-data simulator leveraged by sc_mosim for the single-cell count matrices whose interactions are validated)
- **R** (Implementation environment for sc_mosim, Kendall's Tb computation, and the assessment script)

## Examples

```
devtools::install_github("ConesaLab/MOSim")
```

## Evaluation signals

- ≥ 79.5% of interactions with modelled activator or repressor effects show absolute Kendall correlation > 0.7
- Distribution of absolute correlations for 'no effect' random pairs is broader and centered around 0.32, clearly separated from the regulatory-pair distribution
- Counts were aggregated per cell type and log(x + 1) transformed before correlation, matching the published assessment protocol
- Comparison against ~20,000 random feature pairs is included as the negative control
- Final metrics are recorded (e.g. sc_mosim_assessment_metrics.csv) matching the reported values

## Limitations

- Single-cell simulation in MOSim is restricted to scRNA-seq and scATAC-seq only, so validation of other single-cell omic layers is not possible
- MOSim operates at the count matrix level rather than simulating read data, so correlations reflect simulated count structure only
- MOSim software version was not stated in the assessment section, and the developer GitHub version is needed for sc_mosim functionalities

## Evidence

- [methods] Kendall's Tb correlation for single-cell data: [evidence span withheld — anchor status: not_found]
- [readme] Developer version needed for sc_mosim: [evidence span withheld — anchor status: external_doc]
- [readme] SPARSim citation requirement: [evidence span withheld — anchor status: external_doc]
