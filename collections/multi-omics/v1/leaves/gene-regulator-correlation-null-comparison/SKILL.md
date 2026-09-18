---
name: gene-regulator-correlation-null-comparison
description: 'Use when validating a multi-layer GRN inference method against simulated
  ground truth: MOSim supplies both the count matrices (RNA-seq, miRNA-seq, TF) and
  the active regulatory relationships needed as reference edges and as a correlation
  null.'
when_to_use_negative:
- The input is real (non-simulated) data with no known regulatory ground truth — the
  simulated significant regulations and the no-effect null pairs only exist because
  MOSim generated them
- You need single-cell GRN benchmarking with miRNA or methylation layers — sc_mosim
  simulation is restricted to scRNA-seq and scATAC-seq regulators only
- You need read-level (FASTQ) benchmarking rather than count-matrix-level analysis
  — MOSim operates at the count matrix level rather than simulating read data
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: MOSim
  role: Simulates bulk RNA-seq, miRNA-seq and TF count matrices and outputs the active
    regulatory relationships used as benchmark ground truth
  repo: github.com/ConesaLab/MOSim
- name: NOISeq
  role: Filters the simulated counts prior to TMM normalization
- name: limma
  role: voom transformation and differential expression analysis on the TMM-normalized
    counts
- name: MORE
  role: Infers the multi-layer GRN with the PLS1 option, whose edges are benchmarked
    against the simulated regulations
- name: R
  role: Implementation environment for the whole simulation-to-benchmark workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/421834
    title: 'MOSim: bulk and single-cell multi-layer regulatory network simulator'
  source_content_hash: sha256:78cc85632190586eab26dc5249d39ab77dfb8e12ae49f9e59f792e57ba591cfd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ConesaLab/MOSim
  repo_url: https://github.com/ConesaLab/MOSim
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
yes_rate: 0.8333333333333334
n_claims: 6
---

# Gene-regulator correlation null comparison for GRN benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks a multi-layer GRN inference tool (e.g. MORE with PLS1) by comparing inferred gene-regulator correlations against MOSim-simulated significant regulations, using ~20,000 random feature pairs with no simulated regulatory effects as a null distribution to compute sensitivity and F1-score.

## When to use

Use when validating a multi-layer GRN inference method against simulated ground truth: MOSim supplies both the count matrices (RNA-seq, miRNA-seq, TF) and the active regulatory relationships needed as reference edges and as a correlation null.

## When NOT to use

- The input is real (non-simulated) data with no known regulatory ground truth — the simulated significant regulations and the no-effect null pairs only exist because MOSim generated them
- You need single-cell GRN benchmarking with miRNA or methylation layers — sc_mosim simulation is restricted to scRNA-seq and scATAC-seq regulators only
- You need read-level (FASTQ) benchmarking rather than count-matrix-level analysis — MOSim operates at the count matrix level rather than simulating read data

## Inputs

- STATegra seed dataset (mouse multi-omics counts) for MOSim simulation
- Regulator-gene association matrix linking regulators with transcripts
- Simulated bulk count matrices for RNA-seq, miRNA-seq and TF layers
- Simulated ground truth: significant regulatory interactions output by MOSim
- Inferred GRN edges (e.g. from MORE PLS1)

## Outputs

- Sensitivity of inferred GRN edges against simulated significant regulations (e.g. 85.5%)
- F1-score of inferred GRN edges against simulated significant regulations (e.g. 62.9%)
- Null distribution of gene-regulator correlations from ~20,000 random feature pairs with no simulated regulatory effects

## How to apply

First run the MOSim `mosim` function on the STATegra seed dataset to simulate bulk RNA-seq, miRNA-seq and TF data (e.g. 30M reads, 2 experimental groups, 20 time points, 1 replicate, 50% DEGs, 60% significant regulations), retaining the simulated significant regulatory interactions as ground truth. Pre-process the simulated counts with NOISeq filtering followed by TMM normalization, then apply voom transformation and run limma differential expression. Infer the multi-layer GRN with MORE using the PLS1 option on the processed RNA-seq, miRNA-seq and TF matrices. For benchmarking, compute gene-regulator correlations for the inferred edges, benchmark them against approximately 20,000 random feature pairs with no simulated regulatory effects (the null), and score the inferred edges against the simulated significant regulations to obtain sensitivity and F1-score. MOSim is chosen because, unlike InterSIM, OmicsSIMLA, sismonr, scDesign3, scMultiSim and GRouNdGAN, it reports active regulatory relationships across omic features, which is what makes a correlation-null benchmark possible.

## Related tools

- **MOSim** (Simulates bulk RNA-seq, miRNA-seq and TF count matrices and outputs the active regulatory relationships used as benchmark ground truth) — github.com/ConesaLab/MOSim
- **NOISeq** (Filters the simulated counts prior to TMM normalization)
- **limma** (voom transformation and differential expression analysis on the TMM-normalized counts)
- **MORE** (Infers the multi-layer GRN with the PLS1 option, whose edges are benchmarked against the simulated regulations)
- **R** (Implementation environment for the whole simulation-to-benchmark workflow)

## Examples

```
install.packages("BiocManager"); BiocManager::install("MOSim")
```

## Evaluation signals

- Sensitivity of inferred edges against MOSim-simulated significant regulations reaches approximately 85.5%
- F1-score of inferred edges against simulated ground truth reaches approximately 62.9%
- The null distribution is built from ~20,000 random feature pairs that carry no simulated regulatory effects
- Simulated design parameters match the intended settings (e.g. 50% DEGs, 60% significant regulations, 2 experimental groups, 20 time points)
- K-means clustering of simulated bulk feature profiles reproduces the simulated temporal expression patterns

## Limitations

- MOSim operates at the count matrix level rather than simulating read data, so benchmarking cannot test read-processing steps
- Single-cell simulation is restricted to scRNA-seq and scATAC-seq; proteomics and metabolomics regulatory layers are not supported
- MOSim is not prepared to simulate GRNs with spatial resolution, limiting benchmarking of spatial multi-omics GRN methods

## Evidence
