---
name: adjusted-rand-index-evaluation
description: Use when you have simulated single-cell data with known cell-type labels
  and a clustering result that must be scored for label recovery, e.g. benchmarking
  scDHA on sc_mosim output.
when_to_use_negative:
- Input has no ground-truth labels (e.g., real experimental scRNA-seq without simulated
  cell types) — ARI against truth is undefined; use internal validation instead.
- The task is to assess simulation fidelity of MOSim itself (bulk K-means/K-medoids
  with Silhouette coefficient), not clustering-method benchmarking against known labels.
- Cluster counts or label spaces differ fundamentally between partitions and the goal
  is cluster matching rather than chance-corrected agreement scoring.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: MOSim
  role: Generates the simulated scRNA-seq count matrix (via sc_mosim, leveraging SPARSim)
    that supplies the ground-truth cell-type labels used in the ARI comparison
  repo: https://github.com/ConesaLab/MOSim
- name: R
  role: Implementation environment for the simulation, clustering, and ARI computation
    workflow
- name: SPARSim
  role: Internal single-cell count simulator used by sc_mosim to produce the count
    matrix whose clustering is evaluated by ARI
- name: Seurat
  role: Provides the human pbmc.multiome 10x Genomics seed dataset (SeuratData package)
    used to seed the simulation being benchmarked
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/421834
    title: 'MOSim: bulk and single-cell multi-layer regulatory network simulator'
  source_content_hash: sha256:f19f35487f94183d3bdab6803f6fe58ecdda306ba7d01b1397359f9851e4db1f
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# adjusted-rand-index-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute the Adjusted Rand Index (ARI) to quantitatively score how well a clustering method (e.g., VAE-based scDHA) recovers simulated ground-truth cell-type labels from sc_mosim-generated scRNA-seq data, alongside a per-cluster cell-type recovery table.

## When to use

Use when you have simulated single-cell data with known cell-type labels and a clustering result that must be scored for label recovery, e.g. benchmarking scDHA on sc_mosim output.

## When NOT to use

- Input has no ground-truth labels (e.g., real experimental scRNA-seq without simulated cell types) — ARI against truth is undefined; use internal validation instead.
- The task is to assess simulation fidelity of MOSim itself (bulk K-means/K-medoids with Silhouette coefficient), not clustering-method benchmarking against known labels.
- Cluster counts or label spaces differ fundamentally between partitions and the goal is cluster matching rather than chance-corrected agreement scoring.

## Inputs

- Vector of predicted cluster assignments per cell (e.g., from scDHA clustering of log(x + 1)-transformed simulated counts)
- Ground-truth simulated cell-type labels from the sc_mosim simulation (six cell types incl. cDC and Treg in the pbmc.multiome-seeded simulation)

## Outputs

- Adjusted Rand Index (ARI) score (expected ~0.949 for the scDHA benchmark)
- Tabulation of recovered vs. merged cell types (e.g., one cluster combining cDC and Treg cells)
- clustering_metrics.csv with the recorded metrics

## How to apply

After clustering the simulated data (e.g., scDHA applied to log(x + 1)-transformed counts from sc_mosim/SPARSim), compare the predicted cluster assignments against the ground-truth simulated cell-type labels using the Adjusted Rand Index, which corrects for chance agreement between partitions. In the scDHA benchmark on pbmc.multiome-seeded sc_mosim data, an ARI of 0.949 was achieved when scDHA identified five of six simulated cell types, with one cluster combining cDC and Treg cells. Tabulate which simulated cell types are recovered and which are merged to interpret sub-ARI signal: a high ARI with merged rare populations indicates the method separates most types but confounds closely related ones. Write the numeric metrics to a file such as clustering_metrics.csv for reproducibility.

## Related tools

- **MOSim** (Generates the simulated scRNA-seq count matrix (via sc_mosim, leveraging SPARSim) that supplies the ground-truth cell-type labels used in the ARI comparison) — https://github.com/ConesaLab/MOSim
- **R** (Implementation environment for the simulation, clustering, and ARI computation workflow)
- **SPARSim** (Internal single-cell count simulator used by sc_mosim to produce the count matrix whose clustering is evaluated by ARI)
- **Seurat** (Provides the human pbmc.multiome 10x Genomics seed dataset (SeuratData package) used to seed the simulation being benchmarked)

## Examples

```
Rscript -e 'ari <- mclust::adjustedRandIndex(scDHA_clusters, sim_cell_types); write.csv(data.frame(ARI=ari), "clustering_metrics.csv")'
```

## Evaluation signals

- ARI reaches the expected value of 0.949 for the scDHA benchmark on sc_mosim-simulated pbmc.multiome-seeded data
- Five of six simulated cell types are identified as distinct clusters
- Exactly one cluster is documented as combining cDC and Treg cells (the known confusion mode)
- Metrics are written to clustering_metrics.csv with the ARI and recovery tabulation
- ARI is computed on log(x + 1)-transformed counts, matching the preprocessing applied before clustering

## Limitations

- ARI is chance-corrected but insensitive to which specific labels are merged — a high ARI (0.949) can still hide that closely related populations (cDC and Treg) are combined into one cluster.
- Ground truth only exists because data are simulated; sc_mosim single-cell simulation is restricted to scRNA-seq and scATAC-seq, so ARI-based benchmarking of other single-cell modalities is out of scope.
- The seed dataset (e.g., pbmc.multiome) and MOSim software version influence the simulated label structure, so ARI values are not directly transferable across seed datasets or versions.

## Evidence

- [results] scDHA clustering of the simulated scRNA-seq data identified five of six simulated cell types, with one cluster combining cDC and Treg cells, and achieved an Adjusted Rand Index (ARI) score of 0.949.: [evidence span withheld — anchor status: not_found]
- [methods] Compare the scDHA cluster assignments against the ground-truth simulated cell-type labels by computing the Adjusted Rand Index: [evidence span withheld — anchor status: not_found]
- [methods] Log transformation (log(x + 1)) of the data: "a log transformation (log(x + 1))[42] was applied to the data"
- [discussion] for single-cell data, it leverages the well-established simulator SPARSim: "while for single-cell data, it leverages the well-established simulator SPARSim[11]"
- [discussion] Currently, single-cell simulation is restricted to scRNA-seq and scATAC-seq, as these are presently the only two commercially available sequencing techniques: "Currently, single-cell simulation is restricted to scRNA-seq and scATAC-seq, as these are presently the only two commercially available sequencing techniques"
