---
name: her2-breast-cancer-marker-biology
description: Use when HER2 H1 stImage objects with WNN multimodal integration clusters
  (clusterNum = 6) must be biologically validated against pathologist-annotated regions
  via spatial marker gene plots.
when_to_use_negative:
- The stImage object lacks WNN multimodal integration clusters or the expected SpatialPCA+SCT
  / ImageFeature reductions, since cluster.highlight and integreation.method='wnn'
  depend on them
- The dataset is not the HER2 breast cancer ST dataset (e.g. Adult Mouse Olfactory
  Bulb), because the marker-region expectations (ERBB2 in in-situ cancer, CFD in adipose)
  are HER2-tissue-specific
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2830
tools:
- name: stImage
  role: R package providing AllSpatialFeaturePlot, SCT normalization, WNN multimodal
    integration clusters, and the six-color cluster palette used for marker visualization
  repo: https://github.com/YuWang-VUMC/stImage
- name: Seurat
  role: Underlying object framework for assays (default assay 'Spatial'), SCT normalization,
    and clustered spatial data manipulation
  repo: https://github.com/satijalab/seurat
- name: tidyverse
  role: Data manipulation and plotting for extracting marker expression per spot and
    assembling figure panels
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:c4a9f4d05c9ecad86c6767ccace6bbb17fb4200d5f52fa416789cbc0c5c10fe1
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# her2-breast-cancer-marker-biology

## Summary

Validates stImage's SpatialPCA+WNN clusters on the HER2-positive breast cancer H1 Spatial Transcriptomics dataset by confirming that marker genes (ERBB2, TIMP1, CFD, CD74) show region-specific expression matching the pathologist's annotations, as in the paper's Fig. 3C.

## When to use

Use when HER2 H1 stImage objects with WNN multimodal integration clusters (clusterNum = 6) must be biologically validated against pathologist-annotated regions via spatial marker gene plots.

## When NOT to use

- The stImage object lacks WNN multimodal integration clusters or the expected SpatialPCA+SCT / ImageFeature reductions, since cluster.highlight and integreation.method='wnn' depend on them
- The dataset is not the HER2 breast cancer ST dataset (e.g. Adult Mouse Olfactory Bulb), because the marker-region expectations (ERBB2 in in-situ cancer, CFD in adipose) are HER2-tissue-specific

## Inputs

- HER2 H1 stImage object (Spatial Transcriptomics platform) with SpatialPCA+SCT and ImageFeature SpatialPCA reductions and WNN integration clusters (clusterNum = 6)
- Pathologist-annotated tissue region labels (in-situ cancer, invasive cancer, adipose, immune infiltrate)
- Marker gene list: ERBB2, TIMP1, CFD, CD74

## Outputs

- Spatial feature plots per marker (AllSpatialFeaturePlot with cluster.frame=TRUE and cluster.highlight)
- Per-cluster mean marker expression comparison table
- Assembled four-panel figure reproducing the Fig. 3C marker-validation layout

## How to apply

Load the HER2 H1 stImage object containing the SpatialPCA+SCT and ImageFeature SpatialPCA reductions and the WNN clusters (clusterNum = 6), set the default assay to 'Spatial', and extract SCT-normalized expression of ERBB2, TIMP1, CFD, and CD74 for every spot. For each marker, run stImage's AllSpatialFeaturePlot with normalizeMethod='SCT' and integreation.method='wnn', setting cluster.frame=TRUE and cluster.highlight to the expected cluster (cluster 3 for ERBB2) with the custom six-color palette. Then compare mean marker expression across WNN clusters to verify the region-specific pattern: ERBB2 exclusive to the in-situ cancer cluster, TIMP1 enriched in the invasive cancer cluster, CFD in the adipose cluster, and CD74 in the immune infiltrate cluster. Finally assemble the four spatial feature plots into a single figure panel reproducing the Fig. 3C marker-validation layout.

## Related tools

- **stImage** (R package providing AllSpatialFeaturePlot, SCT normalization, WNN multimodal integration clusters, and the six-color cluster palette used for marker visualization) — https://github.com/YuWang-VUMC/stImage
- **Seurat** (Underlying object framework for assays (default assay 'Spatial'), SCT normalization, and clustered spatial data manipulation) — https://github.com/satijalab/seurat
- **tidyverse** (Data manipulation and plotting for extracting marker expression per spot and assembling figure panels)

## Examples

```
DefaultAssay(obj) <- 'Spatial'; AllSpatialFeaturePlot(obj, features = 'ERBB2', normalizeMethod = 'SCT', integreation.method = 'wnn', cluster.frame = TRUE, cluster.highlight = 3)
```

## Evaluation signals

- ERBB2 expression is exclusive to WNN cluster 3, which corresponds to the in-situ cancer region in the pathologist's annotations (Fig. 3C)
- TIMP1 is enriched in the invasive cancer cluster, CFD in the adipose cluster, and CD74 in the immune infiltrate cluster, with mean expression visibly higher in the expected cluster than in the other five clusters
- AllSpatialFeaturePlot runs use normalizeMethod='SCT' and integreation.method='wnn' with cluster.frame=TRUE and the custom six-color palette matching clusterNum = 6
- The four marker plots assemble into a figure panel matching the Fig. 3C layout of the published HER2 H1 analysis

## Limitations

- The validation is dataset-specific: region-marker correspondence (ERBB2/TIMP1/CFD/CD74) is defined for the HER2 H1 breast cancer ST sample and pathologist annotations, not generalizable to other tissues
- Interpretation depends on cluster numbering: cluster 3 must be identified as the in-situ cancer cluster; cluster identities should be confirmed against pathologist annotations before highlighting
- The HER2 dataset must be obtained separately (Zenodo doi:10.5281/zenodo.4751624); it is not bundled with the package

## Evidence

- [results] ERBB2 was exclusively expressed in SpatialPCA+WNN cluster 3, which corresponded to the in-situ cancer region in the pathologist's annotations (Fig. 3C): [evidence span withheld — anchor status: not_found]
- [methods] HER2 breast cancer (Spatial Transcriptomics platform) dataset can be accessed at https://doi.org/10.5281/zenodo.4751624: "HER2 breast cancer (Spatial Transcriptomics platform) dataset can be accessed at https://doi.org/10.5281/zenodo.4751624"
- [readme] stImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images."
- [abstract] By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information: "By generating deep learning–derived histology features and offering 54 integrative strategies, stImage seamlessly combines transcriptional profiles, histology images, and spatial information."
