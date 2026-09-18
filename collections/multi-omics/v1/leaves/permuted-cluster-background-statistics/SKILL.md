---
name: permuted-cluster-background-statistics
description: 'Use when validating de novo motif discovery on MAPIT-seq edit clusters:
  you need a matched control background (permuted clusters in the same genes) to compute
  fold enrichment of a known motif such as UGCAUG against HOMER output.'
when_to_use_negative:
- The question is calling RBP binding targets or high-confidence clusters themselves
  (use the calltargets / hc_cluster steps instead) — this skill only assesses motif
  enrichment against a matched background.
- No permuted control clusters in the same genes are available; comparing against
  an unmatched genomic background would not reproduce the reported two- to sixfold
  enrichment range.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3511
tools:
- name: HOMER
  role: De novo motif discovery (findMotifsGenome.pl) on expanded RBFOX2 clusters
    with the permuted-cluster background, and comparison against the known motif library
- name: pyfaidx
  role: Extracts GRCh38 genomic sequences underlying the expanded real and permuted
    cluster intervals for background construction
- name: MAPIT-seq pipeline (hc_cluster step)
  role: Produces the high-confidence edit clusters and the --sloplength interval-expansion
    convention used to build the 300 nt motif-search windows
  repo: https://github.com/WangLabPKU/MAPIT-seq
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:b8fabeaebc7ab68b508686e0bed3642dcdfc2f03e9fbd81d6d85c90d8b3f26d5
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
  - HOMER
  - MAPIT-seq pipeline (hc_cluster step)
  - pyfaidx
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# permuted-cluster-background-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a GC- and length-matched permuted edit-cluster background to quantify fold enrichment of de novo motifs (e.g., RBFOX2's UGCAUG) recovered from MAPIT-seq high-confidence edit clusters, validating that motif recovery reflects true RBP-binding signal rather than sequence composition.

## When to use

Use when validating de novo motif discovery on MAPIT-seq edit clusters: you need a matched control background (permuted clusters in the same genes) to compute fold enrichment of a known motif such as UGCAUG against HOMER output.

## When NOT to use

- The question is calling RBP binding targets or high-confidence clusters themselves (use the calltargets / hc_cluster steps instead) — this skill only assesses motif enrichment against a matched background.
- No permuted control clusters in the same genes are available; comparing against an unmatched genomic background would not reproduce the reported two- to sixfold enrichment range.

## Inputs

- High-confidence RBFOX2 edit cluster BED file (from hc_cluster step)
- Permuted edit cluster BED file from the same genes (GC- and length-matched background)
- GENCODE GRCh38.p13 genome FASTA (GRCh38.p13.genome.fa.gz)

## Outputs

- HOMER de novo motif results (known-motif comparison and enrichment statistics)
- UGCAUG consensus fold-enrichment value over the permuted-cluster background
- Summary table homer_ugcaug_enrichment.tsv mirroring Fig. 2g,h and Supplementary Tables 11-12

## How to apply

Take the high-confidence RBFOX2 edit cluster BED file produced by the hc_cluster step and expand each interval up- and downstream to a 300 nt total window (analogous to the hc_cluster --sloplength expansion). Extract the underlying GRCh38.p13 genomic sequences for both the expanded real clusters and the permuted clusters with pyfaidx from the GENCODE FASTA. Run HOMER's findMotifsGenome.pl on the expanded real-cluster set using the permuted clusters as a GC- and length-matched background. Compare recovered motifs against HOMER's known motif library and quantify UGCAUG consensus fold enrichment relative to the permuted background; the paper reports two- to sixfold enrichment of consensus sequences over permuted control clusters in the same genes. Compile de novo motifs, known-motif statistics, and UGCAUG fold-enrichment values into a summary table mirroring Fig. 2g,h and Supplementary Tables 11-12.

## Related tools

- **HOMER** (De novo motif discovery (findMotifsGenome.pl) on expanded RBFOX2 clusters with the permuted-cluster background, and comparison against the known motif library)
- **pyfaidx** (Extracts GRCh38 genomic sequences underlying the expanded real and permuted cluster intervals for background construction)
- **MAPIT-seq pipeline (hc_cluster step)** (Produces the high-confidence edit clusters and the --sloplength interval-expansion convention used to build the 300 nt motif-search windows) — https://github.com/WangLabPKU/MAPIT-seq

## Examples

```
findMotifsGenome.pl rbfox2_hc_clusters.slopped300.bed hg38 homer_out -len 6 -bg permuted_clusters.slopped300.bed
```

## Evaluation signals

- UGCAUG appears among the top-ranked enriched motifs in the HOMER de novo output
- UGCAUG consensus fold enrichment over the permuted-cluster background falls within the reported two- to sixfold range
- Background clusters are matched to real clusters in GC content and interval length, and are permuted within the same genes
- Extended intervals total 300 nt per cluster, consistent with the hc_cluster --sloplength expansion convention
- Summary table homer_ugcaug_enrichment.tsv contains de novo motifs, known-motif statistics, and fold-enrichment values comparable to Fig. 2g,h and Supplementary Tables 11-12

## Limitations

- The two- to sixfold enrichment range is an empirical range from the paper's RBFOX2 analysis; other RBPs may show different enrichment magnitudes.
- Motif recovery depends on the quality of the high-confidence edit clusters; clusters that fail the hc_cluster filters (Poisson p-value default 0.1, minimum coverage 10) will not support reliable motif statistics.

## Evidence

- [readme] known SNP background handling rationale: [evidence span withheld — anchor status: not_found]
