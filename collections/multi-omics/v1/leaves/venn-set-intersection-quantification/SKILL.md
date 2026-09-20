---
name: venn-set-intersection-quantification
description: Use when two RBP target lists exist — one from MAPIT-seq calltargets
  output filtered with default thresholds (fold enrichment > 2, MAPIT score > 0.5,
  adjusted P < 0.
when_to_use_negative:
- The pipeline has not yet produced a thresholded calltargets output — raw FASTQ or
  editing-site tables are not target lists and cannot be intersected meaningfully
- The comparison should be at editing-site or binding-region resolution rather than
  target-gene level; set intersection of gene symbols discards site-level information
- The orthogonal target list is from a different RBP or cell type than the MAPIT-seq
  experiment — the overlap is then not a validation of target recovery
tools:
- name: GATK
  role: Used in the upstream callediting step to produce the GATK-based RNA editing
    detection statistics from which the MAPIT target gene list is filtered before
    intersection
- name: MAPIT-seq pipeline (Mapit callediting/calltargets)
  role: Produces the differential editing statistics table (e.g., RBP_Edit_GE_RPE_DATA.tsv
    under 6-Edit_calling/) and calls RBP binding targets that form one side of the
    set intersection
  repo: https://github.com/WangLabPKU/MAPIT-seq
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:904024ea10e2ef6bf7f84c8cb24830d33d987c07afc98f45aaf19e51720c97a2
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
  - MAPIT-seq pipeline (Mapit callediting/calltargets)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# venn-set-intersection-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the overlap between a thresholded MAPIT-seq RBP target gene list and an orthogonal RBP target list (e.g., G3BP1 PAR-CLIP in HEK293T) via set intersection and Venn rendering, validating that the editing-based method recovers known binding targets.

## When to use

Use when two RBP target lists exist — one from MAPIT-seq calltargets output filtered with default thresholds (fold enrichment > 2, MAPIT score > 0.5, adjusted P < 0.05) and one from an orthogonal assay like PAR-CLIP — and the goal is to compute and report their gene-level intersection and significance.

## When NOT to use

- The pipeline has not yet produced a thresholded calltargets output — raw FASTQ or editing-site tables are not target lists and cannot be intersected meaningfully
- The comparison should be at editing-site or binding-region resolution rather than target-gene level; set intersection of gene symbols discards site-level information
- The orthogonal target list is from a different RBP or cell type than the MAPIT-seq experiment — the overlap is then not a validation of target recovery

## Inputs

- Thresholded MAPIT-seq target gene list derived from the differential editing statistics table (e.g., RBP_Edit_GE_RPE_DATA.tsv under 6-Edit_calling/) with fold enrichment > 2, MAPIT score > 0.5, adjusted P < 0.05
- Published PAR-CLIP target gene list for the same RBP and cell type (e.g., G3BP1 PAR-CLIP targets in HEK293T cells)
- Supplementary Table 2 overlap counts for significance comparison

## Outputs

- MAPIT ∩ PAR-CLIP intersection gene count (e.g., 4,769 for G3BP1 in HEK293T)
- Venn diagram quantifying the set overlap (Fig. 2b,c style)
- Overlap significance assessment referenced against Supplementary Table 2

## How to apply

First generate the MAPIT-seq target list by running the Mapit callediting (GATK-based RNA editing detection) and calltargets (differential editing analysis) steps, then filter to genes passing fold enrichment > 2, MAPIT score > 0.5, and adjusted P < 0.05, retaining only editing sites passing the Poisson significance filter (maximum p-value 0.05) and minimum coverage of 10. Load the published orthogonal target list (e.g., the G3BP1 PAR-CLIP targets in HEK293T cells) as a second gene set. Compute the set intersection (MAPIT ∩ PAR-CLIP) programmatically and quantify it; for the G3BP1 HEK293T comparison the reported intersection is 4,769 target genes detected by both methods. Render the overlap as a Venn diagram (Fig. 2b,c) and, if testing significance, compare the intersection against Supplementary Table 2; additionally check whether MAPIT-seq preferentially recovers the higher-confidence PAR-CLIP targets.

## Related tools

- **GATK** (Used in the upstream callediting step to produce the GATK-based RNA editing detection statistics from which the MAPIT target gene list is filtered before intersection)
- **MAPIT-seq pipeline (Mapit callediting/calltargets)** (Produces the differential editing statistics table (e.g., RBP_Edit_GE_RPE_DATA.tsv under 6-Edit_calling/) and calls RBP binding targets that form one side of the set intersection) — https://github.com/WangLabPKU/MAPIT-seq

## Examples

```
mapit = set(df[(df.fe>2)&(df.score>0.5)&(df.padj<0.05)].gene); parclip = set(pd.read_csv('G3BP1_PAR-CLIP_targets.tsv').gene); print(len(mapit & parclip))
```

## Evaluation signals

- The computed MAPIT ∩ PAR-CLIP intersection for G3BP1 MAPIT-seq in HEK293T cells equals 4,769 target genes, matching the reported finding
- The overlap reaches significance as reported in Fig. 2b,c and agrees with Supplementary Table 2
- The MAPIT target list was thresholded with fold enrichment > 2, MAPIT score > 0.5, and adjusted P < 0.05 before intersecting — applying raw (unthresholded) lists should change the count
- Upstream editing-site filters were applied (Poisson filter maximum p-value 0.05, minimum coverage 10) so only valid sites contributed to target calling
- MAPIT-seq preferentially recovers the higher-confidence PAR-CLIP targets, i.e., overlap enrichment should be higher in the high-confidence PAR-CLIP subset

## Limitations

- The reported intersection depends on the default target-calling thresholds (fold enrichment > 2, MAPIT score > 0.5, adjusted P < 0.05); different thresholds will yield different overlap counts
- Valid editing sites require minimum coverage of 10 and passage of the Poisson significance filter; low-coverage targets will be missed and shrink the intersection
- The workflow is specific to a matched RBP/cell-type pair (G3BP1 MAPIT-seq vs G3BP1 PAR-CLIP in HEK293T); cross-condition overlaps are not validated by this analysis

## Evidence

- [results] G3BP1 MAPIT-seq targets significantly overlap with G3BP1 PAR-CLIP targets in HEK293T cells, with 4,769 target genes detected by both methods (MAPIT ∩ PAR-CLIP): [evidence span withheld — anchor status: not_found]
- [methods] load PAR-CLIP list and compute intersection: [evidence span withheld — anchor status: not_found]
- [methods] default target thresholds: [evidence span withheld — anchor status: not_found]
- [readme] MAPIT-seq target identification: [evidence span withheld — anchor status: not_found]
