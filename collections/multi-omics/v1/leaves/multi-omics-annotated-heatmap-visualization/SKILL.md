---
name: multi-omics-annotated-heatmap-visualization
description: Use when you have fold-change and p-value/q-value matrices (plus metadata
  such as CNA/mutation status) from multi-omics data and need a single annotated heatmap
  with overlaid significance stars and integrated legends.
when_to_use_negative:
- The goal is a volcano plot of differential expression (log2FC vs -log10 p) — use
  the ThresholdedScatterplot class instead
- Input is already a rendered/feature-level expression heatmap needing only clustering
  (e.g., a plain pheatmap use case) with no statistical overlays or annotation tracks
- A 2D tile-and-point pathway grid (z-score tiles, p-value point color, count point
  size) is wanted — use the multifeature grid class instead
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: MultiModalGraphics
  role: Core package providing the AnnotatedHeatmap S4 class with overlaid significance
    markers from fold-change & p-value matrices and MAE + optional iClusterPlus pipelines
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: ComplexHeatmap
  role: Underlying heatmap engine used for rendering and legend assembly of the composite
    multi-panel heatmap
- name: dplyr
  role: Data wrangling to harmonize feature identifiers across the mutation/CNA, miRNA-mRNA,
    DNA methylation, and protein-mRNA tables
- name: MultiAssayExperiment
  role: Bioconductor container interface allowing heatmaps to be built from MAE objects
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:6d38b8e1963ed95bc88c55b6ff02014ecd578c4970221305ea4d48d63bd17f5b
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
  - MultiAssayExperiment
  - MultiModalGraphics
  - R
  - dplyr
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# multi-omics-annotated-heatmap-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Render publication-quality informative heatmaps of multi-omics data using the MultiModalGraphics AnnotatedHeatmap S4 class (built on ComplexHeatmap), which overlays significance markers (star thresholds on q-values/p-values) onto fold-change matrices with CNA/mutation row-annotation tracks, as demonstrated for the pan-cancer panels CESC, OV, PRAD, TGCT, UCEC, and UCS in Fig. 1.

## When to use

Use when you have fold-change and p-value/q-value matrices (plus metadata such as CNA/mutation status) from multi-omics data and need a single annotated heatmap with overlaid significance stars and integrated legends.

## When NOT to use

- The goal is a volcano plot of differential expression (log2FC vs -log10 p) — use the ThresholdedScatterplot class instead
- Input is already a rendered/feature-level expression heatmap needing only clustering (e.g., a plain pheatmap use case) with no statistical overlays or annotation tracks
- A 2D tile-and-point pathway grid (z-score tiles, p-value point color, count point size) is wanted — use the multifeature grid class instead

## Inputs

- Fold-change matrix (e.g., log2FC per feature/cancer type)
- P-value / q-value matrix aligned to the fold-change matrix
- extdata CSV tables: Pan_cancer_CESC_Mutated_CNV_STR_VART.csv, Pan_cancer_miRNA-mRNA_interaction_data.csv, Pan_Cancer_DNA_methylation.csv, Pan_cancer_protein-mRNA_combined_data.csv
- CNA/mutation annotation columns for row-annotation tracks
- Optionally, MultiAssayExperiment objects (MAE + optional iClusterPlus DE pipelines)

## Outputs

- Composite annotated heatmap PNG (e.g., fig1_annotated_heatmap.png) with six cancer panels (CESC, OV, PRAD, TGCT, UCEC, UCS), significance stars, CNA/mutation annotation tracks, and integrated legends
- ComplexHeatmap object reusable for RMarkdown/annotation

## How to apply

Install MultiModalGraphics from GitHub (famanalytics0/MultiModalGraphics) alongside ComplexHeatmap, dplyr, and MultiAssayExperiment, then read the packaged extdata CSVs (e.g., Pan_cancer_CESC_Mutated_CNV_STR_VART.csv, Pan_cancer_miRNA-mRNA_interaction_data.csv, Pan_Cancer_DNA_methylation.csv, Pan_cancer_protein-mRNA_combined_data.csv) and harmonize feature identifiers with dplyr. Assemble the fold-change and p-value/q-value matrices required by the AnnotatedHeatmap constructor, subsetting to the six cancer panels CESC, OV, PRAD, TGCT, UCEC, and UCS. Construct the AnnotatedHeatmap S4 object and apply significance-star thresholds to the q-value/p-value matrices while mapping CNA/mutation columns onto row-annotation tracks, customizing via parameters like pvalue_cutoff, fc_cutoff, and color ramps. Assemble the legends (heatmap colors, annotation tracks, star-significance key) into a composite ComplexHeatmap layout, and render/export the figure with png()/pdf(). The heatmap reveals, as in the article's Fig. 1, significant associations between epigenetic markers (DNA methylation, miRNA) and gene expression products (transcripts and proteins) involved in cellular proliferation and antiapoptotic pathways.

## Related tools

- **MultiModalGraphics** (Core package providing the AnnotatedHeatmap S4 class with overlaid significance markers from fold-change & p-value matrices and MAE + optional iClusterPlus pipelines) — https://github.com/famanalytics0/MultiModalGraphics
- **ComplexHeatmap** (Underlying heatmap engine used for rendering and legend assembly of the composite multi-panel heatmap)
- **dplyr** (Data wrangling to harmonize feature identifiers across the mutation/CNA, miRNA-mRNA, DNA methylation, and protein-mRNA tables)
- **MultiAssayExperiment** (Bioconductor container interface allowing heatmaps to be built from MAE objects)

## Examples

```
suppressPackageStartupMessage(library(MultiModalGraphics)); suppressPackageStartupMessage(library(ComplexHeatmap)); suppressPackageStartupMessage(library(dplyr))
```

## Evaluation signals

- The rendered figure contains all six cancer panels (CESC, OV, PRAD, TGCT, UCEC, UCS) matching the composition of the article's Fig. 1
- Significance stars are overlaid per cell according to the applied q-value/p-value star thresholds
- CNA/mutation annotation tracks are present as row annotations and legends (heatmap colors, annotation tracks, star key) are packaged into the composite layout
- Fold-change and p-value matrices share harmonized feature identifiers across the four extdata CSV modalities after dplyr joins (no dropped/mismatched rows)
- The exported PNG opens with the expected dimensions and the ComplexHeatmap object can be re-drawn via show() without errors

## Limitations

- Advanced features and customization are deferred to external package documentation (ComplexHeatmap, ggplot2); the vignette itself omits the promised example content (data generation, scatterplots, informative heatmaps, multifeature grids)
- No named datasets or accession identifiers are provided in the article/vignette text for the demonstrated biological data; example data must be fetched from the GitHub repository extdata folder
- Package and dependency version numbers are not reported in the vignette (sessionInfo() appears only as unevaluated code); no changelog artifact exists

## Evidence

- [results] The Fig. 1 composite AnnotatedHeatmap reveals significant associations between epigenetic markers and gene expression products (transcripts and proteins) involved in cellular proliferation and antiapoptotic pathways.: [evidence span withheld — anchor status: not_found]
- [readme] README customization tips: main plotting functions accept threshold and color parameters for stringency and visualization.: [evidence span withheld — anchor status: external_doc]
