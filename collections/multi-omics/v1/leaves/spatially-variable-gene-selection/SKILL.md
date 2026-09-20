---
name: spatially-variable-gene-selection
description: Use when preprocessing spatial transcriptomics or spatial-CITE-seq count
  matrices (e.g., GSE213264 tonsil, MISAR-seq E15.5 mouse brain) before dimension
  reduction, and you need spatially informative gene features rather than generic
  HVGs.
when_to_use_negative:
- The data are not spatially resolved (no spot coordinates available) — SPARK spatial
  variability testing is not applicable, use HVG selection instead
- 'You need modality-wide feature selection for proteins/ADT: the paper explicitly
  keeps all proteins (283) unfiltered, applying SVG filtering only to the mRNA modality'
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SPARK
  role: Tests each gene for spatial variability across tissue spots and yields p-values
    that are FDR-adjusted with the Benjamini-Yekutieli procedure to select SVGs
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_008
  - task_001
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:3ab179a12846f87745dc85357fb21b162de64accbcd00f177026812782d4b01a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hemberg-lab/SC3
  tools:
  - MEFISTO
  - SMOPCA
  - SPARK
  - SRTsim
  - SpaVAE
  - SpatialGlue
  - SpatialPCA
  - TotalVI
  - scMDC
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 18
---

# Spatially variable gene selection with SPARK

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify spatially variable genes (SVGs) from spatial transcriptomics count matrices using SPARK with a Benjamini-Yekutieli FDR threshold of 0.05 (up to 3000 genes), so that only spatially informative features feed downstream spatial domain detection methods such as SMOPCA.

## When to use

Use when preprocessing spatial transcriptomics or spatial-CITE-seq count matrices (e.g., GSE213264 tonsil, MISAR-seq E15.5 mouse brain) before dimension reduction, and you need spatially informative gene features rather than generic HVGs.

## When NOT to use

- The data are not spatially resolved (no spot coordinates available) — SPARK spatial variability testing is not applicable, use HVG selection instead
- You need modality-wide feature selection for proteins/ADT: the paper explicitly keeps all proteins (283) unfiltered, applying SVG filtering only to the mRNA modality

## Inputs

- Raw mRNA count matrix from a spatial transcriptomics assay (e.g., GSE213264 human tonsil spatial-CITE-seq: 2491 spots x 28,417 genes)
- Spatial coordinates of spots for SPARK spatial variability testing
- Optionally a gene activity matrix derived from ATAC peaks (MISAR-seq) for the paired mRNA modality

## Outputs

- List of up to 3000 significant spatially variable genes with FDR <= 0.05 (Benjamini-Yekutieli), e.g., written to svg_list.csv
- Filtered count matrix restricted to SVGs (all protein measurements retained unfiltered for CITE-seq data)

## How to apply

Start from the raw mRNA count matrix (e.g., the human tonsil spatial-CITE-seq sample GSE213264 with 2491 spots, 28,417 genes, and 283 proteins) and first filter out genes and proteins with zero total counts across spots. Run SPARK on the filtered count matrix to test each gene for spatial variability across tissue spots, then adjust p-values with the Benjamini-Yekutieli procedure and retain genes at an FDR threshold of 0.05, keeping up to 3000 SVGs. The filtering is limited to the mRNA modality: all proteins (e.g., the 283 tonsil proteins) are kept unfiltered as SMOPCA input. For MISAR-seq data, apply the same SPARK FDR 0.05 BY selection to the mRNA modality after the ATAC peak matrix has been collapsed to a gene activity matrix. Selected SVGs are then normalized by library size, log-transformed, and scaled to zero mean and unit variance before fitting SMOPCA; the sensitivity analysis shows SVG selection is robust versus HVG alternatives (SCTransform, VST) and Matern kernels (nu = 0.5, 1.5, 2.5).

## Related tools

- **SPARK** (Tests each gene for spatial variability across tissue spots and yields p-values that are FDR-adjusted with the Benjamini-Yekutieli procedure to select SVGs)

## Evaluation signals

- The retained gene list contains at most 3000 genes, all with BY-adjusted FDR < 0.05
- Genes and proteins with zero total counts across spots are absent from the retained matrix before SPARK is run
- For spatial-CITE-seq input, all protein features (e.g., 283 tonsil proteins) remain in the analysis — only the mRNA modality is reduced to SVGs
- Downstream SMOPCA run on these SVGs reproduces expected spatial domain structure (e.g., k=7 tonsil clusters with high Moran's I / low LISI relative to baselines)
- Robustness check: swapping SVG selection for HVGs or changing normalization (LogNormalize, SCTransform, VST) changes clustering metrics (e.g., ARI) negligibly, per the paper's stability result

## Limitations

- Filtering is defined only for the mRNA modality; applying it blindly to protein/ADT matrices would deviate from the paper's preprocessing protocol
- SMOPCA downstream uses normalized data rather than raw counts and does not model dropout or the mean-variance relationship of raw counts, which may reduce analytical power
- SVG selection is tied to the FDR 0.05 BY threshold; in very sparse/noisy data or with complicated spatial structure, downstream performance may require manual tuning of kernel lengthscale parameters

## Evidence

- [other] SVG selection scope (task card): [evidence span withheld — anchor status: not_found]
- [other] MISAR-seq preprocessing scope (task card): [evidence span withheld — anchor status: not_found]
