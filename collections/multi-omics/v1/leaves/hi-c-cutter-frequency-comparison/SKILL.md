---
name: hi-c-cutter-frequency-comparison
description: Use when comparing restriction enzyme cutting frequencies across single-cell
  Hi-C protocols, or when reproducing the reported MboI (~7M), AluI (13M), and combination-digest
  (~30M) cut-site totals on the hs37d5/GRCh37 reference to explain differences in
  cis-interaction capture.
when_to_use_negative:
- The genome build differs from the hs37d5/GRCh37 decoy build used by the sci-L3 pipeline
  — motif counts will not match the reported ~7M/13M/30M totals.
- The question is about observed ligation junctions or Hi-C contact maps from sequencing
  data, not in-silico motif counting on a reference FASTA.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3366
tools:
- name: Python
  role: Streams the hs37d5 FASTA and scans for restriction recognition motifs to count
    cut sites per enzyme
  repo: https://github.com/recombinationlab/sciL3Pipe
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:f790ba5753a35c7e741a6a3d7be5f45a1082b56af0f8de3f2c1a22ffeca945ca
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/recombinationlab/sciL3Pipe
  repo_url: https://github.com/recombinationlab/sciL3Pipe
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# hi-c-cutter-frequency-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the in-silico restriction cut-site counts used to justify sci-L3-Hi-C's cutter choice: counting MboI (GATC), AluI (AGCT), and the DpnII+MboII+NlaIII combination motif sites on hs37d5 to quantify how much less frequently sci-L3-Hi-C cuts relative to other single-cell Hi-C methods.

## When to use

Use when comparing restriction enzyme cutting frequencies across single-cell Hi-C protocols, or when reproducing the reported MboI (~7M), AluI (13M), and combination-digest (~30M) cut-site totals on the hs37d5/GRCh37 reference to explain differences in cis-interaction capture.

## When NOT to use

- The genome build differs from the hs37d5/GRCh37 decoy build used by the sci-L3 pipeline — motif counts will not match the reported ~7M/13M/30M totals.
- The question is about observed ligation junctions or Hi-C contact maps from sequencing data, not in-silico motif counting on a reference FASTA.

## Inputs

- hs37d5 reference genome FASTA (decoy-containing GRCh37 build, as configured by hybrid_filter: NC_007605,hs37d5)
- Restriction enzyme motif definitions (MboI GATC, AluI AGCT, DpnII GATC, MboII GAAGA, NlaIII CATG)

## Outputs

- cutsite_counts.csv with enzyme names, recognition motifs, per-enzyme site counts, and percentage reductions versus the combination digest

## How to apply

Obtain the hs37d5 reference FASTA — the same decoy-containing GRCh37 build the sci-L3 pipeline uses via 'hybrid_filter: NC_007605,hs37d5'. Stream the genome with Python (Biopython or a plain str.find scan) and locate every occurrence of the restriction motifs: MboI GATC, AluI AGCT, and the combination DpnII GATC, MboII GAAGA, and NlaIII CATG. For the combination digest, take the deduplicated union of the three motif position sets so GATC sites shared by DpnII and MboI count once. Compute the relative reduction of MboI versus the ~30M-site combination digest (expected 76% fewer cuts) and of AluI versus the same baseline (expected 44% fewer cuts), recovering the paper's 'more frequent cutters by 2-4x' comparison with s3-GCC and droplet Hi-C. Write enzyme names, motifs, site counts, and percentage reductions to cutsite_counts.csv.

## Related tools

- **Python** (Streams the hs37d5 FASTA and scans for restriction recognition motifs to count cut sites per enzyme) — https://github.com/recombinationlab/sciL3Pipe

## Examples

```
python -c "import Bio; seq=str(next(Bio.SeqIO.parse('hs37d5.fa','fasta')).seq); print(seq.count('GATC'), seq.count('AGCT'))"
```

## Evaluation signals

- MboI (GATC) count reproduces ~7M sites on hs37d5 within rounding.
- AluI (AGCT) count reproduces the reported 13M sites.
- Deduplicated union of DpnII (GATC) + MboII (GAAGA) + NlaIII (CATG) sites reproduces the ~30M combination digest total.
- Derived percentage reductions equal 44% (AluI vs combination) and 76% (MboI vs combination).
- GATC sites shared by DpnII and MboI are counted exactly once in the combination digest (deduplication invariant).

## Limitations

- The 2-4x more frequent cutters used by other single-cell Hi-C methods yield a higher capture of cis interactions, so sci-L3-Hi-C's less frequent MboI cutter captures fewer cis interactions — a stated limitation of the method.
- Counts are in-silico motif frequencies on the reference and do not account for chromatin accessibility, methylation sensitivity, or ligation efficiency in actual libraries.

## Evidence

- [discussion] Other single-cell Hi-C methods without ligation junction enrichment utilize more frequent cutters by 2-4x, obtaining a higher capture of cis interactions: [evidence span withheld — anchor status: not_found]
