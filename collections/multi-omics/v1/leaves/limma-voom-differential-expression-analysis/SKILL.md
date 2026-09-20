---
name: limma-voom-differential-expression-analysis
description: Use when normalized RNA-seq count matrices (e.g. TMM-normalized after
  NOISeq filtering, from bulk RNA-seq simulation or real data) need differential expression
  calls to feed downstream GRN inference or benchmarking.
when_to_use_negative:
- Input is raw unnormalized counts that have not undergone NOISeq filtering and TMM
  normalization — voom expects a filtered, normalized count matrix
- Single-cell data — MOSim's single-cell module (sc_mosim/SPARSim) targets scRNA-seq
  and scATAC-seq, for which limma-voom bulk DE is not the appropriate model
- Non-transcriptomic layers such as methylation or ChIP-seq signal, for which a voom
  count-based transformation does not apply
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: limma
  role: Linear-model differential expression analysis applied after voom transformation
    on TMM-normalized counts in the MORE GRN benchmark workflow
- name: NOISeq
  role: Count filtering preceding voom/limma DE in the pre-processing chain
- name: MOSim
  role: Generates the simulated bulk RNA-seq (negative-binomial) count matrices with
    defined DEG proportions that serve as DE-analysis input
  repo: github.com/ConesaLab/MOSim
- name: MORE
  role: Consumes the voom/limma-processed RNA-seq, miRNA-seq and TF matrices (PLS1
    option) to infer the multi-layer GRN
- name: R
  role: Implementation environment for the MOSim → NOISeq → TMM → voom → limma pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/421834
    title: 'MOSim: bulk and single-cell multi-layer regulatory network simulator'
  source_content_hash: sha256:cfcdac5bff848c968992af8c797bfd2880cb38adace4341f2ac2b21d5fd2e949
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# limma-voom differential expression analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies voom transformation to normalized RNA-seq counts followed by limma linear-model differential expression testing, as used in the MORE multi-omics GRN benchmark pipeline built on MOSim-simulated data to prepare differential expression inputs for multi-layer regulatory network inference.

## When to use

Use when normalized RNA-seq count matrices (e.g. TMM-normalized after NOISeq filtering, from bulk RNA-seq simulation or real data) need differential expression calls to feed downstream GRN inference or benchmarking.

## When NOT to use

- Input is raw unnormalized counts that have not undergone NOISeq filtering and TMM normalization — voom expects a filtered, normalized count matrix
- Single-cell data — MOSim's single-cell module (sc_mosim/SPARSim) targets scRNA-seq and scATAC-seq, for which limma-voom bulk DE is not the appropriate model
- Non-transcriptomic layers such as methylation or ChIP-seq signal, for which a voom count-based transformation does not apply

## Inputs

- Normalized bulk RNA-seq count matrix (NOISeq-filtered, TMM-normalized), from MOSim simulation or real data
- Design matrix encoding experimental groups, time points, and replicates

## Outputs

- Voom-transformed logCPM expression matrix with precision weights
- Differential expression results table (logFC, statistics) per feature
- DE-filtered expression matrices used as MORE (PLS1) input for multi-layer GRN inference

## How to apply

After MOSim simulation (via the mosim function, e.g. with 30M reads, 2 experimental groups, 20 time points, 50% DEGs) and NOISeq filtering plus TMM normalization of the count matrices, apply the voom transformation to convert counts to precision-weighted logCPM with associated weights. Fit limma linear models on the voom-transformed data using an experimental design reflecting the simulated groups and time points, then extract differential expression results. In the MORE GRN benchmark workflow, these DE results accompany the RNA-seq, miRNA-seq and TF matrices passed to MORE (PLS1 option) for multi-layer regulatory network inference, which is then scored against the MOSim-simulated significant regulations (sensitivity 85.5%, F1-score 62.9%). Correct application means the design matrix matches the simulated/real experimental structure and the transformation preserves count-based mean-variance weighting.

## Related tools

- **limma** (Linear-model differential expression analysis applied after voom transformation on TMM-normalized counts in the MORE GRN benchmark workflow)
- **NOISeq** (Count filtering preceding voom/limma DE in the pre-processing chain)
- **MOSim** (Generates the simulated bulk RNA-seq (negative-binomial) count matrices with defined DEG proportions that serve as DE-analysis input) — github.com/ConesaLab/MOSim
- **MORE** (Consumes the voom/limma-processed RNA-seq, miRNA-seq and TF matrices (PLS1 option) to infer the multi-layer GRN)
- **R** (Implementation environment for the MOSim → NOISeq → TMM → voom → limma pipeline)

## Examples

```
v <- voom(tmm_norm_counts, design); fit <- eBayes(lmFit(v, design)); de <- topTable(fit, number=Inf)
```

## Evaluation signals

- Design matrix dimensions match the MOSim simulation design (e.g. 2 experimental groups, 20 time points, 1 replicate in the benchmark workflow)
- Voom output is a logCPM matrix with per-gene precision weights, not raw counts
- DE call rates are consistent with the simulated 50% DEG proportion
- Downstream MORE GRN benchmark reproduces sensitivity of 85.5% and F1-score of 62.9% against MOSim-simulated significant regulations
- Edge scoring is benchmarked against 20,000 random feature pairs with no simulated regulatory effects

## Limitations

- MOSim operates at the count matrix level rather than simulating read data, so DE analysis is performed on simulated counts without read-level artifacts
- The benchmark workflow uses 1 replicate per condition (20 time points), a design that limits variance estimation in limma models
- Benchmark performance metrics depend on the simulated ground truth (60% significant regulations) rather than experimentally validated interactions

## Evidence
