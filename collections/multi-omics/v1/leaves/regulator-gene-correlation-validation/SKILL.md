---
name: regulator-gene-correlation-validation
description: Use when you have simulated (or benchmarked) bulk RNA-seq/ATAC-seq count
  matrices plus a regulator-gene association matrix and need to verify that simulated
  regulatory interactions show expected correlations versus no-effect pairs.
when_to_use_negative:
- Evaluating single-cell simulation output, where the article prescribes Kendall's
  Tb correlation on Spearman-distance K-medoids clusters rather than Pearson's correlation
  on bulk profiles
- Input data lacks an association matrix defining which regulator-gene pairs have
  modelled effects, so no-effect baselines cannot be distinguished from true interactions
- Data are raw counts not yet log(x + 1) transformed, since the correlation assessment
  is specified on transformed profiles
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0204
tools:
- name: MOSim
  role: R Bioconductor package used to simulate bulk (negative binomial) and single-cell
    (SPARSim) multi-omics count data whose regulator-gene interactions are then validated
    by correlation
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Implementation environment for MOSim and the correlation, clustering, and
    PCA assessment steps
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/421834
    title: 'MOSim: bulk and single-cell multi-layer regulatory network simulator'
  source_content_hash: sha256:c8311a531af00df1c751f5eb712c0e538a9b7c7750e61ff0c1d8d4aac4ed82a9
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
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# regulator-gene-correlation-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates multi-omics simulation output by computing Pearson's correlation between all simulated regulator-gene pairs and comparing against random feature pairs with no simulated regulatory effects, confirming that modelled activator/repressor relationships are reproduced in the simulated counts.

## When to use

Use when you have simulated (or benchmarked) bulk RNA-seq/ATAC-seq count matrices plus a regulator-gene association matrix and need to verify that simulated regulatory interactions show expected correlations versus no-effect pairs.

## When NOT to use

- Evaluating single-cell simulation output, where the article prescribes Kendall's Tb correlation on Spearman-distance K-medoids clusters rather than Pearson's correlation on bulk profiles
- Input data lacks an association matrix defining which regulator-gene pairs have modelled effects, so no-effect baselines cannot be distinguished from true interactions
- Data are raw counts not yet log(x + 1) transformed, since the correlation assessment is specified on transformed profiles

## Inputs

- Simulated bulk count matrices (RNA-seq and ATAC-seq) from MOSim's mosim() function
- Regulator-gene association matrix linking regulators with transcripts
- Seed dataset (e.g. STATegra mouse multi-omics RNA-seq/ATAC-seq data)
- Random no-effect feature pairs (e.g. 20,000 pairs) as null baseline

## Outputs

- Pearson correlation values for all simulated regulator-gene pairs
- Pearson correlation values for random no-effect feature pairs
- Assessment metrics table (e.g. mosim_bulk_assessment_metrics.csv): fraction of modelled interactions with correlation >0.9 and fraction of no-effect interactions exceeding that threshold

## How to apply

Run the simulator (e.g. mosim() with STATegra seed data, 5 time points, 2 experimental groups, 3 biological replicates, 15% DEGs, negative binomial bulk simulation), then apply a log(x + 1) transformation to the simulated counts. Compute Pearson's correlation for all simulated regulator-gene pairs defined by the association matrix, and separately for ~20,000 random feature pairs with no simulated regulatory effects as a null baseline. Accept the simulation when roughly 99.4% of interactions with a modelled activator or repressor effect exceed a Pearson correlation of 0.9, while 0% of no-effect interactions reach that threshold. Optionally complement this with K-means clustering (k selected by maximising Silhouette's coefficient and minimising intra-cluster variability) and PCA to confirm temporal expression profiles align with simulation settings; use Kendall's Tb correlation instead of Pearson's for single-cell data.

## Related tools

- **MOSim** (R Bioconductor package used to simulate bulk (negative binomial) and single-cell (SPARSim) multi-omics count data whose regulator-gene interactions are then validated by correlation) — github.com/ConesaLab/MOSim
- **R** (Implementation environment for MOSim and the correlation, clustering, and PCA assessment steps)

## Examples

```
if (!requireNamespace("BiocManager", quietly = TRUE)) install.packages("BiocManager"); BiocManager::install("MOSim")
```

## Evaluation signals

- ~99.4% of interactions with a modelled activator or repressor effect show Pearson correlation > 0.9 in the bulk ATAC-seq simulation
- 0% of random no-effect interactions (out of ~20,000 pairs) reach the 0.9 correlation threshold
- Fewer than 0.5% of gene profiles are assigned to an incorrect K-means cluster (k=7 chosen by maximising Silhouette's coefficient while minimising intra-cluster variability)
- PCA on log(x + 1)-transformed data confirms clustering aligns with the simulation settings (5 time points, 2 groups, 3 replicates)

## Limitations

- MOSim operates at the count matrix level rather than simulating read data
- Single-cell simulation is restricted to scRNA-seq and scATAC-seq; proteomics and metabolomics layers are not supported
- Simulation results depend on the seed dataset and regulator-gene association matrix supplied by the user; the article's discussion section does not name the seed dataset source or MOSim software version

## Evidence

- [methods] Pearson's correlation was computed for bulk data: [evidence span withheld — anchor status: not_found]
- [methods] Optimal cluster selection rule: [evidence span withheld — anchor status: not_found]
