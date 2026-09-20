---
name: fdr-multiple-testing-correction
description: Use when testing thousands of features (e.g., 28,417 genes across 2491
  tissue spots) for spatial variability or differential signal and multiplicity-controlled
  significance calls (BY-adjusted FDR ≤ 0.05) are needed for feature selection.
when_to_use_negative:
- 'The modality is protein/ADT rather than mRNA: the article''s protocol keeps all
  283 proteins unfiltered, so applying the SVG FDR filter to proteins would deviate
  from the method'
- The dataset has no spatial coordinates for spots (e.g., dissociated single-cell
  multi-omics without pseudo-spatial locations), since SPARK's spatial variability
  test is undefined
- The input is already a normalized/feature-selected expression matrix rather than
  a raw count matrix from which zero-count genes can be removed
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: SPARK
  role: Tests each gene in the filtered mRNA count matrix for spatial variability
    across the 2491 tissue spots; its p-values are then adjusted by the Benjamini-Yekutieli
    procedure at FDR < 0.05 to select SVGs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:823b75440f8580396ae8f61bb4f25858a2bb634c8c3acdae7d461c89e034576a
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# fdr-multiple-testing-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies statistically significant spatially variable genes (SVGs) by running SPARK across thousands of gene-level tests on a spatial-CITE-seq count matrix and correcting p-values with the Benjamini-Yekutieli (BY) procedure at an FDR threshold of 0.05, yielding up to 3000 SVGs as downstream SMOPCA input.

## When to use

Use when testing thousands of features (e.g., 28,417 genes across 2491 tissue spots) for spatial variability or differential signal and multiplicity-controlled significance calls (BY-adjusted FDR ≤ 0.05) are needed for feature selection.

## When NOT to use

- The modality is protein/ADT rather than mRNA: the article's protocol keeps all 283 proteins unfiltered, so applying the SVG FDR filter to proteins would deviate from the method
- The dataset has no spatial coordinates for spots (e.g., dissociated single-cell multi-omics without pseudo-spatial locations), since SPARK's spatial variability test is undefined
- The input is already a normalized/feature-selected expression matrix rather than a raw count matrix from which zero-count genes can be removed

## Inputs

- Raw mRNA count matrix from spatial-CITE-seq data (e.g., GSE213264 human tonsil: 2491 spots × 28,417 genes)
- Spatial coordinates/spot locations for the tissue section
- Zero-count gene filter applied prior to testing

## Outputs

- List of up to 3000 significant spatially variable genes with BY-adjusted FDR ≤ 0.05 (e.g., svg_list.csv with gene identifiers)
- Per-gene SPARK p-values with Benjamini-Yekutieli adjusted FDR values

## How to apply

First remove genes with zero total counts across spots from the raw mRNA count matrix (e.g., the GSE213264 human tonsil sample with 2491 spots, 28,417 genes, 283 proteins), since zero-count features cannot yield meaningful test statistics. Run SPARK on the filtered mRNA count matrix to test each gene for spatial variability across spots. Correct the resulting p-values using the Benjamini-Yekutieli procedure and retain genes whose adjusted FDR falls below 0.05, capping the selection at up to 3000 significant spatially variable genes. Critically, this filtering applies only to the mRNA modality: the protein (ADT) panel is left unfiltered, so all proteins are passed downstream. Write the retained gene identifiers to a list file (e.g., svg_list.csv) to feed SMOPCA preprocessing.

## Related tools

- **SPARK** (Tests each gene in the filtered mRNA count matrix for spatial variability across the 2491 tissue spots; its p-values are then adjusted by the Benjamini-Yekutieli procedure at FDR < 0.05 to select SVGs)

## Evaluation signals

- All retained genes have Benjamini-Yekutieli adjusted FDR strictly below 0.05; no gene exceeding that threshold appears in the SVG list
- The SVG list contains at most 3000 genes, respecting the 'up to 3000 SVGs' cap
- Genes with zero total counts across spots were removed before testing, so no zero-count gene appears in the SVG list
- The protein panel is untouched: all 283 proteins remain in the downstream SMOPCA input despite the gene-level filtering
- The output SVG list contains valid gene identifiers matching the original 28,417-gene count matrix (e.g., written to svg_list.csv)

## Limitations

- The filtering was limited to genes (mRNA data) and did not extend to proteins, so multi-omic feature selection is asymmetric by design
- Zero-count genes must be filtered first; testing unfiltered matrices with zero-count features wastes power and can distort the BY correction
- The 'up to 3000' cap means the SVG set may be truncated before all FDR-significant genes are included, depending on the dataset

## Evidence

- [other] SPARK run on the filtered count matrix per source-card workflow: [evidence span withheld — anchor status: not_found]
- [other] Selection cap and mRNA-only scope from source-card finding: [evidence span withheld — anchor status: not_found]
