---
name: ring-gene-filtering
description: Use when a raw ST gene count matrix (e.g., H1.tsv from Zenodo 10.5281/zenodo.4751624)
  is about to be loaded into stImage via LoadImageFeature, and ring-pattern artifact
  genes must be removed first.
when_to_use_negative:
- When the input count matrix has already been filtered of ring pattern genes, re-applying
  this step would remove legitimate genes.
- When working with datasets that are not the HER2 breast cancer ST benchmark (e.g.,
  the Adult Mouse Olfactory Bulb Visium data), where the ring-gene list from the original
  paper does not apply.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: stImage
  role: R package whose LoadImageFeature/PreProcessing pipeline consumes the ring-gene-filtered
    count table
  repo: https://github.com/YuWang-VUMC/stImage
- name: tidyverse
  role: R data-manipulation toolkit used in the stImage workflow to subset the gene
    count matrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:840bd766e72c0f405963c4263139f057eba5f68266b9254889214f1fc32af901
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuWang-VUMC/stImage
  tools:
  - BayesSpace
  - R
  - SpatialPCA
  - stImage
  - tidyverse
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# ring-gene-filtering

## Summary

Removes the 21 ring-pattern artifact genes from a Spatial Transcriptomics gene count matrix before building the stImage object, as prescribed in the stImage HER2 breast cancer (patient H1) workflow.

## When to use

Use when a raw ST gene count matrix (e.g., H1.tsv from Zenodo 10.5281/zenodo.4751624) is about to be loaded into stImage via LoadImageFeature, and ring-pattern artifact genes must be removed first.

## When NOT to use

- When the input count matrix has already been filtered of ring pattern genes, re-applying this step would remove legitimate genes.
- When working with datasets that are not the HER2 breast cancer ST benchmark (e.g., the Adult Mouse Olfactory Bulb Visium data), where the ring-gene list from the original paper does not apply.

## Inputs

- Gene count matrix H1.tsv from the HER2-positive breast cancer ST sample (Zenodo 10.5281/zenodo.4751624) with spot names formatted as X<x>x<y>
- List of the 21 ring pattern genes to remove
- Labeled coordinates file H1_labeled_coordinates.tsv (used downstream, must remain row-aligned)

## Outputs

- Filtered gene count matrix with the 21 ring pattern genes removed
- stImage object built via LoadImageFeature(countTable, imageFeatures, positionTable, RGBquantile) ready for PreProcessing with SCT normalization

## How to apply

Before constructing the stImage object, subset the gene-by-spot count matrix to exclude the 21 ring pattern genes mentioned in the original HER2 paper. This is done in R on the H1.tsv count table, keeping spot names formatted as X<x>x<y> so the matrix remains compatible with H1_labeled_coordinates.tsv and the 200-pixel H&E patches from HE_BT24044_D2.jpg. After filtering, pass the cleaned countTable together with imageFeatures and positionTable to stImage's LoadImageFeature, then proceed to PreProcessing with SCT normalization (pcaDim_s=20, pcaDim_i=20, pcaDim_c=6, prefiltergenePercentCut=0.1, prefilterimagePercentCut=0.1, genePercentCut=0.25, imagePercentCut=0.2). Rationale: ring-pattern genes are image-like artifacts that would distort the gene-expression modality and bias downstream SpatialAwareProcess and MultiModalIntegration (WNN, MCIA, tICA, Spectrum) comparisons of ARI against pathologist annotations.

## Related tools

- **stImage** (R package whose LoadImageFeature/PreProcessing pipeline consumes the ring-gene-filtered count table) — https://github.com/YuWang-VUMC/stImage
- **tidyverse** (R data-manipulation toolkit used in the stImage workflow to subset the gene count matrix)

## Examples

```
countTable <- countTable[!rownames(countTable) %in% ring_genes, ]  # remove the 21 ring pattern genes
```

## Evaluation signals

- The filtered matrix has exactly 21 fewer gene rows than the raw H1.tsv count matrix, and none of the 21 ring pattern gene names remain in rownames(countTable).
- Spot identifiers in the filtered matrix still match the X<x>x<y> format and the labels in H1_labeled_coordinates.tsv, so downstream ARI computation against pathologist annotations is possible.
- The stImage object builds without error through LoadImageFeature and PreProcessing with SCT normalization and the specified dimensionality settings (pcaDim_s=20, pcaDim_i=20, pcaDim_c=6).
- Downstream 6-cluster FindSNNClusters results yield ARI values in the reported benchmark range (e.g., Gene/Image ~0.43-0.44 up to SpatialPCA+WNN 0.62), indicating the filtered expression modality behaves as expected.

## Limitations

- The 21 ring pattern genes are specific to the original HER2 breast cancer paper; the list may not transfer to other tissues or platforms.
- The article notes no single integration approach performed well on every dataset, so filtering alone does not guarantee an optimal strategy for a new sample.

## Evidence

- [methods] ring-gene filter step: "remove the 21 ring pattern genes mentioned in the orginal paper"
- [readme] stImage package context: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images."
- [methods] dataset source: "HER2 breast cancer (Spatial Transcriptomics platform) dataset can be accessed at https://doi.org/10.5281/zenodo.4751624"
