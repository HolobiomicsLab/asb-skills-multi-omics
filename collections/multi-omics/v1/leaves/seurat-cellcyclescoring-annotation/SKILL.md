---
name: seurat-cellcyclescoring-annotation
description: Use when a single-cell experiment co-profiles transcriptome and interaction
  scores (e.g., scMAPIT-seq cell×gene MAPIT matrix) and cells must be staged by cell-cycle
  phase before phase-specific correlation or clustering.
when_to_use_negative:
- The dataset is bulk RNA-seq or lacks co-profiled single-cell transcriptomes, so
  per-cell phase scoring is not possible
- The goal is to detect RBP targets from RNA editing events (calltargets/hc_cluster
  steps), not to relate binding scores to expression across phases
- Cells have already been assigned cell-cycle phases or the analysis does not require
  phase stratification
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Provides CellCycleScoring to assign G1/S/G2/M phases and clustering/differential-interaction
    testing on the MAPIT-score matrix
- name: MAPIT-seq pipeline
  role: Generates the scMAPIT-seq cell×gene interaction-score matrix that is the input
    to the Seurat-based phase scoring and correlation analysis
  repo: https://github.com/WangLabPKU/MAPIT-seq
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:aa995b4909ca6120b73f9dfefa51c141b2e75f241c7c446f70e223913f1a4016
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Seurat CellCycleScoring annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns each single cell a G1, S, or G2/M phase by scoring canonical human G1/S and G2/M marker gene sets against the co-profiled single-cell transcriptome, enabling downstream differential-interaction analysis of RBP binding (e.g., G3BP1 MAPIT scores) across cell-cycle phases.

## When to use

Use when a single-cell experiment co-profiles transcriptome and interaction scores (e.g., scMAPIT-seq cell×gene MAPIT matrix) and cells must be staged by cell-cycle phase before phase-specific correlation or clustering.

## When NOT to use

- The dataset is bulk RNA-seq or lacks co-profiled single-cell transcriptomes, so per-cell phase scoring is not possible
- The goal is to detect RBP targets from RNA editing events (calltargets/hc_cluster steps), not to relate binding scores to expression across phases
- Cells have already been assigned cell-cycle phases or the analysis does not require phase stratification

## Inputs

- Cell×gene MAPIT interaction-score matrix (Supplementary Table 28) for 1,499 single cells
- Co-profiled single-cell transcriptome (10x-style expression matrix)
- Canonical human G1/S and G2/M gene sets (e.g., Tirosh et al. lists used by Seurat CellCycleScoring)

## Outputs

- Per-cell cell-cycle phase assignments (G1, S, or G2/M)
- G2/M-enriched differential-interaction cluster membership (expected 916/1,499 cells, 61.1%)
- List of genes with significant positive MAPIT-score/expression Pearson correlation (expected 141 genes)
- List of genes with significant negative MAPIT-score/expression Pearson correlation (expected 94 genes)
- Single results table combining phase assignments, cluster membership, and both correlated-gene lists

## How to apply

Load the cell×gene MAPIT-score matrix (e.g., Supplementary Table 28 with 1,499 cells) into a Seurat object, then run CellCycleScoring with the canonical human G1/S and G2/M gene sets to assign each cell a phase from the co-profiled transcriptome. Next, cluster cells on MAPIT interaction scores with differential-interaction testing and identify the cluster enriched for G2/M-phase cells — the reported analysis recovered a G2/M-peak cluster containing 916 of 1,499 cells (61.1%). Within that cluster, compute Pearson correlations between each gene's MAPIT interaction score and its expression to extract significant positively correlated (141 genes) and negatively correlated (94 genes) targets. Rationale: phasing on transcriptome rather than interaction scores avoids confounding RBP-binding changes with cell-cycle composition when interpreting differential interactions.

## Related tools

- **Seurat** (Provides CellCycleScoring to assign G1/S/G2/M phases and clustering/differential-interaction testing on the MAPIT-score matrix)
- **MAPIT-seq pipeline** (Generates the scMAPIT-seq cell×gene interaction-score matrix that is the input to the Seurat-based phase scoring and correlation analysis) — https://github.com/WangLabPKU/MAPIT-seq

## Examples

```
seurat_obj <- CellCycleScoring(seurat_obj, s.features = s.genes, g2m.features = g2m.genes, set.ident = FALSE)
```

## Evaluation signals

- CellCycleScoring assigns exactly one phase (G1, S, or G2/M) to each of the 1,499 cells with no missing assignments
- Differential-interaction clustering on MAPIT scores yields a G2/M-enriched cluster containing 916/1,499 cells (61.1%), matching the reported composition
- Pearson correlation within the G2/M-peak cluster recovers 141 significantly positively and 94 significantly negatively correlated genes
- Results table contains per-cell phase, cluster membership, and both gene lists as columns
- Genes scored as correlated are RBP (G3BP1) targets present in the MAPIT-score matrix

## Limitations

- Cell stage-specific regulation is inferred from a single-cell dataset of 1,499 cells; phase assignments depend on transcriptome quality of the co-profiled data
- The G2/M-peak cluster and 141/94 correlation counts are dataset-specific expectations and will differ for other RBPs or cell numbers
- Pearson correlation captures only linear MAPIT-score–expression relationships within the selected G2/M cluster

## Evidence
