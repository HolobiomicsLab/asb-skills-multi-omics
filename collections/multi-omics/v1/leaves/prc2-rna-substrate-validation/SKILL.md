---
name: prc2-rna-substrate-validation
description: Use when validating RBP target calls from MAPIT-seq editing data — specifically
  reproducing or extending near-empty PRC2 component target sets (EED, EZH2, SUZ12)
  and verifying XIST adjusted P values.
when_to_use_negative:
- Input is raw sequencing data rather than post-callediting editing event tables —
  the mapping, finetuning and callediting steps must run first
- The goal is transcriptome-level quantification (gene expression co-profiling) rather
  than differential-editing-based RBP target calling
- Detecting RBP-binding motifs or high-confidence binding clusters (hc_cluster) rather
  than validating target-gene calls at adjusted-P level
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: MAPIT-seq pipeline (Mapit calltargets)
  role: Differential editing analysis step that splits transcripts into 50-nt bins,
    applies the Poisson filter (default max p-value 0.1), coverage filter (min 10)
    and calls RBP binding targets for each PRC2 component versus control
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: conda
  role: Recommended installation channel for all software in the MAPIT-seq pipeline
    used to run calltargets on PRC2 component samples
- name: SAILOR
  role: Downstream edit-site finding pipeline supported via the prepare step; part
    of the MAPIT-seq editing-detection toolchain
  repo: https://github.com/YeoLab/FLARE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:183b32062ff5fd0c9bbe7c89341ed3a3f890bd1661b6ba0a75ecc84de630ee31
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
  - MAPIT-seq pipeline (Mapit calltargets)
  - SAILOR
  - conda
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# prc2-rna-substrate-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the MAPIT-seq target-calling result that XIST is the only common RNA target of the core PRC2 components (EED, EZH2, SUZ12), by running differential editing analysis (Mapit calltargets) on per-sample editing event tables and comparing XIST adjusted P values against reported values (0.0463, 1.72 × 10⁻³, 9.94 × 10⁻⁶).

## When to use

Use when validating RBP target calls from MAPIT-seq editing data — specifically reproducing or extending near-empty PRC2 component target sets (EED, EZH2, SUZ12) and verifying XIST adjusted P values.

## When NOT to use

- Input is raw sequencing data rather than post-callediting editing event tables — the mapping, finetuning and callediting steps must run first
- The goal is transcriptome-level quantification (gene expression co-profiling) rather than differential-editing-based RBP target calling
- Detecting RBP-binding motifs or high-confidence binding clusters (hc_cluster) rather than validating target-gene calls at adjusted-P level

## Inputs

- Per-sample RNA editing event tables from the callediting step (*_Edit_*_DATA.tsv under 6-Edit_calling)
- RBP treatment and control sample assignments for EED, EZH2 and SUZ12 MAPIT-seq experiments
- Gene/transcript annotation used for binning transcripts into 50-nt bins

## Outputs

- prc2_target_adjusted_pvalues.csv with reproduced XIST adjusted P values per PRC2 component
- Per-component significant RBP target gene lists (expect near-empty sets for EED, EZH2, SUZ12)

## How to apply

Load the per-sample RNA editing event tables (e.g., *_Edit_*_DATA.tsv under 6-Edit_calling) produced by the callediting (GATK RNA editing detection) step for the EED, EZH2 and SUZ12 samples. Run the differential editing analysis (Mapit calltargets) for each RBP treatment versus its control, splitting transcripts into 50-nt bins (binSize 50) and requiring a minimum editing-site coverage of 10 (default). Apply the Poisson filter of editing sites (default maximum p-value 0.1, tightened to the reported significance threshold), compute gene/transcript-level adjusted P values, and enforce the MAPIT-seq target-calling thresholds: editing fold enrichment > 2, MAPIT score > 0.5, adjusted P < 0.05. Extract XIST adjusted P values per component and compare against the reported values — 0.0463 (EED), 1.72 × 10⁻³ (EZH2), 9.94 × 10⁻⁶ (SUZ12) — then compile per-component significant target gene lists to confirm the near-empty target sets that support weak PRC2–RNA binding.

## Related tools

- **MAPIT-seq pipeline (Mapit calltargets)** (Differential editing analysis step that splits transcripts into 50-nt bins, applies the Poisson filter (default max p-value 0.1), coverage filter (min 10) and calls RBP binding targets for each PRC2 component versus control) — https://github.com/WangLabPKU/MAPIT-seq
- **conda** (Recommended installation channel for all software in the MAPIT-seq pipeline used to run calltargets on PRC2 component samples)
- **SAILOR** (Downstream edit-site finding pipeline supported via the prepare step; part of the MAPIT-seq editing-detection toolchain) — https://github.com/YeoLab/FLARE

## Examples

```
Mapit calltargets --Edit_data 6-Edit_calling/*_Edit_*_DATA.tsv --binSize 50 --coverage 10 --pvalue 0.1
```

## Evaluation signals

- Reproduced XIST adjusted P values match reported values within numerical tolerance: 0.0463 (EED), 1.72 × 10⁻³ (EZH2), 9.94 × 10⁻⁶ (SUZ12)
- Significant target gene sets for each of EED, EZH2 and SUZ12 remain near-empty after applying thresholds (editing fold enrichment > 2, MAPIT score > 0.5, adjusted P < 0.05)
- XIST is the only gene present in the intersection of the three PRC2 component target sets
- Editing sites retained pass the Poisson filter (p ≤ 0.1) and minimum coverage of 10 as enforced by calltargets parameters

## Limitations

- Near-empty target sets for PRC2 components are interpreted as supporting weak PRC2–RNA binding, so absence of calls is itself the expected result — do not treat an empty list as pipeline failure
- The Poisson filter p-value default is 0.1; achieving the reported significance requires tightening this threshold, and results are sensitive to binSize (50 nt) and minimum coverage (10) choices
- Editing detection depends on prior known-SNP masking (dbSNP, 1000 Genomes, EVS/EVA) and chromosome-nomenclature consistency, per the FLARE/SAILOR prerequisites

## Evidence

- [results] Target calling on the three core PRC2 components identified XIST as the only common target: XIST adjusted P values were 0.0463 (EED), 1.72 × 10−3 (EZH2) and 9.94 × 10−6 (SUZ12), with near-empty target sets supporting weak PRC2–RNA binding.: [evidence span withheld — anchor status: not_found]
