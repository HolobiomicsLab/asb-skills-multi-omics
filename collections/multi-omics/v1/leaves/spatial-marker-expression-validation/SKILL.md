---
name: spatial-marker-expression-validation
description: Use when you have computed spot/spot-cluster assignments on a spatial
  transcriptomics dataset and need to confirm clusters correspond to pathologist-annotated
  tissue regions via known marker genes.
when_to_use_negative:
- Clusters have not yet been computed (e.g. FindSNNClusters/MultiModalIntegration
  has not been run) — this skill validates clusters, it does not generate them
- The dataset has no spatial coordinates or histology-informed cluster assignments,
  so marker expression cannot be assessed per spatial region
- Tissue structure is spatially uniform and no region-specific marker pattern is expected,
  so the region-specificity comparison is uninformative
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: stImage
  role: Provides the SpatialPCA+SCT reductions, WNN integration clusters (clusterNum
    = 6), and the AllSpatialFeaturePlot function (normalizeMethod='SCT', integreation.method='wnn',
    cluster.frame=TRUE, cluster.highlight) used to plot marker expression spatially
  repo: https://github.com/YuWang-VUMC/stImage
- name: Seurat
  role: Underlying object/assay framework for the stImage object; default assay 'Spatial'
    with SCT-normalized expression is extracted per spot
  repo: https://satijalab.org/seurat
- name: tidyverse
  role: Used in the methods for data manipulation/aggregation of per-spot marker expression
    across clusters
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf429
    title: 'stImage: a versatile framework for optimizing spatial transcriptomic analysis
      through customizable deep histology and location informed integration'
  source_content_hash: sha256:9f404ba57cd9faac3df3240ebe0cd6cbba8a0833f95dd0cb5a7ab0e9ef6411ed
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# spatial-marker-expression-validation

## Summary

Validates that spatial-transcriptomics clusters (e.g. stImage SpatialPCA+WNN clusters on the HER2 breast cancer H1 dataset) are biologically supported by plotting region-specific marker genes (ERBB2, TIMP1, CFD, CD74) in spatial context and comparing their mean expression across clusters.

## When to use

Use when you have computed spot/spot-cluster assignments on a spatial transcriptomics dataset and need to confirm clusters correspond to pathologist-annotated tissue regions via known marker genes.

## When NOT to use

- Clusters have not yet been computed (e.g. FindSNNClusters/MultiModalIntegration has not been run) — this skill validates clusters, it does not generate them
- The dataset has no spatial coordinates or histology-informed cluster assignments, so marker expression cannot be assessed per spatial region
- Tissue structure is spatially uniform and no region-specific marker pattern is expected, so the region-specificity comparison is uninformative

## Inputs

- stImage/Seurat object with SpatialPCA+SCT and ImageFeature SpatialPCA reductions and WNN multimodal integration clusters (clusterNum = 6)
- SCT-normalized 'Spatial' assay expression values per spot
- Marker gene list matching annotated regions (ERBB2, TIMP1, CFD, CD74 for HER2 breast cancer H1)
- Pathologist-annotated region labels for the H1 tissue section

## Outputs

- Per-marker spatial feature plots (AllSpatialFeaturePlot with cluster.frame=TRUE and cluster.highlight)
- Mean marker expression per WNN cluster comparison table
- Composite figure panel of the four marker plots (Fig. 3C-style layout)

## How to apply

Load the stImage/Seurat object containing the SpatialPCA+SCT and ImageFeature SpatialPCA reductions and the WNN multimodal integration clusters (clusterNum = 6 in the HER2 H1 case). Set the default assay to 'Spatial' and extract SCT-normalized expression for each marker (ERBB2, TIMP1, CFD, CD74) at every spot. For each marker, call stImage's AllSpatialFeaturePlot with normalizeMethod='SCT' and integreation.method='wnn', setting cluster.frame=TRUE, cluster.highlight to the expected cluster (e.g. cluster 3 for ERBB2), and a custom six-color palette. Then compute mean marker expression per WNN cluster and check the region-specific pattern: ERBB2 exclusive to the in-situ cancer cluster, TIMP1 enriched in the invasive cancer cluster, CFD in the adipose cluster, and CD74 in the immune infiltrate cluster. Assemble the four spatial feature plots into a single panel reproducing the Fig. 3C marker-validation layout.

## Related tools

- **stImage** (Provides the SpatialPCA+SCT reductions, WNN integration clusters (clusterNum = 6), and the AllSpatialFeaturePlot function (normalizeMethod='SCT', integreation.method='wnn', cluster.frame=TRUE, cluster.highlight) used to plot marker expression spatially) — https://github.com/YuWang-VUMC/stImage
- **Seurat** (Underlying object/assay framework for the stImage object; default assay 'Spatial' with SCT-normalized expression is extracted per spot) — https://satijalab.org/seurat
- **tidyverse** (Used in the methods for data manipulation/aggregation of per-spot marker expression across clusters)

## Examples

```
AllSpatialFeaturePlot(obj, features="ERBB2", normalizeMethod="SCT", integreation.method="wnn", cluster.frame=TRUE, cluster.highlight=3)
```

## Evaluation signals

- ERBB2 is expressed exclusively in WNN cluster 3, matching the pathologist-annotated in-situ cancer region (Fig. 3C)
- TIMP1 shows enrichment in the invasive cancer cluster, CFD in the adipose cluster, and CD74 in the immune infiltrate cluster
- Feature plots render with normalizeMethod='SCT' and integreation.method='wnn' on the 'Spatial' assay, with cluster.frame=TRUE and the correct cluster.highlight per marker
- Mean marker expression per cluster shows the expected region-specific dominance for all four markers, not a diffuse pattern across clusters
- The four plots assemble into a single panel matching the Fig. 3C marker-validation layout

## Limitations

- No single integration approach performed well on every dataset, so the cluster-to-region mapping (e.g. cluster 3 = in-situ cancer) must be re-derived per dataset rather than assumed
- Validation depends on the availability of pathologist-annotated regions and known region-specific markers; without them the cluster-gene correspondence cannot be judged
- Multi-modal integration is computationally more intensive and algorithm-dependent, though stImage runtime remains manageable on a typical office desktop

## Evidence

- [other] workflow: AllSpatialFeaturePlot usage: [evidence span withheld — anchor status: not_found]
- [methods] HER2 dataset availability: "HER2 breast cancer (Spatial Transcriptomics platform) dataset can be accessed at https://doi.org/10.5281/zenodo.4751624"
- [abstract] stImage scope from abstract: "we introduce stImage, an open-source R package that provides a comprehensive and flexible solution for ST analysis"
- [readme] README description of stImage: "StImage is a R package for integrated analysis of spatial transcriptomics and the corresponding modality images."
- [discussion] caveat on integration strategy: "As we expected, there was no single approach performing well in every dataset."
