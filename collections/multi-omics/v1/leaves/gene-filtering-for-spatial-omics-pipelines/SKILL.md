---
name: gene-filtering-for-spatial-omics-pipelines
description: Use when preprocessing spatial multi-omics data (e.g. spatial-CITE-seq
  with paired mRNA and protein modalities) for SMOPCA, and you must select spatially
  variable genes via SPARK before dimension reduction.
when_to_use_negative:
- Input is single-cell (non-spatial) multi-omics data — SPARK tests genes for spatial
  variability and requires spot-level spatial coordinates
- The goal is basic QC rather than SVG selection — the article's zero-count filtering
  step ('Initially, genes and proteins with zero counts were filtered out.') applies
  to both genes and proteins and precedes SPARK
- You need the protein modality filtered by spatial variability — the SPARK FDR filter
  in this pipeline is applied only to genes (mRNA); all proteins are included
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0121
tools:
- name: SPARK
  role: Tests each zero-count-filtered gene for spatial variability across tissue
    spots; its p-values are adjusted with the Benjamini-Yekutieli procedure and genes
    with FDR < 0.05 become the SVG set for SMOPCA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:06dcf4c92ad873b1bcce467346f1cbb982eba3d39d7295e64c483a7c945c5dc0
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# gene-filtering-for-spatial-omics-pipelines

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reduces the ~28,417-gene mRNA matrix of a spatial multi-omics sample (e.g. the spatial-CITE-seq human tonsil, GSE213264) to a curated set of spatially variable genes by testing each filtered gene with SPARK and retaining up to 3000 genes at FDR < 0.05, while leaving all proteins unfiltered for downstream SMOPCA input.

## When to use

Use when preprocessing spatial multi-omics data (e.g. spatial-CITE-seq with paired mRNA and protein modalities) for SMOPCA, and you must select spatially variable genes via SPARK before dimension reduction.

## When NOT to use

- Input is single-cell (non-spatial) multi-omics data — SPARK tests genes for spatial variability and requires spot-level spatial coordinates
- The goal is basic QC rather than SVG selection — the article's zero-count filtering step ('Initially, genes and proteins with zero counts were filtered out.') applies to both genes and proteins and precedes SPARK
- You need the protein modality filtered by spatial variability — the SPARK FDR filter in this pipeline is applied only to genes (mRNA); all proteins are included

## Inputs

- Raw mRNA count matrix from a spatial multi-omics assay (e.g. spatial-CITE-seq GSE213264: 2491 spots x 28,417 genes)
- Paired protein/ADT count matrix (283 proteins for the tonsil sample)
- Spatial coordinates of spots (required by SPARK for spatial variability testing)

## Outputs

- svg_list.csv containing gene identifiers of up to 3000 SPARK-significant spatially variable genes (FDR <= 0.05)
- Filtered gene count matrix restricted to SVGs, with zero-count genes removed
- Unfiltered protein matrix with all 283 proteins retained

## How to apply

First load the raw mRNA count matrix (for the human tonsil spatial-CITE-seq sample: 2491 spots, 28,417 genes, 283 proteins) and drop genes with zero total counts across spots, per the paper's preprocessing protocol. Run SPARK on the filtered count matrix to test each gene for spatial variability across tissue spots. Adjust SPARK p-values with the Benjamini-Yekutieli procedure and retain genes whose FDR falls below 0.05, capping the selection at 3000 spatially variable genes (SVGs). This SVG filtering applies only to the mRNA modality — all 283 proteins are kept unfiltered so both modalities can be jointly modeled. Write the retained gene identifiers to svg_list.csv for the subsequent SMOPCA preprocessing, which normalizes by library size, log-transforms, and scales to zero mean and unit variance.

## Related tools

- **SPARK** (Tests each zero-count-filtered gene for spatial variability across tissue spots; its p-values are adjusted with the Benjamini-Yekutieli procedure and genes with FDR < 0.05 become the SVG set for SMOPCA)

## Examples

```
svg_list <- gene_ids[which(fdr_by < 0.05)][1:min(3000, sum(fdr_by < 0.05))]; write.csv(svg_list, "svg_list.csv", row.names = FALSE)
```

## Evaluation signals

- svg_list.csv contains only gene identifiers — no protein features should appear, since filtering was limited to mRNA data
- Number of retained genes is at most 3000 and every retained gene has Benjamini-Yekutieli-adjusted FDR < 0.05
- No gene or protein with zero total counts across the 2491 spots survives the initial filtering step
- The protein dimension remains unchanged (all 283 proteins of the tonsil sample present) after gene filtering
- Adjusted p-values were computed with the BY procedure, not Benjamini-Hochberg, per the article's stated FDR control

## Limitations

- Spatial variability testing is applied only to genes; proteins are never assessed for spatial variability and are carried through unfiltered
- Selection is capped at 3000 SVGs, so weakly significant genes beyond the cap are excluded even at FDR < 0.05
- Zero-count genes and proteins are permanently removed, which may drop biologically relevant but sparsely detected features before testing

## Evidence

- [full_text] source_card: [evidence span withheld — anchor status: not_found]
