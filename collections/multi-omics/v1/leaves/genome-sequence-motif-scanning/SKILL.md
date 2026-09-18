---
name: genome-sequence-motif-scanning
description: Use when you need to verify or reproduce claims about restriction enzyme
  cut-site counts on a reference genome — e.g., comparing how frequently different
  single-cell Hi-C protocols (sci-L3-Hi-C's MboI vs. s3-GCC's AluI vs. droplet Hi-C's
  DpnII+MboII+NlaIII) fragment the genome.
when_to_use_negative:
- You need fragment lengths or in vitro digestion products rather than motif occurrence
  counts — this skill only counts recognition sites, not simulated digest fragments.
- The genome build differs from the one used in the claim being validated — cut-site
  totals (~7M MboI, 13M AluI, ~30M combination) are specific to hs37d5/GRCh37 and
  will not match on hg38.
- Motif occurrences depend on CpG methylation context (e.g., DpnII requires methylated
  GATC in vivo) — this in-silico scan ignores methylation sensitivity.
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0080
tools:
- name: Python
  role: Streams the reference FASTA and counts motif occurrences (Biopython or plain
    str.find scan) to compute cut-site totals and percentage reductions
  repo: github.com/recombinationlab/sciL3Pipe
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:0199c32396f7657cfbf0fc8e197f1a895f5cb9b73c0f60fc798a8daa26b479ad
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
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.2857142857142857
n_claims: 7
---

# Genome sequence motif scanning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Streams a reference genome FASTA (e.g., the hs37d5 decoy-containing GRCh37 build used by sci-L3 via hybrid_filter 'NC_007605,hs37d5') and counts occurrences of restriction recognition motifs (MboI GATC, AluI AGCT, MboII GAAGA, NlaIII CATG) to quantify and compare in silico cut-site densities across Hi-C methods.

## When to use

Use when you need to verify or reproduce claims about restriction enzyme cut-site counts on a reference genome — e.g., comparing how frequently different single-cell Hi-C protocols (sci-L3-Hi-C's MboI vs. s3-GCC's AluI vs. droplet Hi-C's DpnII+MboII+NlaIII) fragment the genome.

## When NOT to use

- You need fragment lengths or in vitro digestion products rather than motif occurrence counts — this skill only counts recognition sites, not simulated digest fragments.
- The genome build differs from the one used in the claim being validated — cut-site totals (~7M MboI, 13M AluI, ~30M combination) are specific to hs37d5/GRCh37 and will not match on hg38.
- Motif occurrences depend on CpG methylation context (e.g., DpnII requires methylated GATC in vivo) — this in-silico scan ignores methylation sensitivity.

## Inputs

- Reference genome FASTA (hs37d5 decoy-containing GRCh37 build, matching sci-L3's hybrid_filter: NC_007605,hs37d5)
- Restriction enzyme-to-motif mapping (MboI=GATC, AluI=AGCT, DpnII=GATC, MboII=GAAGA, NlaIII=CATG)

## Outputs

- cutsite_counts.csv with columns for enzyme name, recognition motif, site count, and percentage reduction relative to the combination digest
- Deduplicated union set of motif positions for the DpnII+MboII+NlaIII combination digest (~30M sites)

## How to apply

Obtain the hs37d5 reference FASTA (the decoy-containing GRCh37 build the sci-L3 pipeline uses via 'hybrid_filter: NC_007605,hs37d5'). Stream the genome with Python (Biopython or a plain str.find scan) and record every occurrence of each restriction recognition motif: MboI/DpnII GATC, AluI AGCT, MboII GAAGA, and NlaIII CATG. For a multi-enzyme combination digest (DpnII+MboII+NlaIII, ~30M sites), take the deduplicated union of the per-motif position sets so that GATC sites shared by DpnII and MboI count once. Compute relative reductions — MboI (~7M sites) cuts 44% less frequently than s3-GCC's ~13M AluI sites and 76% less frequently than the ~30M combination digest, consistent with the paper's statement that other single-cell Hi-C methods use cutters more frequent by 2-4x. Write enzyme names, motifs, site counts, and percentage reductions to cutsite_counts.csv.

## Related tools

- **Python** (Streams the reference FASTA and counts motif occurrences (Biopython or plain str.find scan) to compute cut-site totals and percentage reductions) — github.com/recombinationlab/sciL3Pipe

## Examples

```
python -c "import gzip; seq=''.join(l.strip() for l in open('hs37d5.fa') if not l.startswith('>')); print(seq.count('GATC'), seq.count('AGCT'), seq.count('GAAGA'), seq.count('CATG'))"
```

## Evaluation signals

- Reproduced MboI (GATC) count is ~7M sites on hs37d5, within rounding of the reported total.
- Reproduced AluI (AGCT) count is ~13M sites, matching the reported s3-GCC comparison value.
- Deduplicated union of DpnII (GATC), MboII (GAAGA), and NlaIII (CATG) sites totals ~30M.
- Derived percentage reductions are 44% (MboI vs. AluI) and 76% (MboI vs. combination digest).
- GATC sites shared between DpnII and MboI are counted once in the combination digest, not double-counted.

## Limitations

- Counts are build-specific: the reported ~7M/13M/~30M totals apply to the hs37d5 decoy-containing GRCh37 build, not hg38, which the sci-L3 pipeline separately lists for bowtie2 alignment.
- The 2-4x less-frequent cutter limitation means sci-L3-Hi-C captures fewer cis interactions than methods using more frequent cutters — the motif counts alone do not capture ligation or capture efficiency.
- The in-silico scan ignores methylation sensitivity of enzymes (e.g., DpnII vs. MboI share the GATC motif but differ in methylation dependence).

## Evidence

- [discussion] less frequent cutters in other single-cell Hi-C methods: [evidence span withheld — anchor status: not_found]
- [methods] Python tooling in the sci-L3 pipeline: [evidence span withheld — anchor status: not_found]
