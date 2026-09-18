---
name: homer-de-novo-motif-discovery
description: Use when you have a BED file of high-confidence RBP edit clusters (e.g.,
  from the MAPIT-seq hc_cluster step) and need to de novo discover and validate the
  RBP's known binding motif against a permuted control background.
when_to_use_negative:
- Input is raw sequencing reads or unfiltered editing sites rather than high-confidence
  edit clusters — run the MAPIT-seq callediting/calltargets/hc_cluster steps first.
- The RBP has no prior consensus motif to validate against and you only need differential
  editing target calling, not motif discovery.
- The clusters are not from an editing-based RBP targeting assay (e.g., CLIP peaks
  with different geometry) without adjusting window expansion, since the 300 nt slop
  and permuted background are calibrated for MAPIT-seq edit clusters.
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0160
tools:
- name: HOMER
  role: De novo motif discovery (findMotifsGenome.pl) on extended edit clusters and
    comparison against its known motif library
- name: pyfaidx
  role: Extract underlying GRCh38 genomic sequences for the expanded edit-cluster
    intervals
- name: MAPIT-seq pipeline
  role: Produces the high-confidence edit clusters (hc_cluster step) that serve as
    motif-discovery input
  repo: https://github.com/WangLabPKU/MAPIT-seq
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:01d2dd62e9d3a2a98400833d52965e2178dfde6561d42521ad74cf0576dd9d99
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
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# homer-de-novo-motif-discovery

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

De novo discovery of RBP-binding motifs (e.g., the canonical RBFOX2 UGCAUG motif) from high-confidence MAPIT-seq edit clusters using HOMER's findMotifsGenome.pl, with a GC- and length-matched permuted-cluster background and pyfaidx-extracted GRCh38 sequences.

## When to use

Use when you have a BED file of high-confidence RBP edit clusters (e.g., from the MAPIT-seq hc_cluster step) and need to de novo discover and validate the RBP's known binding motif against a permuted control background.

## When NOT to use

- Input is raw sequencing reads or unfiltered editing sites rather than high-confidence edit clusters — run the MAPIT-seq callediting/calltargets/hc_cluster steps first.
- The RBP has no prior consensus motif to validate against and you only need differential editing target calling, not motif discovery.
- The clusters are not from an editing-based RBP targeting assay (e.g., CLIP peaks with different geometry) without adjusting window expansion, since the 300 nt slop and permuted background are calibrated for MAPIT-seq edit clusters.

## Inputs

- High-confidence RBFOX2 edit cluster BED file (output of MAPIT-seq hc_cluster step)
- Permuted edit-cluster BED file from the same genes (GC- and length-matched background)
- GENCODE GRCh38.p13 genome FASTA (e.g., GRCh38.p13.genome.fa.gz)
- pyfaidx / HOMER known motif library for motif comparison

## Outputs

- HOMER de novo motif results (knownResults/ and homerMotifs outputs)
- Fold-enrichment table of the UGCAUG consensus vs. permuted-cluster background
- Summary table homer_ugcaug_enrichment.tsv (de novo motifs, known-motif enrichment statistics, UGCAUG fold enrichment)

## How to apply

Start from the high-confidence edit-cluster BED file produced by the MAPIT-seq hc_cluster step and expand each interval to a 300 nt total window (sloplength-style up- and downstream padding). Extract the underlying GRCh38 sequences with pyfaidx from the GENCODE GRCh38.p13 FASTA. Run HOMER findMotifsGenome.pl on the extended clusters using a background derived from permuted clusters matched for GC content and interval length. Compare recovered de novo motifs against HOMER's known motif library and quantify fold enrichment of the expected consensus (for RBFOX2, UGCAUG) relative to the permuted-cluster background. Compile motifs, known-motif statistics, and fold-enrichment values into a summary table (e.g., homer_ugcaug_enrichment.tsv) mirroring the paper's Fig. 2g,h and Supplementary Tables 11-12.

## Related tools

- **HOMER** (De novo motif discovery (findMotifsGenome.pl) on extended edit clusters and comparison against its known motif library)
- **pyfaidx** (Extract underlying GRCh38 genomic sequences for the expanded edit-cluster intervals)
- **MAPIT-seq pipeline** (Produces the high-confidence edit clusters (hc_cluster step) that serve as motif-discovery input) — https://github.com/WangLabPKU/MAPIT-seq

## Examples

```
findMotifsGenome.pl rbfox2_clusters.extended300.bed hg38 homer_out -len 6 -bg permuted_clusters_matched.bed
```

## Evaluation signals

- UGCAUG appears among the top-ranked enriched motifs in the HOMER knownResults output.
- UGCAUG fold enrichment over the permuted-cluster background falls within the reported two- to sixfold range.
- Permuted control clusters from the same genes yield no comparable consensus enrichment (two- to sixfold less).
- Extended cluster intervals are consistently 300 nt total window length before sequence extraction.
- Summary table contains de novo motif ranks, known-motif enrichment statistics, and UGCAUG fold-enrichment values as in Supplementary Tables 11-12.

## Limitations

- Fold enrichment is validated for RBFOX2's UGCAUG motif; other RBPs may show different enrichment magnitudes not covered by the two- to sixfold range.
- Background matching (GC and length) against permuted clusters is essential; mismatches can produce spurious enrichment.
- Motif discovery depends on the quality of hc_cluster output — low-confidence clusters or insufficient coverage will dilute the motif signal.

## Evidence

- [intro] De novo discovery of RBP-binding motifs via HOMER on extended MAPIT-seq edit clusters: "pinpoint high-confidence RBP-binding regions and *de novo* discover RBP-binding motifs"
- [results] MAPIT-seq edit clusters showed two- to sixfold UGCAUG consensus enrichment over permuted control clusters: "MAPIT-seq edit clusters consistently showed two- to sixfold enrichment of consensus sequences over permuted control clusters in the same genes"
- [methods] pyfaidx is used to extract genomic sequences for motif discovery: "pip install deeptools gffutils pyfaidx Bio"
- [methods] GRCh38.p13 genome FASTA used as sequence reference: "wget https://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_40/GRCh38.p13.genome.fa.gz"
