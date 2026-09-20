---
name: rna-seq-count-filtering-and-normalization
description: Use when raw count matrices (e.g. MOSim-simulated RNA-seq or miRNA-seq
  counts, or experimental bulk counts) must be filtered for low-expression features
  and TMM-normalized before differential expression analysis or regulatory-network
  inference.
when_to_use_negative:
- 'Input is single-cell simulated data: MOSim''s sc_mosim module leverages SPARSim
  and is restricted to scRNA-seq and scATAC-seq, where bulk TMM/voom pipelines are
  not the intended processing route.'
- Input is already a normalized or transformed expression matrix (e.g. voom-transformed
  or log(x + 1)-transformed data), where re-filtering/re-normalizing would distort
  the values.
- Input is a non-count regulatory omics layer such as methylation or accessibility
  data rather than RNA-seq/miRNA-seq count matrices.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: MOSim
  role: Generates the raw bulk RNA-seq/miRNA-seq count matrices (via the mosim function)
    that are subsequently filtered and normalized
  repo: github.com/ConesaLab/MOSim
- name: NOISeq
  role: Performs low-expression count filtering of the simulated RNA-seq counts prior
    to normalization
- name: limma
  role: Applies voom transformation and differential expression analysis on the TMM-normalized
    counts
- name: R
  role: Language in which the filtering, normalization, voom and limma steps are executed
    (Bioconductor workflow)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/421834
    title: 'MOSim: bulk and single-cell multi-layer regulatory network simulator'
  source_content_hash: sha256:d5370d5f19802762e569e5e63223669290ece750aeb3ce63e6b38ed0bae478cd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ConesaLab/MOSim
  tools:
  - MORE
  - MOSim
  - NOISeq
  - R
  - limma
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# rna-seq-count-filtering-and-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-process RNA-seq/miRNA-seq count matrices — including counts produced by the MOSim bulk simulator — by removing low-expression features with NOISeq filtering and applying TMM normalization, so that downstream voom/limma differential expression and GRN benchmarking operate on comparable, well-conditioned count data.

## When to use

Use when raw count matrices (e.g. MOSim-simulated RNA-seq or miRNA-seq counts, or experimental bulk counts) must be filtered for low-expression features and TMM-normalized before differential expression analysis or regulatory-network inference.

## When NOT to use

- Input is single-cell simulated data: MOSim's sc_mosim module leverages SPARSim and is restricted to scRNA-seq and scATAC-seq, where bulk TMM/voom pipelines are not the intended processing route.
- Input is already a normalized or transformed expression matrix (e.g. voom-transformed or log(x + 1)-transformed data), where re-filtering/re-normalizing would distort the values.
- Input is a non-count regulatory omics layer such as methylation or accessibility data rather than RNA-seq/miRNA-seq count matrices.

## Inputs

- Raw bulk RNA-seq count matrix (e.g. MOSim mosim simulation output with 30M reads, 2 experimental groups, 20 time points)
- Raw miRNA-seq count matrix from the same MOSim simulation
- TF (transcription factor) regulatory matrix from the MOSim simulation
- Experimental design metadata (groups, time points, replicates)

## Outputs

- Filtered count matrix (low-expression features removed by NOISeq filtering)
- TMM-normalized count matrix
- Voom-transformed expression matrix
- limma differential expression results per feature

## How to apply

Starting from the simulated or experimental raw count matrices (e.g. MOSim output generated with the mosim function for RNA-seq, miRNA-seq and TF layers), first apply NOISeq-based low-count filtering to remove features with negligible counts, retaining features informative for group comparisons (the benchmark uses 50% DEGs and 60% significant regulations). Then apply TMM normalization to the filtered counts to correct for library-size and composition differences between the 2 experimental groups. Transform the normalized counts with voom so mean-variance weights are estimated for linear modelling, then run limma differential expression analysis on the voom-transformed data. Rationale: MOSim bulk simulation is based on the negative binomial distribution, so counts behave like standard RNA-seq data and standard filter-then-TMM-then-voom/limma processing is appropriate before comparing inferred regulatory edges against simulated ground truth.

## Related tools

- **MOSim** (Generates the raw bulk RNA-seq/miRNA-seq count matrices (via the mosim function) that are subsequently filtered and normalized) — github.com/ConesaLab/MOSim
- **NOISeq** (Performs low-expression count filtering of the simulated RNA-seq counts prior to normalization)
- **limma** (Applies voom transformation and differential expression analysis on the TMM-normalized counts)
- **R** (Language in which the filtering, normalization, voom and limma steps are executed (Bioconductor workflow))

## Examples

```
BiocManager::install("MOSim")
```

## Evaluation signals

- Filtered matrix contains no features with near-zero counts across all 2 experimental groups / 20 time points.
- TMM normalization factors differ from 1 for libraries with composition bias, and normalized counts are used (not raw) for voom.
- Voom transformation produces mean-variance trend weights suitable for limma linear modelling.
- limma DE output is consistent with the simulation settings (e.g. ~50% DEGs in the MOSim benchmark design).
- Downstream GRN benchmarking against MOSim ground truth (vs 20,000 random feature pairs with no simulated regulatory effects) yields sensitivity ≈ 85.5% and F1 ≈ 62.9% for MORE with PLS1.

## Limitations

- MOSim operates at the count matrix level rather than simulating read data, so filtering/normalization must operate on count matrices, not FASTQ.
- MOSim software version is not stated in the article sections, complicating reproducibility of the simulated counts being filtered.
- The seed dataset source for MOSim simulation is not identified (no named dataset or accession) in the discussion section.

## Evidence
