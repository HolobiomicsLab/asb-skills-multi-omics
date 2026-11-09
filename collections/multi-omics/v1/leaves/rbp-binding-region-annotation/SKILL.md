---
name: rbp-binding-region-annotation
description: Use when MAPIT-seq alignments in Mapit_result must be converted into
  high-confidence RBP-binding regions via adjacent hADAR2dd (A-to-G) and rAPOBEC1
  (C-to-U) edit clusters.
when_to_use_negative:
- Input is differential editing data for calling RBP targets — use the 'calltargets'
  step instead, which is a separate pipeline stage
- Data come from a single non-MAPIT-seq experiment without paired A-to-G and C-to-U
  editing signatures from hADAR2dd and rAPOBEC1
- Alignments have not yet passed the two-round uniquely mapping and finetuning stages
  of the MAPIT-seq pipeline
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SAILOR
  role: Per-sample Snakemake workflow calling editing sites with coverage >= 10 and
    confidence thresholds >0.5 (C-to-U) and >0.9 (A-to-G)
  repo: https://github.com/YeoLab/FLARE
- name: FLARE
  role: Snakemake pipeline that statistically determines regions of enriched RNA editing
    from SAILOR-derived sites, invoked with -e AG or -e CT to define edit clusters
  repo: https://github.com/YeoLab/FLARE
- name: Snakemake
  role: Workflow engine executing the SAILOR and FLARE cluster-identification steps
- name: deeptools
  role: Python dependency of the MAPIT-seq pipeline environment supporting the cluster-identification
    step
- name: gffutils
  role: Python dependency used for handling GTF annotations during export of high-confidence
    clusters
- name: pyfaidx
  role: Python dependency for FASTA indexing used when expanding cluster intervals
    by slop length
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:5b7ead75cc9e2527d8346c208cf97a2beac702cfb12222800b6b6e2341999312
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WangLabPKU/MAPIT-seq
  repo_url: https://github.com/YeoLab/FLARE
  tools:
  - FLARE
  - SAILOR
  - Snakemake
  - conda
  - deeptools
  - gffutils
  - pyfaidx
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# rbp-binding-region-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies high-confidence RBP-binding regions in MAPIT-seq data by calling SAILOR editing sites, delineating A-to-G and C-to-U edit clusters with FLARE, and pairing replicate-supported clusters into a BED/GTF annotation.

## When to use

Use when MAPIT-seq alignments in Mapit_result must be converted into high-confidence RBP-binding regions via adjacent hADAR2dd (A-to-G) and rAPOBEC1 (C-to-U) edit clusters.

## When NOT to use

- Input is differential editing data for calling RBP targets — use the 'calltargets' step instead, which is a separate pipeline stage
- Data come from a single non-MAPIT-seq experiment without paired A-to-G and C-to-U editing signatures from hADAR2dd and rAPOBEC1
- Alignments have not yet passed the two-round uniquely mapping and finetuning stages of the MAPIT-seq pipeline

## Inputs

- Aligned reads in the Mapit_result directory (MAPIT-seq BAM alignments)
- Genome regions configuration directory for FLARE
- Known SNP annotation (e.g., dbSNP combined chromosome bed3 files) for editing-site filtering

## Outputs

- SAILOR per-sample editing site tables filtered by coverage >= 10 and confidence thresholds
- FLARE A-to-G (AG) and C-to-U (CT) edit cluster intervals per replicate
- High-confidence paired-cluster BED/GTF annotation of RBP-binding regions

## How to apply

First run the Mapit 'prepare' step on alignments in the Mapit_result directory to stage files for SAILOR. Run the SAILOR Snakemake workflow per sample with a minimum coverage of 10 for valid editing sites and confidence-score thresholds >0.5 for C-to-U sites and >0.9 for A-to-G sites, so only high-quality deamination signals enter clustering. Then run the FLARE (Flagging Areas of RNA-editing Enrichment) workflow with the genome regions configuration directory and the editing-type flag (-e AG or -e CT) to delineate genomic edit clusters from the filtered sites. Intersect cluster calls across all replicates, keeping only reproducibly detected clusters, then pair C-to-U and A-to-G clusters within 100 nt of each other (expanding intervals by the slop length) via the 'hc_cluster' step. Export the paired, replicate-supported clusters as a BED/GTF annotation file that downstream target-calling and motif-discovery steps consume.

## Related tools

- **SAILOR** (Per-sample Snakemake workflow calling editing sites with coverage >= 10 and confidence thresholds >0.5 (C-to-U) and >0.9 (A-to-G)) — https://github.com/YeoLab/FLARE
- **FLARE** (Snakemake pipeline that statistically determines regions of enriched RNA editing from SAILOR-derived sites, invoked with -e AG or -e CT to define edit clusters) — https://github.com/YeoLab/FLARE
- **Snakemake** (Workflow engine executing the SAILOR and FLARE cluster-identification steps)
- **deeptools** (Python dependency of the MAPIT-seq pipeline environment supporting the cluster-identification step)
- **gffutils** (Python dependency used for handling GTF annotations during export of high-confidence clusters)
- **pyfaidx** (Python dependency for FASTA indexing used when expanding cluster intervals by slop length)

## Examples

```
snakemake -e AG
```

## Evaluation signals

- Output BED/GTF exists and contains only high-confidence clusters formed by replicate-intersected C-to-U and A-to-G cluster pairs within the 100 nt pairing distance
- Every retained cluster is reproducibly detected in all replicates after the intersect step
- Editing sites entering FLARE satisfy minimum coverage of 10 and confidence scores >0.5 (C-to-U) and >0.9 (A-to-G)
- Paired clusters arise from AG and CT cluster sets that are within 100 nt after interval expansion by the specified slop length

## Limitations

- Clusters discarded by the replicate-intersection step reduce sensitivity if a binding event is detected in only a subset of replicates
- The 100 nt pairing distance (plus slop) is a fixed heuristic; binding regions with AG and CT clusters spaced further apart are not annotated as high-confidence
- FLARE requires snakemake and Singularity on a Linux environment (e.g., CentOS 7 or above), which may constrain portability

## Evidence

- [readme] a Snakemake-based pipeline that uses a statistical approach to determine regions of enriched RNA editing, using SAILOR-derived editing sites as a starting point: [evidence span withheld — anchor status: not_found]
- [readme] environment requirement for running SAILOR and FLARE: [evidence span withheld — anchor status: not_found]
