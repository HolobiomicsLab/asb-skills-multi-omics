---
name: edit-cluster-delineation-from-per-site-calls
description: Use when per-nucleotide RNA editing calls (SAILOR BED sites from hADAR2dd
  A-to-G and rAPOBEC1 C-to-U deamination) must be aggregated into reproducible paired
  edit clusters marking high-confidence RBP-binding regions.
when_to_use_negative:
- When the goal is per-site editing detection rather than region-level clustering
  — use the SAILOR/GATK callediting step instead
- When data contain only a single editing type (only A-to-G or only C-to-U), since
  high-confidence MAPIT clusters require paired adjacent events from both hADAR2dd
  and rAPOBEC1
- When only a single replicate is available, since clusters must be reproducibly detected
  in all replicates to be retained
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: FLARE
  role: Statistically delineates regions of enriched RNA editing (edit clusters) from
    SAILOR-derived sites, configurable per editing type via the -e AG/CT flag
  repo: https://github.com/YeoLab/FLARE
- name: SAILOR
  role: Snakemake-based per-sample editing-site caller whose filtered outputs (coverage
    >=10, confidence >0.5 CT / >0.9 AG) are the starting point for FLARE
  repo: https://github.com/YeoLab/FLARE
- name: Snakemake
  role: Workflow engine for the SAILOR and FLARE pipelines; requires singularity for
    several FLARE steps
- name: MAPIT-seq pipeline
  role: 'Hosts the prepare and hc_cluster steps: prepare readies Mapit_result files
    for SAILOR; hc_cluster pairs C-to-U and A-to-G clusters within 100 nt and exports
    BED/GTF'
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: pyfaidx
  role: Python dependency installed alongside gffutils/deeptools for the MAPIT-seq
    environment supporting cluster handling and annotation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:c14dd3cc55a77b73bc07dcb5fe121343c4ac5ff670ae214f8820ae54f5a650d1
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
  - MAPIT-seq pipeline
  - SAILOR
  - Snakemake
  - conda
  - deeptools
  - gffutils
  - pyfaidx
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.2
n_claims: 10
---

# Edit-cluster delineation from per-site calls

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A MAPIT-seq skill for converting SAILOR-called RNA editing sites into high-confidence RBP-binding regions: FLARE statistically delineates enriched edit clusters per editing type, replicate intersections enforce reproducibility, and the hc_cluster step pairs adjacent C-to-U and A-to-G clusters within 100 nt.

## When to use

Use when per-nucleotide RNA editing calls (SAILOR BED sites from hADAR2dd A-to-G and rAPOBEC1 C-to-U deamination) must be aggregated into reproducible paired edit clusters marking high-confidence RBP-binding regions.

## When NOT to use

- When the goal is per-site editing detection rather than region-level clustering — use the SAILOR/GATK callediting step instead
- When data contain only a single editing type (only A-to-G or only C-to-U), since high-confidence MAPIT clusters require paired adjacent events from both hADAR2dd and rAPOBEC1
- When only a single replicate is available, since clusters must be reproducibly detected in all replicates to be retained

## Inputs

- Alignment outputs in the Mapit_result directory (BAM alignments from the two-round uniquely mapping step)
- Per-sample SAILOR editing-site calls (BED), filtered at minimum coverage 10 with confidence scores >0.5 (C-to-U) and >0.9 (A-to-G)
- Genome regions configuration directory for FLARE
- Known SNP bedfiles (e.g. dbSNP) split by chromosome with matching nomenclature

## Outputs

- Per-replicate genomic edit clusters from FLARE for AG and CT editing types
- Replicate-intersected C-to-U and A-to-G cluster pairs within 100 nt (high-confidence MAPIT edit clusters)
- BED/GTF annotation file of high-confidence RBP-binding edit clusters

## How to apply

First prepare the alignment outputs in the Mapit_result directory with the Mapit prepare step, then run the SAILOR Snakemake workflow per sample, calling editing sites with minimum coverage of 10 and confidence-score thresholds greater than 0.5 for C-to-U sites and greater than 0.9 for A-to-G sites. Run FLARE — a Snakemake-based pipeline that statistically determines regions of enriched RNA editing — with the genome regions configuration directory and the editing-type flag (-e AG for A-to-G, -e CT for C-to-U) to delineate genomic edit clusters from the filtered sites. Intersect cluster calls across replicates so only clusters reproducibly detected in all replicates are retained. Finally, run the hc_cluster step to pair C-to-U and A-to-G clusters located within 100 nt of each other (expanding cluster intervals by the specified slop length) and export the paired clusters as a BED/GTF annotation file; the dual-editing-type pairing is the rationale, since MAPIT-seq targets transcripts by detecting adjacent editing events from both hADAR2dd and rAPOBEC1.

## Related tools

- **FLARE** (Statistically delineates regions of enriched RNA editing (edit clusters) from SAILOR-derived sites, configurable per editing type via the -e AG/CT flag) — https://github.com/YeoLab/FLARE
- **SAILOR** (Snakemake-based per-sample editing-site caller whose filtered outputs (coverage >=10, confidence >0.5 CT / >0.9 AG) are the starting point for FLARE) — https://github.com/YeoLab/FLARE
- **Snakemake** (Workflow engine for the SAILOR and FLARE pipelines; requires singularity for several FLARE steps)
- **MAPIT-seq pipeline** (Hosts the prepare and hc_cluster steps: prepare readies Mapit_result files for SAILOR; hc_cluster pairs C-to-U and A-to-G clusters within 100 nt and exports BED/GTF) — https://github.com/WangLabPKU/MAPIT-seq
- **pyfaidx** (Python dependency installed alongside gffutils/deeptools for the MAPIT-seq environment supporting cluster handling and annotation)

## Examples

```
git clone https://github.com/YeoLab/FLARE  # then run SAILOR site calling and FLARE (-e AG / -e CT) via snakemake per sample
```

## Evaluation signals

- Output BED/GTF exists and contains only high-confidence clusters formed by replicate-intersected C-to-U and A-to-G cluster pairs within the 100 nt pairing distance
- Every retained cluster is detected in all replicates (replicate intersection is non-empty and consistent)
- Editing-site inputs obey SAILOR thresholds: minimum coverage of 10, confidence score >0.5 for C-to-U and >0.9 for A-to-G sites
- Chromosome nomenclature of the known-SNP bedfiles matches the genome reference, otherwise SAILOR site calling silently loses SNP masking
- Paired clusters co-localize the two editing types within the expanded slop interval, consistent with adjacent hADAR2dd/rAPOBEC1 deamination signatures

## Limitations

- FLARE requires Linux CentOS7 or newer, with snakemake and Singularity installed — several pipeline steps load Singularity images automatically and will fail otherwise
- Known-SNP bedfiles must have the same chromosome nomenclature as the reference and headers stripped, or SNP filtering breaks
- The published paper does not explain why MAPIT-seq's signal-to-noise advantages hold in all contexts; confidence thresholds (0.5/0.9, coverage 10, 100 nt slop) are pipeline defaults rather than empirically optimized per dataset
- Application of the single-cell variant (scMAPIT-seq) to tissue samples is not yet demonstrated

## Evidence

- [readme] FLARE description: [evidence span withheld — anchor status: not_found]
- [readme] Pipeline goal: "The MAPIT-seq pipeline is designed to detect RNA editing events from hADAR2dd and rAPOBEC1, identify RBP targets, pinpoint high-confidence RBP-binding regions"
- [intro] Intro statement: "enables the identification RNA-binding protein (RBP) target transcripts by detecting adjacent RNA editing events introduced by both hADAR2dd and rAPOBEC1"
