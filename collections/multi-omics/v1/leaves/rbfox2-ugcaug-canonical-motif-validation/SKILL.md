---
name: rbfox2-ugcaug-canonical-motif-validation
description: Use when high-confidence edit-cluster BED intervals from MAPIT-seq are
  available for an RBP (e.g., RBFOX2) and you must confirm de novo motif recovery
  against a known consensus like UGCAUG.
when_to_use_negative:
- You lack high-confidence edit clusters or a permuted-cluster background — motif
  enrichment cannot be benchmarked against matched controls.
- The target RBP has no known consensus motif to validate against (this skill is a
  positive-control/validation check for a known motif such as UGCAUG).
- Your genome annotation/build differs (e.g., not GRCh38.p13), since sequence extraction
  and HOMER genome selection must match the original FASTA.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0749
tools:
- name: HOMER
  role: De novo motif discovery (findMotifsGenome.pl) on extended RBFOX2 edit clusters
    with a GC/length-matched permuted-cluster background
- name: pyfaidx
  role: Extracts GRCh38.p13 genomic sequences underlying the 300-nt extended cluster
    intervals for motif analysis
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: MAPIT-seq pipeline
  role: Produces the high-confidence edit cluster BED file (hc_cluster step) that
    serves as motif-discovery input
  repo: https://github.com/WangLabPKU/MAPIT-seq
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:10adacd9ebfbb7637a5c70d472cba3d0b0478c66a97a8acb70fa6136d6c34649
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
  - MAPIT-seq pipeline
  - pyfaidx
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.25
n_claims: 4
---

# rbfox2-ugcaug-canonical-motif-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that MAPIT-seq edit clusters recovered by the hc_cluster step yield the canonical RBFOX2 UGCAUG motif via HOMER de novo motif discovery on 300-nt extended clusters, using GC/length-matched permuted-cluster backgrounds and requiring two- to sixfold enrichment of the consensus (Fig. 2g,h; Supplementary Tables 11-12).

## When to use

Use when high-confidence edit-cluster BED intervals from MAPIT-seq are available for an RBP (e.g., RBFOX2) and you must confirm de novo motif recovery against a known consensus like UGCAUG.

## When NOT to use

- You lack high-confidence edit clusters or a permuted-cluster background — motif enrichment cannot be benchmarked against matched controls.
- The target RBP has no known consensus motif to validate against (this skill is a positive-control/validation check for a known motif such as UGCAUG).
- Your genome annotation/build differs (e.g., not GRCh38.p13), since sequence extraction and HOMER genome selection must match the original FASTA.

## Inputs

- High-confidence RBFOX2 edit cluster BED file from the MAPIT-seq hc_cluster step
- Permuted control cluster BED intervals within the same genes
- GENCODE GRCh38.p13 genome FASTA (release 40, GRCh38.p13.genome.fa.gz)

## Outputs

- HOMER de novo motif output (known-motif and de novo tables) for the extended RBFOX2 cluster set
- UGCAUG fold-enrichment value over permuted clusters
- Summary table homer_ugcaug_enrichment.tsv (de novo motifs, known-motif enrichment statistics, UGCAUG fold enrichment)

## How to apply

Take the high-confidence RBFOX2 edit cluster BED file emitted by the MAPIT-seq hc_cluster step (analogous to hc_cluster --sloplength expansion) and expand each interval so the total window is 300 nt. Extract the underlying genomic sequence for these extended intervals from the GENCODE GRCh38.p13 FASTA (release 40) with pyfaidx. Run HOMER findMotifsGenome.pl on the extended RBFOX2 clusters with a background set derived from permuted clusters matched for GC content and length, then compare recovered motifs against HOMER's known motif library. Quantify fold enrichment of the UGCAUG consensus relative to the permuted-cluster background and verify it falls within the reported two- to sixfold range. Compile de novo motifs, known-motif statistics, and UGCAUG fold-enrichment values into a summary table (homer_ugcaug_enrichment.tsv) mirroring Fig. 2g,h and Supplementary Tables 11-12.

## Related tools

- **HOMER** (De novo motif discovery (findMotifsGenome.pl) on extended RBFOX2 edit clusters with a GC/length-matched permuted-cluster background)
- **pyfaidx** (Extracts GRCh38.p13 genomic sequences underlying the 300-nt extended cluster intervals for motif analysis) — https://github.com/WangLabPKU/MAPIT-seq
- **MAPIT-seq pipeline** (Produces the high-confidence edit cluster BED file (hc_cluster step) that serves as motif-discovery input) — https://github.com/WangLabPKU/MAPIT-seq

## Examples

```
findMotifsGenome.pl rbfox2_hc_clusters_300nt.bed hg38 homer_out/ -len 6 -bg permuted_clusters_300nt.bed
```

## Evaluation signals

- UGCAUG appears among the top-ranked enriched motifs in the HOMER findMotifsGenome.pl output for the RBFOX2 cluster set.
- UGCAUG consensus fold enrichment over permuted control clusters falls within the reported two- to sixfold range.
- Permuted-cluster background is matched for GC content and window length to the real RBFOX2 cluster set.
- Extended cluster windows total 300 nt per cluster (up- and downstream expansion of hc_cluster intervals).
- homer_ugcaug_enrichment.tsv contains de novo motifs, known-motif enrichment statistics, and UGCAUG fold-enrichment values mirroring Fig. 2g,h and Supplementary Tables 11-12.

## Limitations

- Enrichment is benchmarked against permuted clusters in the same genes, so validation depends on the quality of the high-confidence cluster calls from the hc_cluster step.
- Recovery of the canonical motif only demonstrates motif-level fidelity of MAPIT-seq clusters; it does not by itself validate binding-site resolution.
- The reported two- to sixfold enrichment range is specific to the RBFOX2/UGCAUG case analyzed in the paper; other RBPs may show different enrichment magnitudes.

## Evidence

- [results] HC cluster validation via two- to sixfold UGCAUG enrichment over permuted control clusters: [evidence span withheld — anchor status: not_found]
