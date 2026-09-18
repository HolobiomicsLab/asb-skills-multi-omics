---
name: rna-editing-cluster-interval-extension
description: Use when you have a BED file of high-confidence RNA-editing clusters
  from the MAPIT-seq hc_cluster step and need to convert them into sequence windows
  for de novo RBP-binding motif discovery (e.g. testing RBFOX2 UGCAUG enrichment).
when_to_use_negative:
- Input is raw or unfiltered editing sites rather than high-confidence clusters —
  the sloplength expansion is defined only for hc_cluster output
- You need motif enrichment at transcript level rather than genomic sequence windows
  (pyfaidx extraction is genome-based on GRCh38)
- The RBP of interest has no known or expected motif adjacent to editing sites, making
  consensus-based validation meaningless
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0102
tools:
- name: pyfaidx
  role: Extracts GRCh38 genomic sequences underlying the extended (sloplength-expanded)
    edit cluster intervals
- name: HOMER
  role: Runs de novo motif discovery (findMotifsGenome.pl) on extended clusters against
    a GC- and length-matched permuted-cluster background
- name: MAPIT-seq pipeline (hc_cluster)
  role: Produces the high-confidence edit cluster BED file that is expanded to 300-nt
    windows
  repo: https://github.com/WangLabPKU/MAPIT-seq
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:1cec8bd59a6237247369bd946e216b3919548bd8fdc1a79cf7843ceade1de51e
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
  - MAPIT-seq pipeline (hc_cluster)
  - pyfaidx
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# rna-editing-cluster-interval-extension

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Expands high-confidence MAPIT-seq RNA-editing cluster intervals into fixed-width genomic windows (e.g. 300 nt total) and extracts the underlying GRCh38 sequences so they can serve as input for de novo motif discovery with HOMER, with GC/length-matched permuted clusters as background.

## When to use

Use when you have a BED file of high-confidence RNA-editing clusters from the MAPIT-seq hc_cluster step and need to convert them into sequence windows for de novo RBP-binding motif discovery (e.g. testing RBFOX2 UGCAUG enrichment).

## When NOT to use

- Input is raw or unfiltered editing sites rather than high-confidence clusters — the sloplength expansion is defined only for hc_cluster output
- You need motif enrichment at transcript level rather than genomic sequence windows (pyfaidx extraction is genome-based on GRCh38)
- The RBP of interest has no known or expected motif adjacent to editing sites, making consensus-based validation meaningless

## Inputs

- High-confidence RBFOX2 edit cluster BED file from the MAPIT-seq hc_cluster step
- GENCODE GRCh38.p13 (release 40) genome FASTA (GRCh38.p13.genome.fa.gz)
- Permuted control cluster BED file (same genes, GC/length-matched)

## Outputs

- Extended 300-nt interval BED file
- FASTA of extended cluster sequences (extracted via pyfaidx)
- HOMER de novo motif output and known-motif enrichment statistics
- Summary table of UGCAUG fold-enrichment values (homer_ugcaug_enrichment.tsv)

## How to apply

Take the high-confidence edit cluster BED file produced by the MAPIT-seq hc_cluster step (high-confidence edit cluster identification, which follows GATK-based callediting and Poisson/coverage filtering of editing sites) and expand each cluster interval symmetrically up- and downstream so the total window length is 300 nt, mirroring the hc_cluster --sloplength expansion used in the paper. Extract the genomic sequences of the extended intervals from the GENCODE GRCh38.p13 FASTA using pyfaidx. Build a GC- and length-matched background from permuted clusters in the same genes and run HOMER findMotifsGenome.pl on both sets. The rationale is that editing events introduced adjacent to antibody-bound RBP sites cluster around the true binding site, so widening the interval captures the flanking motif context (e.g. the RBFOX2 UGCAUG consensus) without diluting it. Finally, compare recovered motifs against HOMER's known motif library and quantify the fold enrichment of the consensus (two- to sixfold over permuted controls in the published RBFOX2 result).

## Related tools

- **pyfaidx** (Extracts GRCh38 genomic sequences underlying the extended (sloplength-expanded) edit cluster intervals)
- **HOMER** (Runs de novo motif discovery (findMotifsGenome.pl) on extended clusters against a GC- and length-matched permuted-cluster background)
- **MAPIT-seq pipeline (hc_cluster)** (Produces the high-confidence edit cluster BED file that is expanded to 300-nt windows) — https://github.com/WangLabPKU/MAPIT-seq

## Examples

```
from pyfaidx import Fasta; fa = Fasta('GRCh38.p13.genome.fa'); seqs = {iv: fa[iv[0]][max(iv[1]-150,0):iv[2]+150].seq for iv in clusters}
```

## Evaluation signals

- UGCAUG appears among the top-ranked enriched motifs in the HOMER known-motif output
- UGCAUG consensus fold enrichment over permuted clusters falls within the reported two- to sixfold range
- Every extended interval has exactly 300 nt total window length after symmetric up/downstream expansion
- Extracted FASTA sequences match the GRCh38.p13 reference at the expected coordinates
- Summary table (homer_ugcaug_enrichment.tsv) contains de novo motifs, known-motif statistics, and fold-enrichment values mirroring Fig. 2g,h and Supplementary Tables 11-12

## Limitations

- The two- to sixfold enrichment range is calibrated on RBFOX2/UGCAUG; other RBPs may show different enrichment magnitudes
- Background quality depends on permuted clusters being drawn from the same genes and being GC- and length-matched; mismatched background inflates or masks enrichment
- hc_cluster output depends on upstream filtering thresholds (default coverage 10, Poisson p-value 0.1 for editing sites), which propagate into motif discovery sensitivity

## Evidence

- [other] Load the high-confidence RBFOX2 edit cluster BED file produced by the hc_cluster step and expand each cluster interval up- and downstream to 300 nt total window length: [evidence span withheld — anchor status: not_found]
- [other] Extract the underlying GRCh38 genomic sequences for the extended intervals with pyfaidx from the GENCODE GRCh38.p13 FASTA: [evidence span withheld — anchor status: not_found]
