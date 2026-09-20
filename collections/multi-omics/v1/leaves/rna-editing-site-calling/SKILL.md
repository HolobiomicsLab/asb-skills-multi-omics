---
name: rna-editing-site-calling
description: Use when MAPIT-seq alignments (Mapit_result directory) are ready and
  you must detect C-to-U and A-to-G editing events from hADAR2dd and rAPOBEC1 to pinpoint
  RBP-binding regions.
when_to_use_negative:
- The assay targets only a single editing enzyme/type, so paired adjacent C-to-U and
  A-to-G clusters cannot be formed and the hc_cluster confidence logic does not apply
- Input alignments lack matched known-SNP annotation for the organism/build, so genuine
  SNPs cannot be filtered from candidate editing sites
- The goal is generic variant calling on DNA-Seq rather than RNA editing detection
  from RNA-Seq of edited transcriptome data
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SAILOR
  role: Snakemake workflow that calls per-sample RNA editing sites with coverage,
    Poisson, and confidence-score filtering
  repo: https://github.com/YeoLab/FLARE
- name: FLARE
  role: Snakemake pipeline that flags areas of RNA-editing enrichment (edit clusters)
    from SAILOR-derived editing sites
  repo: https://github.com/YeoLab/FLARE
- name: Snakemake
  role: Workflow engine orchestrating the SAILOR and FLARE steps of the MAPIT-seq
    pipeline
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: REDItools2
  role: Alternative/related tool for calling RNA-Seq editing events from BAM alignments
  repo: https://github.com/BioinfoUNIBA/REDItools2
- name: conda
  role: Recommended installation channel for all pipeline software
  repo: https://github.com/WangLabPKU/MAPIT-seq
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:2f217ba534cf5ce6f5995be528a4a7a50f5be5b93a9581bd8c829a4654ebbcfd
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
  - REDItools2
  - SAILOR
  - Snakemake
  - conda
  - deeptools
  - gffutils
  - pyfaidx
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# rna-editing-site-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calls RNA editing events from MAPIT-seq alignments using the SAILOR Snakemake workflow (with known-SNP filtering and Poisson/coverage/confidence thresholds), then delineates enriched editing regions with FLARE to support high-confidence RBP-binding cluster identification.

## When to use

Use when MAPIT-seq alignments (Mapit_result directory) are ready and you must detect C-to-U and A-to-G editing events from hADAR2dd and rAPOBEC1 to pinpoint RBP-binding regions.

## When NOT to use

- The assay targets only a single editing enzyme/type, so paired adjacent C-to-U and A-to-G clusters cannot be formed and the hc_cluster confidence logic does not apply
- Input alignments lack matched known-SNP annotation for the organism/build, so genuine SNPs cannot be filtered from candidate editing sites
- The goal is generic variant calling on DNA-Seq rather than RNA editing detection from RNA-Seq of edited transcriptome data

## Inputs

- BAM alignment files from the Mapit_result directory (after prepare step)
- Known SNP bedfiles split by chromosome (dbSNP, 1000 Genomes, EVS/EVA), reduced to chrom/start/end columns
- Reference genome and genome regions configuration directory for FLARE
- Editing-type flag choice (-e AG for A-to-G, -e CT for C-to-U)

## Outputs

- Per-sample SAILOR editing-site tables filtered by coverage, Poisson p-value, and confidence score
- FLARE edit-cluster calls per editing type (A-to-G, C-to-U)
- Replicate-intersected, paired high-confidence edit clusters as a BED/GTF annotation file

## How to apply

First prepare alignments in the Mapit_result directory via the Mapit prepare step, supplying known SNP bedfiles (e.g. dbSNP, 1000 Genomes, EVS/EVA split by chromosome, reduced to chrom-start-end columns with matching chromosome nomenclature). Run the SAILOR Snakemake workflow per sample to call editing sites, requiring minimum coverage of 10 for valid sites, applying the Poisson filter (default p-value 0.1), and using confidence-score thresholds greater than 0.5 for C-to-U sites and greater than 0.9 for A-to-G sites. Then run the FLARE workflow (-e AG for A-to-G, -e CT for C-to-U) to delineate genomic edit clusters from the filtered SAILOR sites, intersect clusters across replicates, and pair C-to-U and A-to-G clusters within 100 nt (with slop expansion) in the hc_cluster step to define MAPIT high-confidence edit clusters exported as BED/GTF. Rationale: requiring both adjacent editing types (hADAR2dd A-to-G and rAPOBEC1 C-to-U) within 100 nt distinguishes true antibody-directed editing from background, which is what confers confidence to RBP-binding calls.

## Related tools

- **SAILOR** (Snakemake workflow that calls per-sample RNA editing sites with coverage, Poisson, and confidence-score filtering) — https://github.com/YeoLab/FLARE
- **FLARE** (Snakemake pipeline that flags areas of RNA-editing enrichment (edit clusters) from SAILOR-derived editing sites) — https://github.com/YeoLab/FLARE
- **Snakemake** (Workflow engine orchestrating the SAILOR and FLARE steps of the MAPIT-seq pipeline) — https://github.com/WangLabPKU/MAPIT-seq
- **REDItools2** (Alternative/related tool for calling RNA-Seq editing events from BAM alignments) — https://github.com/BioinfoUNIBA/REDItools2
- **conda** (Recommended installation channel for all pipeline software) — https://github.com/WangLabPKU/MAPIT-seq

## Examples

```
for b in $(ls *.bed); do echo $b; tail -n+2 $b | cut -f1,2,3 | sort >> mm10_dbsnp_combined.bed3; done
```

## Evaluation signals

- All retained editing sites have coverage >= 10 (minimum coverage for valid editing sites)
- Confidence scores pass thresholds: > 0.5 for C-to-U sites and > 0.9 for A-to-G sites; Poisson-filter p-values do not exceed the 0.1 default
- Output high-confidence clusters are only those reproducibly detected in all replicates after intersecting FLARE cluster calls
- Each high-confidence cluster is a paired C-to-U + A-to-G cluster with intervals within 100 nt after slop expansion
- Final BED/GTF exists and contains only high-confidence paired clusters; chromosome nomenclature matches the reference and known-SNP bedfiles

## Limitations

- FLARE requires Snakemake and Singularity, and the system must be at least Linux CentOS7; chromosome nomenclature must be consistent between the reference and known-SNP bedfiles
- REDItools2 was tested only with Python 2.7.12 and offers no guarantee of compatibility with other Python versions
- Known SNPs must be split by chromosome and manually combined (header lines removed, only chrom/start/end retained), an error-prone preprocessing step

## Evidence

- [methods] High-confidence cluster definition: [evidence span withheld — anchor status: not_found]
- [readme] FLARE purpose: [evidence span withheld — anchor status: not_found]
