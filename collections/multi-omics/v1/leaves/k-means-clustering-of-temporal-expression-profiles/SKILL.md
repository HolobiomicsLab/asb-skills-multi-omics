---
name: k-means-clustering-of-temporal-expression-profiles
description: Use when you need to validate that simulated multi-omics count data (e.g.
  MOSim bulk RNA-seq with time-series design, biological replicates and DEGs) reproduce
  the temporal expression profiles specified in the simulation settings.
when_to_use_negative:
- Clustering simulated single-cell data across cell types — MOSim's methods use Spearman
  distance with K-medoids (partition around medoids) for the aggregated per-cell-type
  single-cell profiles, not K-means.
- Evaluating simulated regulator-gene regulatory relationships — Pearson's correlation
  against random no-effect pairs is the appropriate assessment for that question,
  not temporal clustering.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: MOSim
  role: R Bioconductor package whose bulk mosim() simulation (negative binomial-based,
    seeded with STATegra data) produces the temporal count profiles that K-means clustering
    validates
  repo: https://github.com/ConesaLab/MOSim
- name: R
  role: Environment in which the log(x + 1) transformation, K-means clustering, PCA
    and Silhouette-based cluster number selection are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/421834
    title: 'MOSim: bulk and single-cell multi-layer regulatory network simulator'
  source_content_hash: sha256:0a7bee3fb9bdbba37b5df84d46a1a3f77b60d805f52ba4394342fb491c0455f2
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# k-means-clustering-of-temporal-expression-profiles

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A validation procedure in which K-means clustering is applied to log(x + 1)-transformed simulated count matrices (e.g. MOSim bulk RNA-seq output seeded from STATegra mouse data) to verify that temporal expression patterns and experimental-design groupings were correctly simulated.

## When to use

Use when you need to validate that simulated multi-omics count data (e.g. MOSim bulk RNA-seq with time-series design, biological replicates and DEGs) reproduce the temporal expression profiles specified in the simulation settings.

## When NOT to use

- Clustering simulated single-cell data across cell types — MOSim's methods use Spearman distance with K-medoids (partition around medoids) for the aggregated per-cell-type single-cell profiles, not K-means.
- Evaluating simulated regulator-gene regulatory relationships — Pearson's correlation against random no-effect pairs is the appropriate assessment for that question, not temporal clustering.

## Inputs

- Simulated bulk count matrix (e.g. MOSim mosim() RNA-seq output, negative binomial-based)
- Simulation design specification (5 time points, 2 experimental groups, 3 biological replicates, 15% DEGs)

## Outputs

- Cluster assignment of simulated gene feature profiles (k-means clusters)
- Optimal cluster number selected via Silhouette's coefficient and intra-cluster variability
- Fraction of profiles assigned to incorrect clusters (validation metric, e.g. contributed to mosim_bulk_assessment_metrics.csv)

## How to apply

Apply a log(x + 1) transformation to the simulated count data as specified in the MOSim methods, then run K-means clustering on the simulated gene feature profiles (k=7 was used for the STATegra five-time-point, two-group, three-replicate bulk simulation). Select the optimal number of clusters by combining maximisation of Silhouette's coefficient with minimisation of intra-cluster variability. Run PCA on the transformed data to confirm that the clustering aligns with the simulation settings, and record the fraction of profiles assigned to incorrect clusters as the key validation metric. The simulation is accepted when fewer than 0.5% of profiles fall into incorrect clusters, indicating that temporal co-expression patterns induced by the simulator (negative binomial-based bulk model) were preserved.

## Related tools

- **MOSim** (R Bioconductor package whose bulk mosim() simulation (negative binomial-based, seeded with STATegra data) produces the temporal count profiles that K-means clustering validates) — https://github.com/ConesaLab/MOSim
- **R** (Environment in which the log(x + 1) transformation, K-means clustering, PCA and Silhouette-based cluster number selection are executed)

## Examples

```
sim_clust <- kmeans(log(simulated_bulk_counts + 1), centers = 7)
```

## Evaluation signals

- Fewer than 0.5% of simulated profiles are assigned to an incorrect temporal cluster (acceptance threshold for the bulk simulation)
- The number of clusters is chosen by maximising Silhouette's coefficient while minimising intra-cluster variability, not fixed arbitrarily
- PCA on the log(x + 1)-transformed data shows cluster separation consistent with the simulation settings (5 time points, 2 experimental groups)
- Cluster count matches the simulation design (k=7 used for the STATegra-seeded five-time-point profile simulation)
- Assessment metrics compiled into mosim_bulk_assessment_metrics.csv for reproducibility

## Limitations

- Validation result is specific to the simulation configuration (STATegra seed data, 5 time points, 2 groups, 3 replicates, 15% DEGs); other designs may need re-tuned k
- MOSim simulates at the count matrix level only (no read-level data), so clustering validates count-matrix profiles rather than raw sequencing output
- Optimal cluster number depends on the combined maximisation of Silhouette's coefficient and minimisation of intra-cluster variability; a misspecified k can distort the incorrect-assignment fraction

## Evidence

- [methods] the optimal number of clusters was obtained by combining the maximisation of Silhouette's coefficient and minimising the intra-cluster variability.: [evidence span withheld — anchor status: not_found]
