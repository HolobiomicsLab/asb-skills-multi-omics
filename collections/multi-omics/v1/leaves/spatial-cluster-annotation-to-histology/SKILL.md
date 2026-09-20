---
name: spatial-cluster-annotation-to-histology
description: Use when spot-level clusters from a spatial transcriptomics dataset must
  be biologically interpreted against H&E/pathologist-annotated regions, or when marker-gene
  spatial plots are needed to confirm cluster identities.
when_to_use_negative:
- The histological image is uninformative or conflicts with gene expression — the
  article advises skipping modality integration and using only a spatially aware process,
  so WNN clusters would not be trustworthy for histology matching
- Tissue is spatially uniform with no histologically distinct regions to annotate
  against — rely on gene expression alone (e.g. PCA) instead
- Clusters come from a method other than stImage's SpatialPCA+WNN pipeline with matching
  SCT/ImageFeature reductions — the cluster.highlight highlighting and WNN-integrated
  feature plots assume stImage object structure
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: stImage
  role: Provides SpatialAwareProcess (SpatialPCA), MultiModalIntegration (WNN), AllSpatialDimPlot
    and AllSpatialFeaturePlot used to cluster spots and annotate them against histology
  repo: https://github.com/YuWang-VUMC/stImage
- name: Seurat
  role: Underlying object model for the 'Spatial'/'ImageFeature' assays, SCT normalization
    and cluster label extraction
  repo: https://github.com/satijalab/seurat
- name: SpatialPCA
  role: Spatially aware dimensionality reduction applied separately to gene and image
    modalities before WNN clustering
  repo: https://github.com/shangll123/SpatialPCA
- name: tidyverse
  role: Data wrangling and plotting of per-spot cluster labels and marker expression
    summaries
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:9918fb3af8526376f638833defffac5df4e3964e20e876c63c226ec0be15e27e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuWang-VUMC/stImage
  tools:
  - R
  - Seurat
  - SpatialPCA
  - stImage
  - tidyverse
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# Spatial cluster annotation to histology

## Summary

Annotates stImage-derived SpatialPCA+WNN spatial transcriptomics clusters against pathologist/histology-annotated tissue regions, validating each cluster with region-specific marker genes (e.g. ERBB2 in in-situ cancer, COL12A1-high CAF desmoplasia).

## When to use

Use when spot-level clusters from a spatial transcriptomics dataset must be biologically interpreted against H&E/pathologist-annotated regions, or when marker-gene spatial plots are needed to confirm cluster identities.

## When NOT to use

- The histological image is uninformative or conflicts with gene expression — the article advises skipping modality integration and using only a spatially aware process, so WNN clusters would not be trustworthy for histology matching
- Tissue is spatially uniform with no histologically distinct regions to annotate against — rely on gene expression alone (e.g. PCA) instead
- Clusters come from a method other than stImage's SpatialPCA+WNN pipeline with matching SCT/ImageFeature reductions — the cluster.highlight highlighting and WNN-integrated feature plots assume stImage object structure

## Inputs

- stImage object with 'Spatial' (SCT-normalized counts) and 'ImageFeature' (VGG16 + RGB quantile) assays
- Spot coordinates and H&E image paired to the ST dataset (e.g. PDAC-A from GEO GSE111672; HER2 H1 from Zenodo 10.5281/zenodo.4751624)
- SpatialPCA reductions ('SCTSpatialPCA', 'ImageFeatureSpatialPCA') and WNN cluster labels per spot
- Pathologist/histology-annotated tissue region labels and candidate marker genes (ERBB2, TIMP1, CFD, CD74, COL12A1)

## Outputs

- Per-spot WNN cluster labels exported as a table
- Spatial scatter plots of gene-only SpatialPCA, image-only SpatialPCA, and SpatialPCA+WNN clusters (AllSpatialDimPlot panel)
- Spatial feature plots of marker genes with cluster highlighting (AllSpatialFeaturePlot panel, e.g. Fig. 3C-style layout)
- Cluster-to-histology annotation mapping (e.g. cluster 7 = cancer clone A, cluster 5 = clone B, cluster 4 = COL12A1-high CAF/desmoplasia, cluster 2 = ductal epithelium, cluster 0 = stroma)

## How to apply

Run SpatialAwareProcess (method='SpatialPCA') separately on the SCT gene assay (pcaDim_g=20, percentCut=0.25) and the VGG16 ImageFeature assay (pcaDim_i=20, percentCut=0.2), then integrate with MultiModalIntegration (MultimodalMethod='WNN') to obtain per-spot WNN cluster labels. Visualize clusters over tissue with AllSpatialDimPlot and align them to the annotated domains — e.g. in PDAC-A, clusters 4/5/7 should fall exclusively in cancer cells and desmoplasia, with clone A in cluster 7 distinguished from clone B in cluster 5, ductal epithelium in cluster 2, and stroma in cluster 0 (Fig. 4 layout). Validate each annotation by extracting SCT-normalized expression of region markers on the 'Spatial' assay and plotting with AllSpatialFeaturePlot (normalizeMethod='SCT', integreation.method='wnn', cluster.frame=TRUE, cluster.highlight=<expected cluster>): ERBB2 exclusive to the in-situ cancer cluster (3 in HER2 H1), TIMP1 in invasive cancer, CFD in adipose, CD74 in immune infiltrate, and COL12A1-high in the CAF/desmoplasia cluster. The rationale is that WNN integration of SpatialPCA gene and image embeddings yields clusters whose spatial extent matches histology, so marker specificity within the WNN partition (not the gene-only partition) is the acceptance test.

## Related tools

- **stImage** (Provides SpatialAwareProcess (SpatialPCA), MultiModalIntegration (WNN), AllSpatialDimPlot and AllSpatialFeaturePlot used to cluster spots and annotate them against histology) — https://github.com/YuWang-VUMC/stImage
- **Seurat** (Underlying object model for the 'Spatial'/'ImageFeature' assays, SCT normalization and cluster label extraction) — https://github.com/satijalab/seurat
- **SpatialPCA** (Spatially aware dimensionality reduction applied separately to gene and image modalities before WNN clustering) — https://github.com/shangll123/SpatialPCA
- **tidyverse** (Data wrangling and plotting of per-spot cluster labels and marker expression summaries)

## Examples

```
AllSpatialFeaturePlot(st, features='ERBB2', normalizeMethod='SCT', integreation.method='wnn', cluster.frame=TRUE, cluster.highlight=3)
```

## Evaluation signals

- PDAC-A acceptance: reproduced SpatialPCA+WNN map reproduces the Fig. 4 domain layout — clusters 4/5/7 exclusively within cancer cells and desmoplasia, cluster 2 in ductal epithelium, cluster 0 in stroma
- Clone separation: cluster 7 (clone A) spatially distinguishable from cluster 5 (clone B) within the cancer/desmoplasia region
- HER2 H1 acceptance: ERBB2 is expressed exclusively in WNN cluster 3, which coincides with the pathologist-annotated in-situ cancer region (Fig. 3C)
- Marker specificity check: mean SCT-normalized expression of TIMP1, CFD, CD74 is each enriched in its expected WNN cluster (invasive cancer, adipose, immune infiltrate) rather than spread across clusters
- Per-spot cluster label export matches the AllSpatialDimPlot spatial rendering (label count and cluster IDs consistent)

## Limitations

- No single integration approach performed well on every dataset tested, so cluster-to-histology alignment achieved on PDAC-A or HER2 H1 may not transfer to other tissues
- Multi-modal (WNN) integration is computationally more intensive and algorithm-dependent, though runtime remains manageable on a typical office desktop
- For datasets like the Adult Mouse Olfactory Bulb, the 21 ring pattern genes mentioned in the original paper must be removed before clustering, otherwise ring artifacts can confound cluster annotation

## Evidence

- [abstract] By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information.: "By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information."
- [discussion] we recommend applying a spatial aware process followed by modality integration to fully leverage the complementary information: "we recommend applying a spatial aware process followed by modality integration to fully leverage the complementary information"
- [methods] HER2 breast cancer (Spatial Transcriptomics platform) dataset can be accessed at https://doi.org/10.5281/zenodo.4751624: "HER2 breast cancer (Spatial Transcriptomics platform) dataset can be accessed at https://doi.org/10.5281/zenodo.4751624"
- [methods] remove the 21 ring pattern genes mentioned in the orginal paper: "remove the 21 ring pattern genes mentioned in the orginal paper"
- [readme] StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images.: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images."
