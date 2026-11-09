---
name: single-cell-cell-cycle-phase-scoring
description: Use when you have a single-cell cell×gene matrix (e.g., MAPIT interaction
  scores or expression from scMAPIT-seq, such as 1,499 cells) plus matched transcriptome
  data and need to stratify cells by cell-cycle phase and find phase-enriched clusters.
when_to_use_negative:
- Data are bulk RNA-seq or bulk RBP-binding measurements without single-cell resolution
  — phase scoring requires per-cell transcriptomes
- The input matrix lacks a co-profiled transcriptome, so canonical G1/S and G2/M gene-set
  scoring cannot be computed
- The goal is bulk RBP target calling from editing sites (use the MAPIT-seq calltargets
  differential-editing workflow instead)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Seurat
  role: R toolkit used to load the cell×gene MAPIT-score matrix and run CellCycleScoring
    with canonical G1/S and G2/M gene sets to assign per-cell phases
- name: MAPIT-seq pipeline
  role: Generates the scMAPIT-seq cell×gene MAPIT interaction-score matrix (Supplementary
    Table 28) that supplies the interaction scores whose phase-specific correlations
    are tested
  repo: https://github.com/WangLabPKU/MAPIT-seq
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:93c9789339fdc894f2d48f5a0193b67937bcf64532132bb503c52412b8729c5e
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Single-cell cell-cycle phase scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns each single cell a G1, S, or G2/M phase from co-profiled single-cell transcriptomes using Seurat CellCycleScoring with canonical human G1/S and G2/M gene sets, then links phase-specific subpopulations to differences in RBP-binding strength measured as MAPIT interaction scores.

## When to use

Use when you have a single-cell cell×gene matrix (e.g., MAPIT interaction scores or expression from scMAPIT-seq, such as 1,499 cells) plus matched transcriptome data and need to stratify cells by cell-cycle phase and find phase-enriched clusters.

## When NOT to use

- Data are bulk RNA-seq or bulk RBP-binding measurements without single-cell resolution — phase scoring requires per-cell transcriptomes
- The input matrix lacks a co-profiled transcriptome, so canonical G1/S and G2/M gene-set scoring cannot be computed
- The goal is bulk RBP target calling from editing sites (use the MAPIT-seq calltargets differential-editing workflow instead)

## Inputs

- 10x single-cell cell×gene MAPIT interaction-score matrix (Supplementary Table 28), 1,499 cells
- Co-profiled single-cell transcriptome (gene expression) matrix from the same cells
- Canonical human G1/S and G2/M cell-cycle gene sets

## Outputs

- Per-cell cell-cycle phase assignments (G1, S, G2/M)
- G2/M-enriched differential-interaction cluster membership (916/1,499 cells, 61.1%)
- List of 141 genes with significant positive MAPIT-score vs expression Pearson correlations
- List of 94 genes with significant negative MAPIT-score vs expression Pearson correlations
- Single combined results table of phases, cluster membership, and correlated-gene lists

## How to apply

Load the 10x scMAPIT-seq cell×gene MAPIT-score matrix (Supplementary Table 28) into a Seurat object, then run Seurat CellCycleScoring with the canonical human G1/S and G2/M gene sets to assign each of the 1,499 cells a G1, S, or G2/M phase using the co-profiled single-cell transcriptome. Cluster cells on MAPIT interaction scores and run differential-interaction testing to identify the cluster enriched for G2/M-phase cells, confirming the reported composition of 916 of 1,499 cells (61.1%). Within the G2/M-peak cluster, compute the Pearson correlation between each gene's MAPIT interaction score and its expression to recover the expected 141 significantly positively and 94 significantly negatively correlated G3BP1 target genes. Write per-cell phase assignments, G2/M-cluster membership, and the two correlated-gene lists to a single results table.

## Related tools

- **Seurat** (R toolkit used to load the cell×gene MAPIT-score matrix and run CellCycleScoring with canonical G1/S and G2/M gene sets to assign per-cell phases)
- **MAPIT-seq pipeline** (Generates the scMAPIT-seq cell×gene MAPIT interaction-score matrix (Supplementary Table 28) that supplies the interaction scores whose phase-specific correlations are tested) — https://github.com/WangLabPKU/MAPIT-seq

## Examples

```
Rscript -e 'seu <- CreateSeuratObject(expr); seu <- CellCycleScoring(seu, s.features = s.genes, g2m.features = g2m.genes); Idents(seu) <- seu$Phase'
```

## Evaluation signals

- G2/M-enriched differential-interaction cluster contains exactly 916 of 1,499 cells (61.1%)
- Correlation analysis recovers 141 genes with significant positive and 94 with significant negative MAPIT-score vs expression correlations in the G2/M-peak cluster
- Every one of the 1,499 cells receives exactly one phase assignment (G1, S, or G2/M)
- Results table contains per-cell phase assignments, G2/M-cluster membership, and both correlated-gene lists in a single output

## Limitations

- scMAPIT-seq application to tissue samples is not yet demonstrated and is deferred to future work
- Phase scoring depends on the quality of the co-profiled single-cell transcriptome; weak expression capture will degrade G1/S and G2/M gene-set scores

## Evidence

- [supplementary] G2/M-peak cluster composition of 916/1,499 cells: [evidence span withheld — anchor status: not_found]
