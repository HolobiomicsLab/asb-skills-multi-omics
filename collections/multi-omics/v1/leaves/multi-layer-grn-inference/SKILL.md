---
name: multi-layer-grn-inference
description: Use when you have matched bulk RNA-seq counts plus at least one regulatory
  omics layer (miRNA-seq, TF, ATAC/ChIP-seq) and need to infer cross-omics regulator-gene
  interactions, ideally with a simulated ground truth for validation.
when_to_use_negative:
- Single-cell datasets outside scRNA-seq/scATAC-seq — MOSim single-cell simulation
  is currently restricted to those two modalities
- Studies requiring proteomics, metabolomics, or spatially resolved GRN simulation
  — MOSim modules do not support these layers
- Input is already read-level sequencing data — MOSim operates at the count matrix
  level rather than simulating reads
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: MOSim
  role: Simulates the bulk multi-omics dataset (RNA-seq, miRNA-seq, TF) and emits
    the ground-truth active regulatory relationships used to benchmark the inferred
    GRN
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Bioconductor environment hosting MOSim and the preprocessing/benchmarking
    pipeline (NOISeq, limma, voom)
- name: NOISeq
  role: Count filtering step applied to simulated counts before normalization
- name: limma
  role: voom transformation and differential expression analysis on TMM-normalized
    counts prior to GRN inference
- name: MORE
  role: Multi-Omics Regulation GRN inference tool, run with the PLS1 option on the
    processed RNA-seq, miRNA-seq and TF matrices
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/421834
    title: 'MOSim: bulk and single-cell multi-layer regulatory network simulator'
  source_content_hash: sha256:a8150d0bc21c6d0b6d07b8ae8a17194a4d8a51848c1806c5129525fb6fd561fc
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# multi-layer-grn-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infers multi-layer gene regulatory networks by integrating regulatory omics layers (miRNA-seq, TF activity, ATAC/ChIP-seq) with gene expression, and benchmarks the inferred regulatory edges against simulator-generated ground truth (e.g. MOSim's simulated significant regulations).

## When to use

Use when you have matched bulk RNA-seq counts plus at least one regulatory omics layer (miRNA-seq, TF, ATAC/ChIP-seq) and need to infer cross-omics regulator-gene interactions, ideally with a simulated ground truth for validation.

## When NOT to use

- Single-cell datasets outside scRNA-seq/scATAC-seq — MOSim single-cell simulation is currently restricted to those two modalities
- Studies requiring proteomics, metabolomics, or spatially resolved GRN simulation — MOSim modules do not support these layers
- Input is already read-level sequencing data — MOSim operates at the count matrix level rather than simulating reads

## Inputs

- Simulated or real bulk RNA-seq count matrix (e.g. from MOSim mosim() with STATegra seed dataset)
- Matched regulatory omics count matrices (miRNA-seq, TF activity, and optionally ATAC-seq/ChIP-seq/Methyl-seq)
- MOSim-simulated significant regulatory interactions (ground truth edge list)
- Set of random feature pairs with no simulated regulatory effects (negative benchmark set)

## Outputs

- Inferred multi-layer GRN edge list (regulator-gene interactions across omics layers)
- Benchmark metrics: sensitivity and F1-score of inferred edges vs. simulated ground truth (e.g. 85.5% sensitivity, 62.9% F1)
- Normalized/voom-transformed expression matrices and limma differential expression results

## How to apply

Simulate a labeled multi-omics dataset with MOSim: load the STATegra seed dataset and run the mosim R function for bulk RNA-seq, miRNA-seq and TF simulation (30M reads, 2 experimental groups, 20 time points, 1 replicate, 50% DEGs, 60% significant regulations), retaining the simulated significant regulatory interactions as ground truth. Pre-process the simulated counts with NOISeq filtering followed by TMM normalization, then apply the voom transformation and run limma differential expression on the normalized counts. Run the GRN inference tool (e.g. MORE with the PLS1 option) on the processed RNA-seq, miRNA-seq and TF matrices to infer the multi-layer regulatory network. Benchmark inferred edges against the MOSim ground truth by comparing with 20,000 random feature pairs that carry no simulated regulatory effects, and compute sensitivity and F1-score (reference result for MORE/PLS1: sensitivity 85.5%, F1 62.9%).

## Related tools

- **MOSim** (Simulates the bulk multi-omics dataset (RNA-seq, miRNA-seq, TF) and emits the ground-truth active regulatory relationships used to benchmark the inferred GRN) — github.com/ConesaLab/MOSim
- **R** (Bioconductor environment hosting MOSim and the preprocessing/benchmarking pipeline (NOISeq, limma, voom))
- **NOISeq** (Count filtering step applied to simulated counts before normalization)
- **limma** (voom transformation and differential expression analysis on TMM-normalized counts prior to GRN inference)
- **MORE** (Multi-Omics Regulation GRN inference tool, run with the PLS1 option on the processed RNA-seq, miRNA-seq and TF matrices)

## Examples

```
BiocManager::install("MOSim")
```

## Evaluation signals

- Sensitivity of inferred edges against MOSim significant regulations reaches ~85.5% (MORE/PLS1 reference)
- F1-score against MOSim simulated ground truth reaches ~62.9% (MORE/PLS1 reference)
- Negative set of 20,000 random feature pairs with no simulated regulatory effects is used for comparison, so specificity/F1 are computable
- Simulated count matrices reproduce the requested design: 2 experimental groups, 20 time points, 1 replicate, 50% DEGs, 60% significant regulations
- Preprocessing chain applied in order: NOISeq filtering → TMM normalization → voom transformation → limma DE

## Limitations

- Single-cell simulation is restricted to scRNA-seq and scATAC-seq only; other single-cell omics are future work
- Proteomics and metabolomics layers are not supported; gene-proteome-metabolome interaction simulation is deferred to future work
- MOSim does not simulate GRNs with spatial resolution, and operates at the count matrix level rather than simulating read-level data
- Benchmark performance depends on the MOSim simulation parameters (e.g. 60% significant regulations) and the seed dataset chosen (STATegra)

## Evidence

- [abstract] MOSim enables users to generate quantification matrices for each simulated omics data type ... and elucidates the active regulatory relationships between regulatory omics and gene expression data at: [evidence span withheld — anchor status: not_found]
