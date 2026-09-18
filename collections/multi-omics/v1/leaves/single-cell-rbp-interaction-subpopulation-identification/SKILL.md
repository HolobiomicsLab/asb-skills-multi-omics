---
name: single-cell-rbp-interaction-subpopulation-identification
description: Use when you have a single-cell cell×gene MAPIT-score matrix plus co-profiled
  transcriptomes and need to detect cell stage- or subpopulation-specific RBP-binding
  preferences and correlate binding strength with target expression.
when_to_use_negative:
- Bulk MAPIT-seq or tissue-section data without single-cell resolution — subpopulation-specific
  binding preferences cannot be resolved
- Input is already a cell-cluster assignment table or precomputed correlation table
  — the clustering/correlation step would be redundant
- Data lacking co-profiled transcriptome information, since phase assignment and score-vs-expression
  correlation depend on simultaneous RNA measurements
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: MAPIT-seq
  role: Source of the scMAPIT-seq cell×gene MAPIT interaction-score matrix and co-profiled
    transcriptome used for clustering and correlation
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: Seurat
  role: R framework used to build the Seurat object, run CellCycleScoring with G1/S
    and G2/M gene sets, and cluster cells on MAPIT interaction scores
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:8f794d4109051b4fe52881950d11f057a698b53e14a36b44d64736c0a8644f28
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
  - MAPIT-seq
  - Seurat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# single-cell-rbp-interaction-subpopulation-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies RBP-binding preference subpopulations in scMAPIT-seq data by clustering cells on their cell×gene MAPIT interaction-score matrix and integrating the co-profiled single-cell transcriptome (e.g., Seurat CellCycleScoring to assign G1/S/G2M phases), then correlating MAPIT scores with gene expression within each subpopulation.

## When to use

Use when you have a single-cell cell×gene MAPIT-score matrix plus co-profiled transcriptomes and need to detect cell stage- or subpopulation-specific RBP-binding preferences and correlate binding strength with target expression.

## When NOT to use

- Bulk MAPIT-seq or tissue-section data without single-cell resolution — subpopulation-specific binding preferences cannot be resolved
- Input is already a cell-cluster assignment table or precomputed correlation table — the clustering/correlation step would be redundant
- Data lacking co-profiled transcriptome information, since phase assignment and score-vs-expression correlation depend on simultaneous RNA measurements

## Inputs

- 10x-formatted single-cell cell×gene MAPIT interaction-score matrix (e.g., Supplementary Table 28)
- Co-profiled single-cell transcriptome count matrix from the same cells
- Canonical human G1/S and G2/M gene sets for Seurat CellCycleScoring

## Outputs

- Per-cell cell-cycle phase assignments (G1, S, G2/M)
- Differential-interaction cluster membership (e.g., G2/M-enriched cluster with 916/1,499 cells, 61.1%)
- Lists of genes with significant positive (141) and negative (94) MAPIT-score-vs-expression Pearson correlations
- A single results table combining phase assignments, cluster membership, and correlated-gene lists

## How to apply

Load the 10x cell×gene MAPIT-score matrix (e.g., Supplementary Table 28) into a Seurat object and run Seurat CellCycleScoring with the canonical human G1/S and G2/M gene sets to assign each cell (e.g., 1,499 cells) a G1, S, or G2/M phase from the co-profiled transcriptome. Cluster cells on MAPIT interaction scores with differential-interaction testing, and identify the cluster enriched for a given phase (e.g., the G2/M-peak cluster containing 916 of 1,499 cells, 61.1%). Within that cluster, compute the Pearson correlation between each gene's MAPIT interaction score and its expression, retaining genes with significant positive and negative correlations (expected: 141 positive, 94 negative). The rationale is that integrating single-cell transcriptome information with interaction scores reveals RBP-binding preferences that differ across distinct cell subpopulations, which bulk MAPIT-seq cannot resolve.

## Related tools

- **MAPIT-seq** (Source of the scMAPIT-seq cell×gene MAPIT interaction-score matrix and co-profiled transcriptome used for clustering and correlation) — https://github.com/WangLabPKU/MAPIT-seq
- **Seurat** (R framework used to build the Seurat object, run CellCycleScoring with G1/S and G2/M gene sets, and cluster cells on MAPIT interaction scores)

## Examples

```
seu <- CellCycleScoring(seu, s.features = s.genes, g2m.features = g2m.genes); Idents(seu) <- 'seurat_clusters'; cor.test(mapit_score[gene, g2m_cells], expr[gene, g2m_cells], method = 'pearson')
```

## Evaluation signals

- The G2/M-enriched differential-interaction cluster contains exactly 916 of 1,499 cells (61.1%)
- Correlation analysis recovers 141 genes with significant positive and 94 with significant negative MAPIT-score/expression correlations
- Every cell receives exactly one of three phase labels (G1, S, G2/M) from CellCycleScoring
- Per-cell phase assignments, cluster membership, and both correlated-gene lists are all present in the single output results table
- Cluster-enrichment composition matches the reported phase distribution when cross-tabulating clusters against CellCycleScoring phases

## Limitations

- Application of scMAPIT-seq to tissue samples is not yet demonstrated and is deferred to future work
- The correlation analysis depends on the specific cell-cycle gene sets and clustering resolution; reproducing exact gene counts (141/94) may be sensitive to these choices

## Evidence
