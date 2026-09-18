---
name: pearson-correlation-expression-interaction-analysis
description: Use when a cell×gene interaction-score matrix (e.g., MAPIT scores) is
  co-profiled with single-cell transcriptome data and you need per-gene correlation
  of binding strength with expression across subpopulations defined by cell cycle
  or clustering.
when_to_use_negative:
- Input is bulk sequencing data without single-cell transcriptome co-profiling — the
  phase assignment and per-cell correlation require single-cell resolution
- You need the upstream MAPIT-seq pipeline (editing detection, calltargets, motif
  discovery) rather than downstream analysis of an existing MAPIT-score matrix
- Cells have not been barcoded with RBP-targeted editing so no MAPIT interaction scores
  exist to correlate against expression
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3307
tools:
- name: Seurat
  role: CellCycleScoring with G1/S and G2/M gene sets to assign per-cell phase; clustering
    on MAPIT interaction scores with differential-interaction testing
- name: MAPIT-seq pipeline
  role: upstream producer of the cell×gene MAPIT-score matrix and co-profiled transcriptome
    used as correlation input
  repo: https://github.com/WangLabPKU/MAPIT-seq
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:b86ab67a140f5c7fbed650bd5d7cb3eb55a3569f1a8b559fa32c281aec604696
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# pearson-correlation-expression-interaction-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-gene Pearson correlations between MAPIT interaction scores (RBP-binding strength) and co-profiled RNA expression in single cells, after assigning cells to cell-cycle phases, to identify targets whose expression is positively or negatively coupled to RBP binding (141 positive / 94 negative in the G2/M-peak cluster).

## When to use

Use when a cell×gene interaction-score matrix (e.g., MAPIT scores) is co-profiled with single-cell transcriptome data and you need per-gene correlation of binding strength with expression across subpopulations defined by cell cycle or clustering.

## When NOT to use

- Input is bulk sequencing data without single-cell transcriptome co-profiling — the phase assignment and per-cell correlation require single-cell resolution
- You need the upstream MAPIT-seq pipeline (editing detection, calltargets, motif discovery) rather than downstream analysis of an existing MAPIT-score matrix
- Cells have not been barcoded with RBP-targeted editing so no MAPIT interaction scores exist to correlate against expression

## Inputs

- cell×gene MAPIT interaction-score matrix (10x-format, e.g., Supplementary Table 28)
- co-profiled single-cell transcriptome count matrix
- canonical human G1/S and G2/M gene sets for CellCycleScoring

## Outputs

- per-cell cell-cycle phase assignments (G1/S/G2/M) for 1,499 cells
- G2/M-enriched differential-interaction cluster membership (916/1,499 cells, 61.1%)
- list of 141 genes with significant positive score-expression correlations
- list of 94 genes with significant negative score-expression correlations
- single consolidated results table

## How to apply

Load the 10x scMAPIT-seq cell×gene MAPIT-score matrix (e.g., Supplementary Table 28) into a Seurat object, then run Seurat CellCycleScoring with canonical human G1/S and G2/M gene sets to assign each of the 1,499 cells a G1, S, or G2/M phase from the co-profiled transcriptome. Cluster cells on MAPIT interaction scores with differential-interaction testing and select the cluster enriched for G2/M-phase cells (expected: 916/1,499 cells, 61.1%). Within that G2/M-peak cluster, compute the Pearson correlation between each gene's MAPIT interaction score and its expression, and retain genes whose correlations pass the significance threshold, yielding 141 positively and 94 negatively correlated genes. Writing per-cell phase assignments, cluster membership, and both gene lists to a single results table enables direct validation against the reported counts.

## Related tools

- **Seurat** (CellCycleScoring with G1/S and G2/M gene sets to assign per-cell phase; clustering on MAPIT interaction scores with differential-interaction testing)
- **MAPIT-seq pipeline** (upstream producer of the cell×gene MAPIT-score matrix and co-profiled transcriptome used as correlation input) — https://github.com/WangLabPKU/MAPIT-seq

## Examples

```
cor <- cor.test(as.numeric(expr[gene, g2m_cells]), as.numeric(mapit[gene, g2m_cells]), method = "pearson")
```

## Evaluation signals

- CellCycleScoring assigns all 1,499 cells a phase and the G2/M-enriched differential-interaction cluster contains exactly 916/1,499 cells (61.1%)
- Correlation screen recovers 141 genes with significant positive and 94 genes with significant negative score-expression correlations
- Per-cell phase assignments are consistent with the co-profiled transcriptome (G2/M gene-set scores peak in the assigned G2/M cluster)
- Results table contains phase assignment, cluster membership, and both gene lists for every cell/gene without missing values

## Limitations

- Correlation does not imply direct binding-mediated regulation; MAPIT scores reflect binding strength, not causal effects on expression
- The G2/M-peak cluster dominates the analysis (61.1% of cells), so correlations in other phases are underpowered
- scMAPIT-seq application to tissue samples is not yet demonstrated and is deferred to future work

## Evidence
