---
name: grn-edge-recovery-benchmarking
description: Use when benchmarking a multi-omics GRN inference tool (e.g., MORE with
  the PLS1 option) against MOSim-simulated significant regulations, or when you need
  sensitivity/F1 of regulatory-edge recovery against a known simulated ground truth.
when_to_use_negative:
- Real (non-simulated) data with no ground-truth significant regulations — edge-recovery
  sensitivity and F1 cannot be computed
- Benchmarking requires read-level (FASTQ) input or omics layers MOSim cannot simulate,
  e.g. proteomics, metabolomics or spatially-resolved GRNs
- The goal is generating simulated multi-omics data itself (call mosim()/sc_mosim()
  directly) rather than scoring an inference tool's edge recovery
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
tools:
- name: MOSim
  role: Simulates bulk RNA-seq, miRNA-seq and TF count matrices from the STATegra
    seed dataset and outputs the ground-truth significant regulatory interactions
    used for edge-recovery scoring
  repo: github.com/ConesaLab/MOSim
- name: MORE
  role: Multi-omics GRN inference tool run with the PLS1 option on the processed RNA-seq,
    miRNA-seq and TF matrices; its inferred edges are the ones scored for sensitivity
    and F1-score
- name: NOISeq
  role: Filters the simulated counts prior to TMM normalization in the benchmark preprocessing
- name: limma
  role: Differential expression analysis on voom-transformed normalized counts as
    part of the benchmark preprocessing
- name: R
  role: Execution environment; MOSim is a Bioconductor R package installed via BiocManager
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/421834
    title: 'MOSim: bulk and single-cell multi-layer regulatory network simulator'
  source_content_hash: sha256:72d67f690b20bc0d2a6398d1b3a3d6b46fe1068945267cc253627a29fbade557
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
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# grn-edge-recovery-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark multi-layer GRN inference by scoring inferred regulatory edges against MOSim-simulated significant regulations used as ground truth, quantifying sensitivity and F1-score. Reference run: MORE with the PLS1 option achieved 85.5% sensitivity and 62.9% F1-score.

## When to use

Use when benchmarking a multi-omics GRN inference tool (e.g., MORE with the PLS1 option) against MOSim-simulated significant regulations, or when you need sensitivity/F1 of regulatory-edge recovery against a known simulated ground truth.

## When NOT to use

- Real (non-simulated) data with no ground-truth significant regulations — edge-recovery sensitivity and F1 cannot be computed
- Benchmarking requires read-level (FASTQ) input or omics layers MOSim cannot simulate, e.g. proteomics, metabolomics or spatially-resolved GRNs
- The goal is generating simulated multi-omics data itself (call mosim()/sc_mosim() directly) rather than scoring an inference tool's edge recovery

## Inputs

- STATegra seed dataset (mouse multi-omics) used as the MOSim bulk-simulation seed
- Regulator-gene association matrix linking regulators with transcripts
- mosim() output count matrices for bulk RNA-seq, miRNA-seq and the TF regulatory layer
- MOSim-simulated significant regulatory interactions retained as the ground-truth edge list

## Outputs

- Sensitivity and F1-score of inferred regulatory edges versus the simulated ground truth
- Inferred multi-layer gene regulatory network (gene-regulator edge list from MORE PLS1)
- Edge scores benchmarked against 20,000 random feature pairs with no simulated regulatory effects

## How to apply

Simulate ground truth with the mosim R function on the STATegra seed dataset for bulk RNA-seq, miRNA-seq and TF layers (30M reads, 2 experimental groups, 20 time points, 1 replicate, 50% DEGs, 60% significant regulations); since MOSim reports the active regulatory relationships between regulatory omics and gene expression, retain the simulated significant regulatory interactions as the edge gold standard. Pre-process the simulated counts with NOISeq filtering followed by TMM normalization, then apply the voom transformation and run limma differential expression analysis on the normalized counts. Run the GRN inference tool under test — MORE with the PLS1 option — on the processed RNA-seq, miRNA-seq and TF matrices to infer the multi-layer gene regulatory network. Compare inferred regulatory edges against the simulated significant regulations, benchmarking gene-regulator correlations against 20,000 random feature pairs with no simulated regulatory effects, and compute sensitivity and F1-score. A faithful reproduction is validated against the reference metrics of 85.5% sensitivity and 62.9% F1-score for MORE.

## Related tools

- **MOSim** (Simulates bulk RNA-seq, miRNA-seq and TF count matrices from the STATegra seed dataset and outputs the ground-truth significant regulatory interactions used for edge-recovery scoring) — github.com/ConesaLab/MOSim
- **MORE** (Multi-omics GRN inference tool run with the PLS1 option on the processed RNA-seq, miRNA-seq and TF matrices; its inferred edges are the ones scored for sensitivity and F1-score)
- **NOISeq** (Filters the simulated counts prior to TMM normalization in the benchmark preprocessing)
- **limma** (Differential expression analysis on voom-transformed normalized counts as part of the benchmark preprocessing)
- **R** (Execution environment; MOSim is a Bioconductor R package installed via BiocManager)

## Examples

```
BiocManager::install("MOSim")  # then call mosim() for bulk RNA-seq, miRNA-seq and TF simulation from the STATegra seed
```

## Evaluation signals

- Sensitivity of edge recovery versus the simulated significant regulations reproduces the reference value of ~85.5% for MORE (PLS1)
- F1-score reproduces the reference value of ~62.9% for MORE (PLS1)
- Simulation design matches spec: 30M reads, 2 experimental groups, 20 time points, 1 replicate, 50% DEGs and 60% significant regulations
- Ground-truth edge list comes directly from the retained mosim() simulated significant regulations, not from any inferred network
- Random-pair baseline is applied: gene-regulator correlations are compared against 20,000 random feature pairs with no simulated regulatory effects

## Limitations

- Metrics measure agreement with MOSim's generative model (negative binomial bulk simulation), not real biological regulation
- MOSim operates at the count matrix level rather than simulating read data
- Single-cell simulation is restricted to scRNA-seq and scATAC-seq, and proteomics and metabolomics layers are not supported by the simulation modules
- MOSim cannot simulate GRNs with spatial resolution

## Evidence

- [abstract] MOSim provides quantification matrices plus active regulatory relationships usable as ground truth: [evidence span withheld — anchor status: not_found]
- [other] Task workflow: mosim simulation parameters for the MORE benchmark: [evidence span withheld — anchor status: not_found]
