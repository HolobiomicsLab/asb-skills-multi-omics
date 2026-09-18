---
name: label-transfer-between-datasets
description: Use when an unannotated cohort (e.g. Mayo-PDX GBM bulk RNA-Seq, n=66)
  must receive subtype or metadata labels from an annotated reference cohort (e.g.
  CPTAC) after mosaicMPI program-network integration, without shared sample IDs or
  full feature overlap.
when_to_use_negative:
- Datasets cannot be decomposed into comparable molecular programs — the transfer
  relies on shared program communities, not direct gene or sample matching
- The reference cohort lacks a reliable annotation layer to propagate; mosaicMPI transfers
  community-specific metadata, so a source label must exist
- Features are expected to be subset to a shared intersection — mosaicMPI's mosaic
  integration explicitly does not require subsetting features/genes, so forcing a
  shared-gene workflow discards its advantage
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: mosaicMPI
  role: Provides the integration network (mosaicmpi integrate) and the transfer-labels
    subcommand that statistically propagates source-cohort annotations to the destination
    cohort via program communities
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: cNMF
  role: Consensus non-negative matrix factorization used to discover the low- to high-resolution
    programs whose communities anchor the label transfer
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: AnnData
  role: Container format (h5ad) for per-cohort expression matrices and metadata layers
    used as integration and label-transfer inputs
- name: pandas
  role: Source data matrices from which the AnnData h5ad objects can be built
- name: NumPy
  role: Consumes program-level correlation outputs (pearson.df.npz) from the integrated
    network for downstream checks
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:bb25f0bccaa53f3f9fe966c78f7527ec71e815791b56152c15d94c845a2eab29
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# label-transfer-between-datasets

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Community-based label transfer in mosaicMPI: after integrating datasets into a cross-cohort program network, statistical propagation of annotated metadata (e.g. transcriptional subtypes) moves labels from a source cohort to a destination cohort through shared program communities.

## When to use

Use when an unannotated cohort (e.g. Mayo-PDX GBM bulk RNA-Seq, n=66) must receive subtype or metadata labels from an annotated reference cohort (e.g. CPTAC) after mosaicMPI program-network integration, without shared sample IDs or full feature overlap.

## When NOT to use

- Datasets cannot be decomposed into comparable molecular programs — the transfer relies on shared program communities, not direct gene or sample matching
- The reference cohort lacks a reliable annotation layer to propagate; mosaicMPI transfers community-specific metadata, so a source label must exist
- Features are expected to be subset to a shared intersection — mosaicMPI's mosaic integration explicitly does not require subsetting features/genes, so forcing a shared-gene workflow discards its advantage

## Inputs

- Bulk RNA-Seq expression matrix and metadata (e.g. Mayo-PDX GBM cohort, n=66) as text files for txt-to-h5ad import
- Annotated reference cohort with subtype/metadata layer (e.g. CPTAC GBM) as .h5ad
- mosaicMPI TOML configuration file naming source and destination datasets
- Integrated network object network_integration.pkl.gz with program communities

## Outputs

- Transferred subtype labels per destination-cohort sample (e.g. CPTAC-derived transcriptional subtypes for Mayo-PDX samples)
- Concordance metric between transferred labels and independently derived subtypes (reported ~72%)

## How to apply

First import each cohort (expression matrix plus metadata) into AnnData h5ad files with 'mosaicmpi txt-to-h5ad', select highly-variable features ('mosaicmpi select-hvf default' models mean-variance dependence on normalized data with F-distribution p-values, --alpha 0.05), and run cNMF factorization over a rank range (e.g. --k_range 2 60 1) followed by 'mosaicmpi factorize' and 'mosaicmpi postprocess'. Build a TOML config listing both datasets ('mosaicmpi create-config') and run 'mosaicmpi integrate' to construct the cross-cohort program network with communities (max_k_filter based on max_k_median_corr) and a network_integration.pkl.gz object. Then run 'mosaicmpi transfer-labels -n network_integration.pkl.gz' with the annotated cohort (e.g. CPTAC) as source and the target cohort (e.g. Mayo-PDX) as destination on the subtype annotation layer; mosaicMPI propagates community-specific metadata from every input modality, so no shared sample IDs are needed. Validate by computing overall label-transfer concordance against an independent subtype assignment (Mayo-based transcriptional subtypes); in the source task this was ~72% (Figure 6G).

## Related tools

- **mosaicMPI** (Provides the integration network (mosaicmpi integrate) and the transfer-labels subcommand that statistically propagates source-cohort annotations to the destination cohort via program communities) — https://github.com/MorrissyLab/mosaicMPI
- **cNMF** (Consensus non-negative matrix factorization used to discover the low- to high-resolution programs whose communities anchor the label transfer) — https://github.com/MorrissyLab/mosaicMPI
- **AnnData** (Container format (h5ad) for per-cohort expression matrices and metadata layers used as integration and label-transfer inputs)
- **pandas** (Source data matrices from which the AnnData h5ad objects can be built)
- **NumPy** (Consumes program-level correlation outputs (pearson.df.npz) from the integrated network for downstream checks)

## Examples

```
mosaicmpi transfer-labels -n network_integration.pkl.gz -c config.toml -s CPTAC -d Mayo -l subtype -o output_directory
```

## Evaluation signals

- Overall concordance between CPTAC-transferred and Mayo-based transcriptional subtype labels should reach approximately the reported 72% (Figure 6G)
- Integration step must complete and produce network_integration.pkl.gz containing program communities that passed the max_k_filter (max_k_median_corr)
- Each destination sample should receive the most abundant CPTAC label among its matched program communities; missing/unassigned labels indicate the network lacked shared communities for that sample
- Sanity-check that programs were refit across all features: cNMF factorizes on the over-dispersed (HVF) subset then NNLS-refits programs across all features, so transferred programs should not be limited to the HVF intersection
- Verify source/destination assignment: 'mosaicmpi transfer-labels -n network_integration.pkl.gz' must specify the annotated cohort (CPTAC) as source and Mayo as destination on the subtype layer

## Limitations

- Potential loss of relevant information between modalities in mosaic integration because not all features are shared between cohorts
- Label transfer depends on program-community alignment across cohorts with heterogeneous statistical properties; datasets have incomplete feature overlap and no shared sample IDs, so communities with no cross-cohort programs cannot transfer labels
- No software version information is stated for the tools/methods in the article, complicating exact reproduction

## Evidence
