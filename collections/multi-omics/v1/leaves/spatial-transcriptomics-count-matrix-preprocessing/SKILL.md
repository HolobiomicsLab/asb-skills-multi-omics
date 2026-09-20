---
name: spatial-transcriptomics-count-matrix-preprocessing
description: Use when preparing a raw spatial multi-omics count matrix (mRNA + protein
  modalities, e.g., spatial-CITE-seq or Stereo-CITE-seq) as input for spatial dimension
  reduction such as SMOPCA, and gene-level feature selection is needed.
when_to_use_negative:
- When the input is already a feature-selected or normalized expression matrix — this
  skill expects raw count matrices and its first step is zero-count feature removal
- When the goal is to filter proteins as well as genes — the SMOPCA preprocessing
  protocol restricts SPARK-based filtering to the mRNA modality and keeps all proteins
  unfiltered
- When no spatial coordinates are available for the spots, since SPARK tests spatial
  variability using spot locations
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0121
tools:
- name: SPARK
  role: Identifies spatially variable genes from the filtered mRNA count matrix; p-values
    are adjusted with the Benjamini-Yekutieli procedure at FDR 0.05 to select up to
    3000 SVGs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:585de3aefbb7104fc3d3aa95a21fde6097141cf35458f20727b144213f34dd4c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hemberg-lab/SC3
  tools:
  - SPARK
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# spatial-transcriptomics-count-matrix-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess spatial multi-omics count matrices (e.g., the spatial-CITE-seq human tonsil sample GSE213264 with 2491 spots, 28,417 genes, and 283 proteins) by filtering zero-count features, selecting spatially variable genes with SPARK (FDR ≤ 0.05 via Benjamini-Yekutieli, up to 3000 SVGs), and keeping all proteins unfiltered before SMOPCA input.

## When to use

Use when preparing a raw spatial multi-omics count matrix (mRNA + protein modalities, e.g., spatial-CITE-seq or Stereo-CITE-seq) as input for spatial dimension reduction such as SMOPCA, and gene-level feature selection is needed.

## When NOT to use

- When the input is already a feature-selected or normalized expression matrix — this skill expects raw count matrices and its first step is zero-count feature removal
- When the goal is to filter proteins as well as genes — the SMOPCA preprocessing protocol restricts SPARK-based filtering to the mRNA modality and keeps all proteins unfiltered
- When no spatial coordinates are available for the spots, since SPARK tests spatial variability using spot locations

## Inputs

- Raw mRNA count matrix from a spatial-CITE-seq sample (e.g., GSE213264 human tonsil: 2491 spots x 28,417 genes)
- Protein/ADT abundance matrix from the same sample (e.g., 283 proteins measured on the same 2491 spots)
- Spatial coordinates of spots for SPARK spatial variability testing

## Outputs

- Filtered mRNA count matrix restricted to significant spatially variable genes (up to 3000 SVGs with FDR ≤ 0.05)
- svg_list.csv containing the retained SVG gene identifiers for downstream SMOPCA preprocessing
- Unfiltered protein matrix (all proteins retained, e.g., all 283 proteins)

## How to apply

Load the raw mRNA count matrix and the protein (ADT) matrix from the spatial dataset (e.g., GSE213264 human tonsil: 2491 spots, 28,417 genes, 283 proteins). First filter out features with zero total counts across spots: 'Initially, genes and proteins with zero counts were filtered out.' Then run SPARK on the filtered mRNA matrix to test each gene for spatial variability across spots, adjusting p-values with the Benjamini-Yekutieli procedure and retaining genes with FDR ≤ 0.05; cap the selection at 3000 significant SVGs. Apply the SVG filtering only to the mRNA modality — all 283 proteins are kept unfiltered so both modalities enter downstream preprocessing (library-size normalization, log transformation, scaling to zero mean and unit variance). Write the retained SVG gene identifiers to a file such as svg_list.csv for downstream SMOPCA preprocessing.

## Related tools

- **SPARK** (Identifies spatially variable genes from the filtered mRNA count matrix; p-values are adjusted with the Benjamini-Yekutieli procedure at FDR 0.05 to select up to 3000 SVGs)

## Evaluation signals

- The gene count matrix contains no genes with zero total counts across the 2491 spots after filtering
- The retained SVG set contains at most 3000 genes, each with FDR ≤ 0.05 under the Benjamini-Yekutieli adjustment
- svg_list.csv contains gene identifiers that all appear in the filtered mRNA count matrix
- The protein matrix retains all 283 proteins — no protein-level filtering is applied
- Downstream normalization yields zero mean and unit variance after library-size normalization and log transformation

## Limitations

- The paper reports that SMOPCA uses normalized data rather than raw counts, 'potentially resulting in a loss of inference accuracy and subsequent reduction in analytical power'
- The SMOPCA model 'does not consider the dropout events and the inherent mean–variance relationship presents in raw counts', which may make preprocessing choices suboptimal
- SPARK-based SVG selection caps the gene set (up to 3000 SVGs), which discards non-spatially-variable genes that might still carry modality-integrating signal

## Evidence

- [discussion] here, we used human tonsil sample for the analysis, which contains 2491 spots with 28,417 genes and 283 proteins: [evidence span withheld — anchor status: not_found]
- [other] Up to 3000 significant spatially variable genes (SVGs) with FDR ≤ 0.05 were selected as SMOPCA input; the filtering process was limited to genes (mRNA data), and all proteins were included.: [evidence span withheld — anchor status: not_found]
