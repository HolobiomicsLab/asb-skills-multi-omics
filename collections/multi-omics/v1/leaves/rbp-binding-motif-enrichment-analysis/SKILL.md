---
name: rbp-binding-motif-enrichment-analysis
description: Use when MAPIT-seq high-confidence edit cluster BED files (hc_cluster
  output) are available for an RBP such as RBFOX2 and the goal is to de novo discover
  or validate its canonical binding motif (UGCAUG) with fold-enrichment statistics.
when_to_use_negative:
- Input is raw editing sites rather than high-confidence edit clusters — unfiltered
  sites carry background noise and invalid low-coverage events (default coverage filter
  is 10)
- The RBP of interest is not sequence-specific or its binding is primarily mediated
  by co-factors, so motif enrichment over permuted clusters is not expected
- Cluster coordinates and the genome FASTA reference are mismatched (e.g., clusters
  lifted to a different assembly than the GENCODE GRCh38.p13 FASTA used for sequence
  extraction)
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0097
tools:
- name: HOMER
  role: De novo motif discovery (findMotifsGenome.pl) on extended RBFOX2 cluster intervals
    with a GC- and length-matched permuted background, plus known-motif comparison
    and fold-enrichment quantification
- name: pyfaidx
  role: Extracts GRCh38 genomic sequences underlying the extended 300 nt cluster intervals
    from the GENCODE GRCh38.p13 FASTA
- name: MAPIT-seq pipeline
  role: Upstream producer of the high-confidence edit cluster BED file via the hc_cluster
    step (differential editing analysis to call RBP binding targets)
  repo: https://github.com/WangLabPKU/MAPIT-seq
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:1de9bc6e1ac8a4024e2a0b877f30aa54932891a8eec816c499d11cdbb30abf94
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0097
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# rbp-binding-motif-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

De novo and known-motif enrichment analysis on high-confidence RBP edit clusters from MAPIT-seq, testing whether sequence motifs (e.g., RBFOX2 UGCAUG) are recovered above a permuted-cluster background via HOMER, mirroring Fig. 2g,h and Supplementary Tables 11-12.

## When to use

Use when MAPIT-seq high-confidence edit cluster BED files (hc_cluster output) are available for an RBP such as RBFOX2 and the goal is to de novo discover or validate its canonical binding motif (UGCAUG) with fold-enrichment statistics.

## When NOT to use

- Input is raw editing sites rather than high-confidence edit clusters — unfiltered sites carry background noise and invalid low-coverage events (default coverage filter is 10)
- The RBP of interest is not sequence-specific or its binding is primarily mediated by co-factors, so motif enrichment over permuted clusters is not expected
- Cluster coordinates and the genome FASTA reference are mismatched (e.g., clusters lifted to a different assembly than the GENCODE GRCh38.p13 FASTA used for sequence extraction)

## Inputs

- High-confidence RBP edit cluster BED file from the hc_cluster step (e.g., RBFOX2 clusters)
- Permuted control cluster BED file from the same genes (GC- and length-matched background)
- GENCODE GRCh38.p13 genome FASTA (e.g., GRCh38.p13.genome.fa.gz, release 40)
- HOMER known motif library for known-motif comparison

## Outputs

- Extended (300 nt window) cluster interval/sequence set
- HOMER de novo motif discovery output (ranked enriched motifs)
- Known-motif enrichment statistics from HOMER's library comparison
- Summary table of UGCAUG fold-enrichment values (homer_ugcaug_enrichment.tsv)

## How to apply

Start from the high-confidence RBFOX2 edit cluster BED file produced by the hc_cluster step of the MAPIT-seq pipeline and expand each interval to a 300 nt total window (analogous to hc_cluster --sloplength). Extract the underlying GRCh38.p13 sequences with pyfaidx from the GENCODE GRCh38.p13 genome FASTA. Run HOMER's findMotifsGenome.pl on the extended cluster set using a GC- and length-matched background derived from permuted clusters in the same genes, then compare recovered motifs against HOMER's known motif library. Quantify UGCAUG consensus fold enrichment relative to the permuted background and confirm it falls in the reported two- to sixfold range; compile de novo motifs and known-motif statistics into a summary table saved as homer_ugcaug_enrichment.tsv.

## Related tools

- **HOMER** (De novo motif discovery (findMotifsGenome.pl) on extended RBFOX2 cluster intervals with a GC- and length-matched permuted background, plus known-motif comparison and fold-enrichment quantification)
- **pyfaidx** (Extracts GRCh38 genomic sequences underlying the extended 300 nt cluster intervals from the GENCODE GRCh38.p13 FASTA)
- **MAPIT-seq pipeline** (Upstream producer of the high-confidence edit cluster BED file via the hc_cluster step (differential editing analysis to call RBP binding targets)) — https://github.com/WangLabPKU/MAPIT-seq

## Examples

```
findMotifsGenome.pl rbfox2_hc_clusters.slopped.bed hg38 homer_out -len 6 -bg permuted_clusters.slopped.bed
```

## Evaluation signals

- UGCAUG appears among the top-ranked enriched motifs in the HOMER de novo output
- UGCAUG fold enrichment over permuted control clusters falls within the reported two- to sixfold consensus range
- Permuted background clusters are derived from the same genes and are GC- and length-matched to the true clusters
- Extended windows are 300 nt total around each high-confidence cluster interval, consistent with the hc_cluster --sloplength expansion
- Summary table (homer_ugcaug_enrichment.tsv) reproduces the structure of Fig. 2g,h and Supplementary Tables 11-12

## Limitations

- Motif recovery quality depends entirely on the upstream high-confidence cluster step; invalid or low-coverage editing sites (coverage < 10, Poisson p > 0.1) leaking into clusters can dilute enrichment
- Enrichment is measured against permuted clusters within the same genes, so results are sensitive to background construction choices (GC- and length-matching)
- The 300 nt window around edit clusters captures only local sequence; binding sites distant from edited regions cannot be detected by this approach

## Evidence

- [methods] I(**hc_cluster**
  High-confidence edit cluster identification): "I(**hc_cluster**
  High-confidence edit cluster identification)"
