---
name: genomic-interval-pairing-and-slop-extension
description: Use when a MAPIT-seq analysis needs to convert filtered editing sites
  into high-confidence RBP-binding regions by requiring adjacent C-to-U and A-to-G
  edit clusters reproducible across replicates.
when_to_use_negative:
- Editing sites come from a single editing enzyme only — the high-confidence definition
  requires both C-to-U and A-to-G clusters paired within 100 nt
- Data are single-replicate with no cross-replicate intersection possible
- You only need per-site editing calls, not clustered regions
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: SAILOR
  role: Snakemake workflow that calls per-sample RNA editing sites with coverage and
    confidence-score thresholds, feeding FLARE cluster delineation
  repo: https://github.com/YeoLab/FLARE
- name: FLARE
  role: Flags areas of RNA-editing enrichment to delineate genomic edit clusters from
    SAILOR-derived editing sites, configured per editing type via -e AG/CT
  repo: https://github.com/YeoLab/FLARE
- name: Snakemake
  role: Workflow engine for the SAILOR and FLARE pipelines
  repo: https://github.com/YeoLab/FLARE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:e0b75c087682165ff716ef384602a7a33aad9cc00aac8e450014a390d1c5cd2d
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# Genomic interval pairing and slop extension

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies high-confidence RBP-binding regions in MAPIT-seq by delineating edit clusters with FLARE from SAILOR-called sites, intersecting them across replicates, and pairing C-to-U and A-to-G clusters within a 100 nt slop-expanded window via the hc_cluster step.

## When to use

Use when a MAPIT-seq analysis needs to convert filtered editing sites into high-confidence RBP-binding regions by requiring adjacent C-to-U and A-to-G edit clusters reproducible across replicates.

## When NOT to use

- Editing sites come from a single editing enzyme only — the high-confidence definition requires both C-to-U and A-to-G clusters paired within 100 nt
- Data are single-replicate with no cross-replicate intersection possible
- You only need per-site editing calls, not clustered regions

## Inputs

- Alignment outputs in the Mapit_result directory prepared via the Mapit prepare step
- SAILOR-called editing sites filtered by coverage >=10 and confidence thresholds (>0.5 C-to-U, >0.9 A-to-G)
- Genome regions configuration directory for FLARE

## Outputs

- Replicate-intersected C-to-U and A-to-G edit cluster sets (FLARE output)
- BED/GTF annotation file of paired high-confidence edit clusters from the hc_cluster step

## How to apply

Run the SAILOR Snakemake workflow per sample on Mapit_result alignments (min coverage 10; confidence score >0.5 for C-to-U, >0.9 for A-to-G), then run FLARE with the genome regions configuration and the editing-type flag (-e AG or -e CT) to delineate genomic edit clusters. Intersect cluster calls across replicates so only clusters detected in all replicates are retained. Use the hc_cluster step to pair C-to-U and A-to-G clusters whose intervals, expanded by the specified slop length, fall within 100 nt of each other — the dual-editing signature of hADAR2dd and rAPOBEC1 marks genuine RBP-binding sites. Export the paired, replicate-supported clusters as a BED/GTF annotation for downstream motif discovery and target calling.

## Related tools

- **SAILOR** (Snakemake workflow that calls per-sample RNA editing sites with coverage and confidence-score thresholds, feeding FLARE cluster delineation) — https://github.com/YeoLab/FLARE
- **FLARE** (Flags areas of RNA-editing enrichment to delineate genomic edit clusters from SAILOR-derived editing sites, configured per editing type via -e AG/CT) — https://github.com/YeoLab/FLARE
- **Snakemake** (Workflow engine for the SAILOR and FLARE pipelines) — https://github.com/YeoLab/FLARE

## Examples

```
snakemake --snakefile FLARE/Snakefile --config regions=genome_regions -e AG
```

## Evaluation signals

- Output BED/GTF exists and contains only clusters formed by replicate-intersected C-to-U and A-to-G cluster pairs
- Every retained cluster pair lies within the 100 nt pairing distance after slop expansion of cluster intervals
- Editing sites entering cluster calling satisfy minimum coverage 10 and confidence-score thresholds (>0.5 C-to-U, >0.9 A-to-G)
- Per-editing-type FLARE runs used the correct flag (-e AG for A-to-G, -e CT for C-to-U)

## Limitations

- High-confidence clusters require reproducibility across all replicates, so single-replicate designs cannot produce them
- The 100 nt pairing distance is a fixed pipeline parameter; clusters separated by more than the slop-expanded distance are not paired

## Evidence

- [other] Dual editing types define high-confidence clusters: [evidence span withheld — anchor status: not_found]
- [other] SAILOR thresholds: [evidence span withheld — anchor status: not_found]
- [readme] FLARE Snakemake design: [evidence span withheld — anchor status: not_found]
- [methods] hc_cluster step in methods: "I(**hc_cluster**
  High-confidence edit cluster identification)"
- [intro] Purpose of pinpointing binding regions: "pinpoint high-confidence RBP-binding regions"
