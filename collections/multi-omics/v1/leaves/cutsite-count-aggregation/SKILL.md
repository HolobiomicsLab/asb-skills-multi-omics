---
name: cutsite-count-aggregation
description: Use when benchmarking Hi-C protocol choice by restriction motif frequency
  — e.g. reproducing reported cut-site totals (MboI ~7M, AluI 13M, combination ~30M)
  on hs37d5 or comparing cutter frequency across single-cell Hi-C methods.
when_to_use_negative:
- When the goal is empirical fragment coverage or observed cut-site usage from sequenced
  Hi-C data rather than genome-wide in-silico motif counts on the reference
- When the genome is hg38/GRCh38 (the sci-L3Pipe bowtie2 index genome) rather than
  the hs37d5 GRCh37 hybrid-filter reference — counts will not match the reported totals
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0625
tools:
- name: Python
  role: Streaming the FASTA with Biopython or str.find to locate and count restriction
    motif occurrences and compute percentage reductions
- name: juicer
  role: Reference Hi-C pipeline whose setup requires specifying a restriction site
    such as MboI, illustrating where cut-site annotation feeds into Hi-C processing
  repo: https://github.com/aidenlab/juicer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:c9dd7c427bf66f1a79bd52e153ae86c26fa7e5ebba73fd5372a32b25e2469409
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/recombinationlab/sciL3Pipe
  repo_url: https://github.com/aidenlab/juicer
  tools:
  - Python
  - juicer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 3
---

# cutsite-count-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Enumerates in-silico restriction enzyme cut sites (MboI GATC, AluI AGCT, and the DpnII GATC + MboII GAAGA + NlaIII CATG combination) on the hs37d5 decoy-containing GRCh37 reference to quantify how much less frequently sci-L3-Hi-C cuts compared with other single-cell Hi-C methods.

## When to use

Use when benchmarking Hi-C protocol choice by restriction motif frequency — e.g. reproducing reported cut-site totals (MboI ~7M, AluI 13M, combination ~30M) on hs37d5 or comparing cutter frequency across single-cell Hi-C methods.

## When NOT to use

- When the goal is empirical fragment coverage or observed cut-site usage from sequenced Hi-C data rather than genome-wide in-silico motif counts on the reference
- When the genome is hg38/GRCh38 (the sci-L3Pipe bowtie2 index genome) rather than the hs37d5 GRCh37 hybrid-filter reference — counts will not match the reported totals

## Inputs

- hs37d5 decoy-containing GRCh37 reference FASTA (same build used by sci-L3Pipe via hybrid_filter: NC_007605,hs37d5)
- Restriction enzyme motif definitions: MboI GATC, AluI AGCT, DpnII GATC, MboII GAAGA, NlaIII CATG

## Outputs

- cutsite_counts.csv with columns for enzyme name, recognition motif, per-enzyme site count, and percentage reduction versus the DpnII+MboII+NlaIII combination digest
- Deduplicated union position set for the DpnII/MboII/NlaIII combination digest (~30M sites)

## How to apply

Obtain the hs37d5 reference FASTA that the sci-L3 pipeline uses via 'hybrid_filter: NC_007605,hs37d5', then stream the genome in Python (Biopython or a plain str.find scan) and record positions of every MboI (GATC), AluI (AGCT), DpnII (GATC), MboII (GAAGA), and NlaIII (CATG) motif. For the combination digest, take the deduplicated union of the three motif position sets, counting GATC sites shared by DpnII and MboI only once. Convert each motif hit set to counts, then compute MboI and AluI percentage reductions relative to the ~30M-site combination digest, recovering the reported 44% and 76% figures. Write enzyme names, motifs, site counts, and percentage reductions to cutsite_counts.csv. The rationale comes from the paper's limitation that other single-cell Hi-C methods without ligation junction enrichment use cutters 2-4x more frequent than sci-L3-Hi-C's MboI, capturing more cis interactions.

## Related tools

- **Python** (Streaming the FASTA with Biopython or str.find to locate and count restriction motif occurrences and compute percentage reductions)
- **juicer** (Reference Hi-C pipeline whose setup requires specifying a restriction site such as MboI, illustrating where cut-site annotation feeds into Hi-C processing) — https://github.com/aidenlab/juicer

## Examples

```
python -c "s=open('hs37d5.fa').read(); mboi=s.count('GATC'); alui=s.count('AGCT'); combo=len(set(i for m in ['GATC','GAAGA','CATG'] for i in [s.find(m,i+1) for i in range(len(s))]) )"
```

## Evaluation signals

- MboI GATC count is ~7M on hs37d5, within rounding of the reported total
- AluI AGCT count is ~13M, matching the s3-GCC cut-site total
- Deduplicated DpnII+MboII+NlaIII union is ~30M sites; DpnII and MboI share GATC and each shared site is counted once
- Derived percentage reductions are 44% (MboI vs combination) and 76% (AluI vs combination), consistent with the 2-4x less-frequent-cutter claim
- cutsite_counts.csv contains exactly the enzyme names, motifs, counts, and percentage reductions without duplicated union positions

## Limitations

- The comparison captures only motif-frequency differences; other single-cell Hi-C methods still obtain higher cis-interaction capture because of their more frequent cutters (2-4x)
- Counts are reference-build-specific and sensitive to decoy/contig content — using a genome other than hs37d5 will not reproduce the reported totals
- In-silico GATC counting does not model methylation sensitivity (MboI vs DpnII) or incomplete digestion

## Evidence

- [discussion] Limitation: sci-L3-Hi-C uses less frequent cutters: [evidence span withheld — anchor status: not_found]
