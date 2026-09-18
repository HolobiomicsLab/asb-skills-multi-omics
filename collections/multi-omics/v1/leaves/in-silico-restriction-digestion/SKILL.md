---
name: in-silico-restriction-digestion
description: Use when you need to reproduce or compare in-silico cut-site counts for
  restriction enzymes on a reference genome, e.g. to justify enzyme choice for a Hi-C
  protocol or quantify 'how many fewer sites' one cutter has than another.
when_to_use_negative:
- You need fragment-length distributions or Hi-C QC from actual digested/sequenced
  material (e.g. BAM or .pairs files) rather than genome-wide motif counts — use a
  Hi-C processing pipeline instead.
- Your reference is not the same decoy-containing build (hs37d5/GRCh37); counts are
  build-specific and will not match the reported ~7M/13M/~30M totals on hg38 or non-decoy
  assemblies.
- The enzyme set differs from the MboI/AluI/DpnII/MboII/NlaIII comparison — motif-specific
  counts (e.g. methylation-sensitive or degenerate-site enzymes) are out of scope
  of this workflow.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Python
  role: Streaming the hs37d5 FASTA and scanning for restriction motifs (Biopython
    or str.find) to produce per-enzyme counts and the cutsite_counts.csv output
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:12fd6d10d72fd39888745cede7a5a26c082f7f95100b1b8d7c5fbc6601bce682
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/recombinationlab/sciL3Pipe
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: thin
yes_rate: 0.0
n_claims: 2
---

# in-silico-restriction-digestion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Counts all occurrences of restriction recognition motifs (MboI GATC, AluI AGCT, and the DpnII GATC + MboII GAAGA + NlaIII CATG combination) on the hs37d5 reference genome to quantify how much less frequently sci-L3-Hi-C's MboI cuts than the cutters used by other single-cell Hi-C methods.

## When to use

Use when you need to reproduce or compare in-silico cut-site counts for restriction enzymes on a reference genome, e.g. to justify enzyme choice for a Hi-C protocol or quantify 'how many fewer sites' one cutter has than another.

## When NOT to use

- You need fragment-length distributions or Hi-C QC from actual digested/sequenced material (e.g. BAM or .pairs files) rather than genome-wide motif counts — use a Hi-C processing pipeline instead.
- Your reference is not the same decoy-containing build (hs37d5/GRCh37); counts are build-specific and will not match the reported ~7M/13M/~30M totals on hg38 or non-decoy assemblies.
- The enzyme set differs from the MboI/AluI/DpnII/MboII/NlaIII comparison — motif-specific counts (e.g. methylation-sensitive or degenerate-site enzymes) are out of scope of this workflow.

## Inputs

- hs37d5 reference genome FASTA (GRCh37 with decoy/contig sequences, as used by the sci-L3 pipeline via hybrid_filter: NC_007605,hs37d5)
- Restriction enzyme motif list: MboI GATC, AluI AGCT, DpnII GATC, MboII GAAGA, NlaIII CATG

## Outputs

- cutsite_counts.csv with columns for enzyme name, recognition motif, per-enzyme cut-site count, and percentage reduction versus the combination digest

## How to apply

Obtain the hs37d5 reference FASTA (the decoy-containing GRCh37 build referenced by the sci-L3 pipeline's 'hybrid_filter: NC_007605,hs37d5'). Stream the genome with Python (Biopython or a plain str.find scan) and locate every occurrence of the recognition motifs: MboI GATC, AluI AGCT, DpnII GATC, MboII GAAGA, and NlaIII CATG. Count sites per enzyme; for the combination digest, take the deduplicated union of the three motif position sets so GATC sites shared by DpnII and MboI count once. Compute the relative reduction of MboI and AluI cut sites versus the ~30M-site combination digest to recover the reported 44% and 76% figures, then write enzyme names, motifs, counts, and percentage reductions to cutsite_counts.csv. Validation: reproduced totals must match MboI ~7M, AluI 13M, combination ~30M within rounding, and derived reductions must be 44% and 76%.

## Related tools

- **Python** (Streaming the hs37d5 FASTA and scanning for restriction motifs (Biopython or str.find) to produce per-enzyme counts and the cutsite_counts.csv output)

## Examples

```
python scan_cutsites.py --fasta hs37d5.fa --enzymes MboI:GATC,AluI:AGCT,DpnII:GATC,MboII:GAAGA,NlaIII:CATG --union DpnII+MboII+NlaIII --out cutsite_counts.csv
```

## Evaluation signals

- Reproduced MboI (GATC) count is approximately 7,000,000 on hs37d5 (within rounding of the reported ~7M).
- Reproduced AluI (AGCT) count is 13,000,000 (within rounding of the reported 13M).
- Deduplicated union of DpnII GATC + MboII GAAGA + NlaIII CATG sites totals approximately 30,000,000 (within rounding of the reported ~30M).
- Derived percentage reductions of MboI and AluI versus the combination digest equal 44% and 76%, matching the reported 'more frequent cutters by 2-4x' comparison.
- cutsite_counts.csv contains one row per enzyme with non-negative integer counts and the two percentage-reduction columns, with no duplicate counting of GATC sites shared by DpnII and MboI in the union digest.

## Limitations

- Reported totals are approximate (~7M, 13M, ~30M); exact counts depend on the genome build and will shift on non-hs37d5 references.
- The scan is a plain motif-occurrence count and ignores methylation state or genome accessibility, so counts overestimate accessible cut sites in situ.
- The paper's motivation is a stated limitation: sci-L3-Hi-C uses less frequent cutters than other single-cell Hi-C methods, capturing fewer cis interactions — this skill quantifies that but does not fix the interaction-capture deficit.

## Evidence

- [discussion] Other single-cell Hi-C methods without ligation junction enrichment utilize more frequent cutters by 2-4x, obtaining a higher capture of cis interactions: [evidence span withheld — anchor status: not_found]
- [other] Write the enzyme names, motif, site counts, and percentage reductions to cutsite_counts.csv.: [evidence span withheld — anchor status: not_found]
