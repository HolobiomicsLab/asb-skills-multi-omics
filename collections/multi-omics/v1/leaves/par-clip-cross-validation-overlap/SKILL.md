---
name: par-clip-cross-validation-overlap
description: Use when you have a MAPIT-seq differential editing statistics table (e.g.,
  RBP_Edit_GE_RPE_DATA.tsv from the callediting/calltargets steps) and want to reproduce
  or validate the target list against an orthogonal RBP-binding dataset such as PAR-CLIP.
when_to_use_negative:
- You only have raw MAPIT-seq reads and have not yet run the mapping, callediting,
  and calltargets pipeline steps — no editing statistics table exists to threshold
- The comparison RBP or cell type has no published PAR-CLIP target list, so no orthogonal
  cross-validation is possible
- You need binding-site-resolution validation rather than gene-level target overlap
  — the skill works at gene-level set intersection
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GATK
  role: Used in the callediting step for GATK-based RNA editing detection, producing
    the editing statistics table that is thresholded for target calling
- name: MAPIT-seq pipeline
  role: Provides the calltargets (differential editing analysis) step whose output
    target list is intersected with PAR-CLIP targets
  repo: https://github.com/WangLabPKU/MAPIT-seq
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:11360c3ad69aeb9d1e212eeae2e86279e7b3b3c9a392f861b90df7fcf2dce4ab
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
  - GATK
  - MAPIT-seq pipeline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# par-clip-cross-validation-overlap

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-validates MAPIT-seq RBP target calls against published PAR-CLIP targets by thresholding the differential editing statistics table (fold enrichment > 2, MAPIT score > 0.5, adjusted P < 0.05, Poisson P < 0.05, coverage >= 10) and computing the set intersection, quantified as a Venn overlap.

## When to use

Use when you have a MAPIT-seq differential editing statistics table (e.g., RBP_Edit_GE_RPE_DATA.tsv from the callediting/calltargets steps) and want to reproduce or validate the target list against an orthogonal RBP-binding dataset such as PAR-CLIP.

## When NOT to use

- You only have raw MAPIT-seq reads and have not yet run the mapping, callediting, and calltargets pipeline steps — no editing statistics table exists to threshold
- The comparison RBP or cell type has no published PAR-CLIP target list, so no orthogonal cross-validation is possible
- You need binding-site-resolution validation rather than gene-level target overlap — the skill works at gene-level set intersection

## Inputs

- MAPIT-seq differential editing statistics table (e.g., RBP_Edit_GE_RPE_DATA.tsv under 6-Edit_calling/)
- Published PAR-CLIP target list for the same RBP and cell type (e.g., G3BP1 PAR-CLIP in HEK293T cells)
- Threshold parameters: fold enrichment > 2, MAPIT score > 0.5, adjusted P < 0.05, Poisson p-value <= 0.05, minimum coverage 10

## Outputs

- Thresholded MAPIT-seq target gene list
- Set intersection (overlap) between MAPIT-seq and PAR-CLIP target genes
- Venn overlap figure/counts comparable to Fig. 2b,c and Supplementary Table 2

## How to apply

Load the GATK-based differential editing statistics table produced by the MAPIT-seq pipeline's callediting and calltargets steps, then filter candidate targets with the default thresholds: editing fold enrichment > 2, MAPIT score > 0.5, and adjusted P < 0.05. Additionally retain only editing sites passing the Poisson significance filter (maximum p-value 0.05 per the example calltargets command) and the minimum coverage of 10 for valid editing sites. Load the published PAR-CLIP target list for the same RBP and cell type and compute the set intersection at the gene level. Quantify the MAPIT ∩ PAR-CLIP overlap and compare it against the published counts (e.g., 4,769 overlapping target genes for G3BP1 in HEK293T cells, Fig. 2b,c and Supplementary Table 2). Significance of the overlap (hypergeometric/permutation test) and preferential recovery of higher-confidence PAR-CLIP targets indicate correct application.

## Related tools

- **GATK** (Used in the callediting step for GATK-based RNA editing detection, producing the editing statistics table that is thresholded for target calling)
- **MAPIT-seq pipeline** (Provides the calltargets (differential editing analysis) step whose output target list is intersected with PAR-CLIP targets) — https://github.com/WangLabPKU/MAPIT-seq

## Examples

```
mapit calltargets -p 0.05 -c 10
```

## Evaluation signals

- Filtered target list respects all three default thresholds (fold enrichment > 2, MAPIT score > 0.5, adjusted P < 0.05) plus Poisson p <= 0.05 and coverage >= 10
- MAPIT ∩ PAR-CLIP intersection count reproduces the published value (4,769 target genes for G3BP1 in HEK293T cells per Supplementary Table 2)
- Overlap significance matches the reported significant enrichment in Fig. 2b,c
- Preferential recovery of higher-confidence PAR-CLIP targets by MAPIT-seq is observed in the stratified comparison
- Only editing sites with valid coverage and Poisson-filter significance enter the target intersection

## Limitations

- Results depend on the example calltargets Poisson p-value and coverage settings; the pipeline default Poisson p-value is 0.1 while the G3BP1 reproduction used 0.05, so deviating settings will change overlap counts
- Overlap is a gene-level set intersection; site-level or region-level validation requires high-confidence edit cluster identification (hc_cluster) rather than this skill
- Cross-validation is only possible where an orthogonal binding dataset (e.g., PAR-CLIP) exists for the same RBP and cell type

## Evidence

- [results] Using the default thresholds, G3BP1 MAPIT-seq targets significantly overlap with G3BP1 PAR-CLIP targets in HEK293T cells: [evidence span withheld — anchor status: not_found]
