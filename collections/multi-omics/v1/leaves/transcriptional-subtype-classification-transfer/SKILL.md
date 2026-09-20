---
name: transcriptional-subtype-classification-transfer
description: Use when a new bulk RNA-Seq cohort needs subtype labels propagated from
  an already-annotated reference cohort, especially when cohorts share no samples
  and only partially overlapping genes.
when_to_use_negative:
- The destination cohort already has trusted subtype annotations from its own analysis
  — compare instead of transferring.
- Datasets share all samples and features, so direct joint analysis or re-analysis
  of both cohorts is feasible rather than mosaic integration.
- No annotated reference cohort with subtype labels exists to serve as the transfer
  source.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: mosaicMPI
  role: Provides the CLI (txt-to-h5ad, select-hvf, integrate, transfer-labels) that
    builds the cross-cohort program network and performs community-based subtype label
    transfer
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: cNMF
  role: Consensus non-negative matrix factorization over a rank range (e.g. k 2–60)
    that discovers the low to high-resolution programs used as anchoring features
    for integration and label transfer
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: AnnData
  role: h5ad object format holding the Mayo expression matrix, metadata, HVF selections,
    and cNMF usages
- name: pandas
  role: Data matrices that can be imported into AnnData files/objects for the workflow
- name: NumPy
  role: Consumes program correlation matrices (pearson.df.npz) produced during integration
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:77b591b8ed18dd2745705d443f91c06dc0ba59a797c4a569862791bae84e192d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/MorrissyLab/mosaicMPI
  repo_url: https://github.com/MorrissyLab/mosaicMPI
  tools:
  - AnnData
  - NumPy
  - cNMF
  - mosaicMPI
  - pandas
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# transcriptional-subtype-classification-transfer

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Transfer transcriptional subtype annotations from an annotated reference cohort (CPTAC) to a new bulk RNA-Seq cohort (Mayo-PDX, n=66) by integrating both cohorts into a mosaicMPI program network via cNMF-derived anchoring programs and running community-based label transfer, achieving ~72% concordance with Mayo-based subtypes.

## When to use

Use when a new bulk RNA-Seq cohort needs subtype labels propagated from an already-annotated reference cohort, especially when cohorts share no samples and only partially overlapping genes.

## When NOT to use

- The destination cohort already has trusted subtype annotations from its own analysis — compare instead of transferring.
- Datasets share all samples and features, so direct joint analysis or re-analysis of both cohorts is feasible rather than mosaic integration.
- No annotated reference cohort with subtype labels exists to serve as the transfer source.

## Inputs

- Mayo-PDX bulk RNA-Seq expression matrix (.txt) with sample metadata (.txt)
- Annotated CPTAC reference cohort h5ad with transcriptional subtype labels
- TOML configuration file listing both datasets (via mosaicmpi create-config)
- network_integration.pkl.gz from mosaicmpi integrate

## Outputs

- CPTAC-derived transcriptional subtype labels per Mayo sample (label transfer results)
- Overall label-transfer concordance metric (expected ~72%, Figure 6G)
- Integrated program network with communities (network_integration.pkl.gz)

## How to apply

First build an h5ad for the destination cohort (e.g. Mayo-PDX bulk RNA-Seq expression matrix plus metadata via 'mosaicmpi txt-to-h5ad'), select highly-variable features with 'select-hvf default' (modeling mean-variance dependence on normalized data, F-distribution p-values, --alpha 0.05), and run cNMF factorization across a rank range (e.g. k 2–60) with factorize and postprocess to obtain consensus programs and usages. Then create a TOML config listing both the Mayo and CPTAC datasets and run 'mosaicmpi integrate' to build the cross-cohort program network, where communities are defined with max_k_filter based on max_k_median_corr, producing network_integration.pkl.gz. Finally run 'mosaicmpi transfer-labels' with the annotated CPTAC dataset as source and Mayo as destination on the subtype layer: statistically propagated community metadata assigns each Mayo sample the most abundant CPTAC label among its anchored communities. Rationale: program-level integration avoids subsetting genes to an intersection and permits mosaic integration across cohorts without shared sample IDs.

## Related tools

- **mosaicMPI** (Provides the CLI (txt-to-h5ad, select-hvf, integrate, transfer-labels) that builds the cross-cohort program network and performs community-based subtype label transfer) — https://github.com/MorrissyLab/mosaicMPI
- **cNMF** (Consensus non-negative matrix factorization over a rank range (e.g. k 2–60) that discovers the low to high-resolution programs used as anchoring features for integration and label transfer) — https://github.com/MorrissyLab/mosaicMPI
- **AnnData** (h5ad object format holding the Mayo expression matrix, metadata, HVF selections, and cNMF usages)
- **pandas** (Data matrices that can be imported into AnnData files/objects for the workflow)
- **NumPy** (Consumes program correlation matrices (pearson.df.npz) produced during integration)

## Examples

```
mosaicmpi transfer-labels -n network_integration.pkl.gz
```

## Evaluation signals

- Overall concordance between CPTAC-transferred and Mayo-based transcriptional subtype labels is approximately 72% across samples (Figure 6G)
- transfer-labels runs successfully on network_integration.pkl.gz with CPTAC as source and Mayo as destination on the subtype layer
- Communities in the integrated network pass the max_k_filter based on max_k_median_corr
- cNMF postprocess completes and yields consensus programs and usages in the Mayo h5ad for the requested k range (2–60)
- HVF selection stratified by patient where appropriate returns features at --alpha 0.05 from the F-distribution mean-variance model

## Limitations

- Mosaic integration may lose relevant information between modalities because not all features are shared
- Label transfer concordance is partial (~72%), so transferred labels are approximate rather than definitive assignments
- cNMF supports sparse (zero-containing) input data but not missing values
- Large cNMF factorization runs may need HPC job submission (sample slurm scripts are provided for factorize and postprocess)

## Evidence

- [other] Label transfer from CPTAC to Mayo-PDX showed good agreement, with 72% overall concordance across samples of the most abundant CPTAC-label with the Mayo-based transcriptional subtypes.: [evidence span withheld — anchor status: not_found]
- [methods] mosaicmpi transfer-labels command from workflow: [evidence span withheld — anchor status: not_found]
- [readme] Programs are first discovered using unsupervised deconvolution (consensus non-negative matrix factorization, cNMF) across multiple ranks separately for each dataset.: [evidence span withheld — anchor status: external_doc]
