---
name: mapit-score-matrix-clustering
description: Use when you have a single-cell MAPIT-score matrix (e.g., Supplementary
  Table 28, 1,499 cells × genes) plus co-profiled transcriptome and need to find cell
  subpopulations with distinct RBP-binding preferences or correlate binding strength
  with target RNA abundance across cell-cycle phases.
when_to_use_negative:
- Input is bulk MAPIT-seq data without single-cell resolution — the workflow depends
  on per-cell phase assignment and per-cell clustering; use bulk calltargets differential
  editing analysis instead
- The RBP of interest is expected to act uniformly across cell states — clustering
  on interaction scores is designed to reveal cell stage-specific RBP regulation,
  not global binding signatures
- You lack the co-profiled transcriptome; CellCycleScoring and score-expression Pearson
  correlations both require paired RNA abundance measurements from the same cells
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: Seurat
  role: R framework used to load the 10x MAPIT-score matrix, run CellCycleScoring
    with G1/S and G2/M gene sets, and cluster cells on interaction scores
- name: MAPIT-seq pipeline
  role: Upstream pipeline producing the MAPIT interaction scores and co-profiled transcriptome
    that feed the cell×gene score matrix
  repo: https://github.com/WangLabPKU/MAPIT-seq
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:ce3c1a9a8316469b7a0354bec780e94d62ae37c0bf3efa4c8d8ad9c50b911245
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WangLabPKU/MAPIT-seq
  repo_url: https://github.com/WangLabPKU/MAPIT-seq
  tools:
  - MAPIT-seq pipeline
  - Seurat
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# mapit-score-matrix-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cluster single cells on a cell×gene MAPIT interaction-score matrix (differential-interaction testing) and integrate co-profiled transcriptome phase scoring (Seurat CellCycleScoring) to identify cell-stage-specific RBP regulation, as demonstrated for G3BP1 in 1,499 scMAPIT-seq cells where a G2/M-enriched cluster contained 916/1,499 cells (61.1%).

## When to use

Use when you have a single-cell MAPIT-score matrix (e.g., Supplementary Table 28, 1,499 cells × genes) plus co-profiled transcriptome and need to find cell subpopulations with distinct RBP-binding preferences or correlate binding strength with target RNA abundance across cell-cycle phases.

## When NOT to use

- Input is bulk MAPIT-seq data without single-cell resolution — the workflow depends on per-cell phase assignment and per-cell clustering; use bulk calltargets differential editing analysis instead
- The RBP of interest is expected to act uniformly across cell states — clustering on interaction scores is designed to reveal cell stage-specific RBP regulation, not global binding signatures
- You lack the co-profiled transcriptome; CellCycleScoring and score-expression Pearson correlations both require paired RNA abundance measurements from the same cells

## Inputs

- 10x-format cell×gene MAPIT interaction-score matrix (e.g., Supplementary Table 28) from 1,499 scMAPIT-seq cells
- Co-profiled single-cell transcriptome (gene expression counts) from the same cells
- Canonical human G1/S and G2/M cell-cycle gene sets for Seurat CellCycleScoring

## Outputs

- Per-cell cell-cycle phase assignments (G1, S, or G2/M)
- G2/M-enriched cluster membership labels (916/1,499 cells, 61.1%)
- List of genes with significant positive MAPIT-score–expression correlations (141 expected)
- List of genes with significant negative MAPIT-score–expression correlations (94 expected)
- Consolidated results table combining phases, cluster membership, and correlated-gene lists

## How to apply

Load the 10x cell×gene MAPIT-score matrix (e.g., Supplementary Table 28) into a Seurat object, using the co-profiled single-cell transcriptome for parallel analyses. Run Seurat CellCycleScoring with canonical human G1/S and G2/M gene sets to assign each of the 1,499 cells a G1, S, or G2/M phase. Cluster cells on the MAPIT interaction scores with differential-interaction testing, and identify the cluster enriched for G2/M-phase cells — the paper reports 916 of 1,499 cells (61.1%) in this G2/M-peak cluster. Within the G2/M-peak cluster, compute the Pearson correlation between each gene's MAPIT interaction score and its expression, selecting significant positive (expected 141 genes) and negative (expected 94 genes) correlations. Write per-cell phase assignments, G2/M-cluster membership, and the two correlated-gene lists to a single results table for validation against expected counts.

## Related tools

- **Seurat** (R framework used to load the 10x MAPIT-score matrix, run CellCycleScoring with G1/S and G2/M gene sets, and cluster cells on interaction scores)
- **MAPIT-seq pipeline** (Upstream pipeline producing the MAPIT interaction scores and co-profiled transcriptome that feed the cell×gene score matrix) — https://github.com/WangLabPKU/MAPIT-seq

## Examples

```
seurat_obj <- CreateSeuratObject(counts = mapit_score_matrix); seurat_obj <- CellCycleScoring(seurat_obj, s.features = s_genes, g2m.features = g2m_genes)
```

## Evaluation signals

- G2/M-enriched differential-interaction cluster contains exactly 916 of 1,499 cells (61.1%)
- Correlation analysis within the G2/M-peak cluster recovers 141 significantly positive and 94 significantly negative MAPIT-score–expression correlated genes
- CellCycleScoring assigns every cell exactly one of G1, S, or G2/M phase (no missing or double assignments)
- All per-cell phases, cluster memberships, and both gene lists are written to a single results table with consistent cell identifiers
- Cluster-level differential-interaction testing shows the G2/M-peak cluster is statistically enriched for G2/M-phase cells

## Limitations

- scMAPIT-seq application to tissue samples is not yet demonstrated and is deferred to future work, so this clustering workflow is validated on isolated single cells only
- RBP-binding preference identification across subpopulations relies on integrating single-cell transcriptome information; without the paired transcriptome the phase-enriched clustering and correlations are not possible

## Evidence

- [results] scMAPIT-seq cell-cycle clustering workflow: "we develop high-throughput single-cell MAPIT-seq (scMAPIT-seq) to reveal cell stage-specific RBP regulation"
- [methods] Cluster enriched for G2/M with 916/1,499 cells: [evidence span withheld — anchor status: not_found]
- [methods] Load score matrix into Seurat: [evidence span withheld — anchor status: not_found]
