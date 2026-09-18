---
name: differential-rna-editing-analysis
description: Use when you have per-sample RNA editing event tables (e.g. _Edit__DATA.tsv
  from Mapit callediting) for an RBP treatment and matched control, and need to identify
  which transcripts are significant RBP binding targets.
when_to_use_negative:
- Input is raw sequencing data or BAM files without a called editing event table —
  run the upstream callediting (GATK RNA editing detection) step first
- Samples lack a matched control; differential editing target calling requires treatment-vs-control
  comparison and would be invalid on unpaired single samples
- Goal is detecting individual RNA editing sites or clusters rather than calling RBP
  target transcripts — use callediting or hc_cluster (high-confidence edit cluster
  identification) instead
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MAPIT-seq pipeline
  role: Provides 'Mapit calltargets', the differential editing analysis step that
    calls RBP binding targets from editing event tables
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: GATK
  role: Underlying RNA editing detection engine for the callediting step whose output
    tables feed calltargets
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: conda
  role: Recommended installation route for all pipeline software including Mapit calltargets
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: SAILOR/FLARE
  role: Alternative/edit-site-finding branch of the workflow (prepare step prepares
    files for SAILOR; FLARE flags regions of RNA editing enrichment)
  repo: https://github.com/YeoLab/FLARE
- name: REDItools2
  role: RNA editing event detection tool cited alongside MAPIT-seq/GATK for editing
    analysis
  repo: https://github.com/BioinfoUNIBA/REDItools2
merged_aliases:
- differential-rna-editing-target-calling
merged_alias_records:
- alias: differential-rna-editing-target-calling
  slug: differential-rna-editing-target-calling
  cosine_score: 0.8573
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_002
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:c723990b5b45d5671a61e0747267500c34ff097175e8da56146315ca3e9d4f42
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
  - REDItools2
  - SAILOR/FLARE
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# differential-rna-editing-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Differential editing analysis to call RBP binding targets from MAPIT-seq RNA editing event tables: compare enzyme-expressing (treatment) vs control samples per transcript bin, using Poisson-filtered editing sites and coverage/fold-enrichment/MAPIT-score/adjusted-P thresholds to define target transcripts.

## When to use

Use when you have per-sample RNA editing event tables (e.g. *_Edit_*_DATA.tsv from Mapit callediting) for an RBP treatment and matched control, and need to identify which transcripts are significant RBP binding targets.

## When NOT to use

- Input is raw sequencing data or BAM files without a called editing event table — run the upstream callediting (GATK RNA editing detection) step first
- Samples lack a matched control; differential editing target calling requires treatment-vs-control comparison and would be invalid on unpaired single samples
- Goal is detecting individual RNA editing sites or clusters rather than calling RBP target transcripts — use callediting or hc_cluster (high-confidence edit cluster identification) instead

## Inputs

- Per-sample RNA editing event table from Mapit callediting (e.g. RBP_Edit_GE_RPE_DATA.tsv or *_Edit_*_DATA.tsv under 6-Edit_calling/)
- Treatment (RBP enzyme-expressing) and control sample identifiers for --treatName/--controlName
- Transcript annotation and genome build (e.g. GRCh38, binSize 50 transcript binning)
- Published RBP target lists for overlap validation (e.g. G3BP1 PAR-CLIP targets, Supplementary Tables 2–3)

## Outputs

- Per-RBP significant target gene/transcript list filtered at fold enrichment > 2, MAPIT score > 0.5, adjusted P < 0.05
- Adjusted P values per target (e.g. prc2_target_adjusted_pvalues.csv with XIST values for EED, EZH2, SUZ12)
- Target recovery comparison across editor configurations (e.g. target_recovery_comparison.csv)
- Venn overlap statistics of MAPIT targets vs PAR-CLIP targets (Fig. 2b,c)

## How to apply

Load the RNA editing event table produced by the GATK-based callediting step (e.g. RBP_Edit_GE_RPE_DATA.tsv under 6-Edit_calling) and run 'Mapit calltargets' for each RBP treatment versus its control, splitting transcripts into 50-nt bins (binSize 50) and requiring minimum editing-site coverage of 10 for valid editing sites. Apply the Poisson filter on editing sites (default maximum p-value 0.1, tightened to 0.05 in the example calltargets command) and set --treatName/--controlName to the enzyme-expressing vs control samples (e.g. dual-editor rAPOBEC1–pAG–hADAR2dd vs single-deaminase configurations, -v GRCh38). Filter candidate targets with the default thresholds: editing fold enrichment > 2, MAPIT score > 0.5, adjusted P < 0.05, computing adjusted P values at gene/transcript level. The rationale is that editing events introduced adjacent to binding sites by hADAR2dd (A-to-I) and rAPOBEC1 (C-to-U) are enriched on genuine RBP-bound transcripts, so between-condition differential editing statistics identify targets. Intersect called target transcripts with published target sets (e.g. PAR-CLIP targets, Supplementary Tables 2–3) to quantify and validate recovery rates.

## Related tools

- **MAPIT-seq pipeline** (Provides 'Mapit calltargets', the differential editing analysis step that calls RBP binding targets from editing event tables) — https://github.com/WangLabPKU/MAPIT-seq
- **GATK** (Underlying RNA editing detection engine for the callediting step whose output tables feed calltargets) — https://github.com/WangLabPKU/MAPIT-seq
- **conda** (Recommended installation route for all pipeline software including Mapit calltargets) — https://github.com/WangLabPKU/MAPIT-seq
- **SAILOR/FLARE** (Alternative/edit-site-finding branch of the workflow (prepare step prepares files for SAILOR; FLARE flags regions of RNA editing enrichment)) — https://github.com/YeoLab/FLARE
- **REDItools2** (RNA editing event detection tool cited alongside MAPIT-seq/GATK for editing analysis) — https://github.com/BioinfoUNIBA/REDItools2

## Examples

```
Mapit calltargets -v GRCh38 --treatName RBP --controlName control --binSize 50 --coverage 10 -p 0.05
```

## Evaluation signals

- Recomputed XIST adjusted P values match reported values within tolerance: 0.0463 (EED), 1.72 × 10⁻³ (EZH2), 9.94 × 10⁻⁶ (SUZ12)
- G3BP1 MAPIT-seq targets at default thresholds significantly overlap G3BP1 PAR-CLIP targets with ~4,769 shared target genes, matching Fig. 2b,c / Supplementary Table 2
- Dual-editor (rAPOBEC1–pAG–hADAR2dd) configuration recovers approximately 20–30% more published targets than either single editor alone
- PRC2 component (EED/EZH2/SUZ12) target sets remain near-empty, supporting weak PRC2–RNA binding
- All called targets pass the Poisson editing-site filter (p ≤ 0.05), minimum editing-site coverage 10, and transcript-level thresholds fold enrichment > 2, MAPIT score > 0.5, adjusted P < 0.05

## Limitations

- The Poisson filter default is a maximum editing-site p-value of 0.1; the reported results use a tightened 0.05, so reusing defaults will loosen target calls
- Called targets depend on matched controls and the chosen --treatName/--controlName pairing; mismatched controls invalidate differential statistics
- Target lists are transcript-binned (binSize 50) with minimum coverage 10 — low-coverage transcripts with fewer than 10 reads at editing sites are dropped as invalid

## Evidence

- [readme] dual-deaminase target identification rationale: "enables the identification RNA-binding protein (RBP) target transcripts by detecting adjacent RNA editing events introduced by both hADAR2dd and rAPOBEC1"
- [readme] installation route: "All softwares in our pipeline are available in [conda](https://docs.conda.io/en/latest/miniconda.html). We recommend you to install these softwares via conda"
