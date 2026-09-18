---
name: replicate-reproducibility-intersection
description: Use when you have per-replicate editing-site/cluster calls (e.g., SAILOR+FLARE
  output for A-to-G or C-to-U edits) and need to keep only clusters supported in every
  replicate before pairing edit types.
when_to_use_negative:
- You have only a single replicate — the intersection of one cluster set provides
  no reproducibility filtering, and confidence must be assessed by other means (e.g.,
  score thresholds alone).
- You want to detect condition-specific editing changes rather than constitutive RBP-bound
  regions — replicate intersection would discard genuine differential signal.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: FLARE
  role: Produces the per-editing-type genomic edit clusters that are intersected across
    replicates
  repo: https://github.com/YeoLab/FLARE
- name: SAILOR
  role: Calls per-sample editing sites (coverage >=10, confidence >0.5 C-to-U / >0.9
    A-to-G) that feed FLARE clustering
  repo: https://github.com/YeoLab/FLARE
- name: Snakemake
  role: Workflow engine executing the SAILOR/FLARE steps per sample and replicate
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:f7c5675533633fa25055832422c616db79f22bc9993804d9cad5278b2f6a868a
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# replicate-reproducibility-intersection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retains only genomic edit clusters that are reproducibly detected across all biological replicates by intersecting per-replicate FLARE cluster calls, a filtering step that underlies MAPIT-seq high-confidence RBP-binding region definition.

## When to use

Use when you have per-replicate editing-site/cluster calls (e.g., SAILOR+FLARE output for A-to-G or C-to-U edits) and need to keep only clusters supported in every replicate before pairing edit types.

## When NOT to use

- You have only a single replicate — the intersection of one cluster set provides no reproducibility filtering, and confidence must be assessed by other means (e.g., score thresholds alone).
- You want to detect condition-specific editing changes rather than constitutive RBP-bound regions — replicate intersection would discard genuine differential signal.

## Inputs

- Per-replicate FLARE genomic edit-cluster calls (BED interval sets) for each editing type (A-to-G / -e AG, C-to-U / -e CT)
- SAILOR-derived editing sites filtered at minimum coverage 10 with confidence scores >0.5 (C-to-U) and >0.9 (A-to-G)
- Genome regions configuration directory used to configure FLARE

## Outputs

- Replicate-intersected edit-cluster sets containing only clusters detected in all replicates
- Downstream paired, replicate-supported high-confidence clusters exported as BED/GTF annotation (via hc_cluster)

## How to apply

First run the SAILOR Snakemake workflow per sample to call editing sites, requiring minimum coverage of 10 and confidence-score thresholds >0.5 for C-to-U and >0.9 for A-to-G sites. Then run FLARE with the editing-type flag (-e AG for A-to-G, CT for C-to-U) to delineate genomic edit clusters from the filtered sites. Run FLARE independently on each replicate so every replicate yields its own cluster set. Intersect the per-replicate cluster calls so that only clusters reproducibly detected in all replicates are retained — this removes stochastic editing artifacts. The retained replicate-intersected C-to-U and A-to-G clusters are then paired within 100 nt (expanded by the slop length) by the hc_cluster step to define MAPIT high-confidence edit clusters.

## Related tools

- **FLARE** (Produces the per-editing-type genomic edit clusters that are intersected across replicates) — https://github.com/YeoLab/FLARE
- **SAILOR** (Calls per-sample editing sites (coverage >=10, confidence >0.5 C-to-U / >0.9 A-to-G) that feed FLARE clustering) — https://github.com/YeoLab/FLARE
- **Snakemake** (Workflow engine executing the SAILOR/FLARE steps per sample and replicate)

## Examples

```
snakemake --configfile config.yaml --cores 8  # run FLARE per replicate (-e AG / -e CT), then intersect per-replicate cluster BEDs: bedtools intersect -a rep1_clusters.bed -b rep2_clusters.bed -c > intersected_clusters.bed
```

## Evaluation signals

- Every cluster in the intersected output is present in the cluster set of each individual replicate (set-inclusion invariant).
- No cluster lacking a matched partner in every replicate survives the intersection.
- The downstream hc_cluster BED/GTF contains only high-confidence clusters formed by replicate-intersected C-to-U and A-to-G cluster pairs within the 100 nt pairing distance.
- Cluster coordinates are consistent with the slop-expanded intervals used for pairing C-to-U and A-to-G clusters.

## Limitations

- Reproducibility-by-intersection is a strict AND operation: clusters missing from even one replicate are discarded, which can drop weaker but real binding sites.
- The intersection is only as good as the upstream SAILOR thresholds (coverage 10; scores >0.5 C-to-U, >0.9 A-to-G) — misconfigured thresholds propagate to cluster sets before intersection.

## Evidence

- [other] Intersect cluster calls across replicates so that only clusters reproducibly detected in all replicates are retained.: [evidence span withheld — anchor status: not_found]
- [readme] FLARE is a Snakemake-based pipeline that uses a statistical approach to determine regions of enriched RNA editing, using SAILOR-derived editing sites as a starting point.: [evidence span withheld — anchor status: not_found]
- [methods] requiring minimum coverage of 10 for valid editing sites and applying confidence-score thresholds greater than 0.5 for C-to-U sites and greater than 0.9 for A-to-G sites: [evidence span withheld — anchor status: not_found]
- [other] verify the output BED/GTF exists and contains only high-confidence clusters formed by replicate-intersected C-to-U and A-to-G cluster pairs within the 100 nt pairing distance: [evidence span withheld — anchor status: not_found]
