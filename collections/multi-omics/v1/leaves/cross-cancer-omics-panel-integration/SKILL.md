---
name: cross-cancer-omics-panel-integration
description: Use when you need to render a pan-cancer multi-omics AnnotatedHeatmap
  from fold-change and p/q-value matrices with significance stars and annotation tracks,
  e.g. epigenetic-marker vs. gene-expression associations across TCGA cancer panels.
when_to_use_negative:
- The goal is a faceted volcano plot of a single precomputed DE table — use ThresholdedScatterplot
  instead of an AnnotatedHeatmap.
- The input is raw count/expression data with no precomputed fold-change or p-value
  matrices — a differential expression step (limma/voom, DESeq2) must run first.
- Only one omics modality is available, so no cross-modal association panels (mutation/CNV,
  methylation, miRNA-mRNA, protein-mRNA) can be composed.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MultiModalGraphics
  role: Provides the AnnotatedHeatmap S4 class that assembles fold-change/p-value
    matrices with overlaid significance markers and annotation tracks
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: ComplexHeatmap
  role: Rendering engine used to draw the annotated heatmaps and assemble the composite
    legend layout
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: dplyr
  role: Harmonizes feature identifiers across the four extdata CSV tables before matrix
    assembly
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: R
  role: Runtime environment for the package installation, data wrangling, and figure
    export
  repo: https://github.com/famanalytics0/MultiModalGraphics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:ab7a8ef4bf24bd87cff4daaee114fcf522b84381aa388d5455a9a4f4cafc02ab
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/famanalytics0/MultiModalGraphics
  repo_url: https://github.com/famanalytics0/MultiModalGraphics
  tools:
  - ComplexHeatmap
  - MultiModalGraphics
  - R
  - dplyr
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# cross-cancer-omics-panel-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the MultiModalGraphics Fig. 1 composite AnnotatedHeatmap that integrates mutation/CNV, miRNA-mRNA, DNA methylation, and protein-mRNA data into six cancer panels (CESC, OV, PRAD, TGCT, UCEC, UCS) with significance-star overlays and CNA/mutation annotation tracks.

## When to use

Use when you need to render a pan-cancer multi-omics AnnotatedHeatmap from fold-change and p/q-value matrices with significance stars and annotation tracks, e.g. epigenetic-marker vs. gene-expression associations across TCGA cancer panels.

## When NOT to use

- The goal is a faceted volcano plot of a single precomputed DE table — use ThresholdedScatterplot instead of an AnnotatedHeatmap.
- The input is raw count/expression data with no precomputed fold-change or p-value matrices — a differential expression step (limma/voom, DESeq2) must run first.
- Only one omics modality is available, so no cross-modal association panels (mutation/CNV, methylation, miRNA-mRNA, protein-mRNA) can be composed.

## Inputs

- Pan_cancer_CESC_Mutated_CNV_STR_VART.csv (mutations/CNA/STR/VART features)
- Pan_cancer_miRNA-mRNA_interaction_data.csv
- Pan_Cancer_DNA_methylation.csv
- Pan_cancer_protein-mRNA_combined_data.csv
- Fold-change matrix and p-value/q-value matrix keyed by feature and cancer type
- Cancer-panel subset list: CESC, OV, PRAD, TGCT, UCEC, UCS

## Outputs

- Composite multi-omics AnnotatedHeatmap figure (fig1_annotated_heatmap.png) with six cancer panels, significance stars, CNA/mutation annotation tracks, and integrated legends

## How to apply

Install MultiModalGraphics (remotes::install_github) with its Bioconductor dependencies (ComplexHeatmap, MultiAssayExperiment, limma) and load it together with ComplexHeatmap and dplyr. Read the four packaged extdata CSVs (Pan_cancer_CESC_Mutated_CNV_STR_VART.csv, Pan_cancer_miRNA-mRNA_interaction_data.csv, Pan_Cancer_DNA_methylation.csv, Pan_cancer_protein-mRNA_combined_data.csv) and harmonize feature identifiers across the mutation/CNA, miRNA-mRNA, methylation, and protein-mRNA tables with dplyr. Assemble the fold-change and p-value/q-value matrices required by the AnnotatedHeatmap constructor, subsetting to the six cancer panels CESC, OV, PRAD, TGCT, UCEC, and UCS. Call the annotated-heatmap function (built on ComplexHeatmap) applying significance-star thresholds to the q-value/p-value matrices and mapping CNA/mutation columns to row-annotation tracks. Assemble the heatmap-color, annotation-track, and star-significance legends into the composite layout, then render to fig1_annotated_heatmap.png via png().

## Related tools

- **MultiModalGraphics** (Provides the AnnotatedHeatmap S4 class that assembles fold-change/p-value matrices with overlaid significance markers and annotation tracks) — https://github.com/famanalytics0/MultiModalGraphics
- **ComplexHeatmap** (Rendering engine used to draw the annotated heatmaps and assemble the composite legend layout) — https://github.com/famanalytics0/MultiModalGraphics
- **dplyr** (Harmonizes feature identifiers across the four extdata CSV tables before matrix assembly) — https://github.com/famanalytics0/MultiModalGraphics
- **R** (Runtime environment for the package installation, data wrangling, and figure export) — https://github.com/famanalytics0/MultiModalGraphics

## Examples

```
remotes::install_github("famanalytics0/MultiModalGraphics"); suppressPackageStartupMessage(library(MultiModalGraphics)); suppressPackageStartupMessage(library(ComplexHeatmap)); suppressPackageStartupMessage(library(dplyr))
```

## Evaluation signals

- Rendered figure contains all six cancer panels (CESC, OV, PRAD, TGCT, UCEC, UCS) in the composite layout
- Significance stars are overlaid on heatmap cells according to the applied q-value/p-value thresholds
- CNA/mutation columns appear as row-annotation tracks alongside the heatmap body
- Legends for heatmap colors, annotation tracks, and the star-significance key are all present and integrated
- Composition matches the article's Fig. 1 (epigenetic markers vs. proliferation/antiapoptotic gene-expression products)

## Limitations

- The vignette claims heatmap/scatterplot/multifeature-grid examples but no such example content appears in the document body; advanced customization is deferred to ComplexHeatmap and ggplot2 documentation.
- No named datasets or accession identifiers are provided for the demonstrated multi-omics data, and package/dependency versions are not reported (sessionInfo() appears only as unevaluated code).
- The repository README ships incomplete/truncated documentation of the heatmap classes, so exact AnnotatedHeatmap argument names may need to be confirmed from the package manual.

## Evidence

- [abstract] abstract: "We introduce MultiModalGraphics, an R package designed specifically for creating annotated scatterplots and heatmaps of multi-omics and high-dimensional biological data."
- [discussion] discussion: "This vignette demonstrates the integration of complex heatmaps and annotations for biological datasets, providing actionable insights into multi-omics data."
- [intro] intro: "**Informative heatmaps** with overlaid significance markers from: Fold-change & p-value matrices"
- [readme] readme: "A unified R/Bioconductor package for creating publication-quality volcano plots and custom heatmaps from diverse biological data sources."
