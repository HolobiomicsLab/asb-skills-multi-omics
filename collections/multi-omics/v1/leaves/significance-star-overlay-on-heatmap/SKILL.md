---
name: significance-star-overlay-on-heatmap
description: Use when you have fold-change and p-value/q-value matrices from multi-omics
  data (mutations, CNA, methylation, miRNA-mRNA, protein-mRNA) and need a heatmap
  with overlaid significance markers.
when_to_use_negative:
- The goal is a volcano plot of log2 fold-change vs. -log10(p-value) rather than a
  heatmap — use ThresholdedScatterplot instead
- The input is raw expression data without precomputed fold-change and p-value matrices
  and no MAE pipeline is desired
- A 2D tile-and-point grid of z-scores, p-values, and counts is needed — use the multifeature
  grid class instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0091
tools:
- name: MultiModalGraphics
  role: Provides the AnnotatedHeatmap S4 class and createPlot/show methods for heatmaps
    with overlaid significance markers
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: ComplexHeatmap
  role: Underlying engine for rendering the heatmap, annotation tracks, and composite
    legend assembly
- name: dplyr
  role: Harmonizing feature identifiers across multi-omics input tables before matrix
    assembly
- name: MultiAssayExperiment
  role: Optional input container; heatmaps can be built from MAE objects with optional
    iClusterPlus DE pipelines
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:f251feafd9716c4367e6862f3a24b4265ac90df175ace70d46834d7545b8b391
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# significance-star-overlay-on-heatmap

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Create informative multi-omics heatmaps with overlaid significance markers (stars) using the MultiModalGraphics AnnotatedHeatmap class, which embeds fold-change, p-value, and q-value summaries directly on the plot. Built on ComplexHeatmap, it enables direct quantitative comparison across omics modalities.

## When to use

Use when you have fold-change and p-value/q-value matrices from multi-omics data (mutations, CNA, methylation, miRNA-mRNA, protein-mRNA) and need a heatmap with overlaid significance markers.

## When NOT to use

- The goal is a volcano plot of log2 fold-change vs. -log10(p-value) rather than a heatmap — use ThresholdedScatterplot instead
- The input is raw expression data without precomputed fold-change and p-value matrices and no MAE pipeline is desired
- A 2D tile-and-point grid of z-scores, p-values, and counts is needed — use the multifeature grid class instead

## Inputs

- Fold-change matrix (features x conditions)
- P-value and/or q-value matrix aligned to the fold-change matrix
- Precomputed DE tables or MultiAssayExperiment (MAE) objects (e.g., miniACC)
- Auxiliary annotation tables (e.g., CNA/mutation status columns, extdata CSVs such as Pan_Cancer_DNA_methylation.csv)

## Outputs

- Composite annotated heatmap (PNG/PDF) with overlaid significance stars on features
- Row/column annotation tracks (e.g., CNA/mutation status)
- Integrated legends for heatmap colors, annotation tracks, and star-significance key

## How to apply

Install MultiModalGraphics (remotes::install_github("famanalytics0/MultiModalGraphics")) together with ComplexHeatmap and dplyr. Harmonize feature identifiers across the input tables (e.g., mutation/CNA, miRNA-mRNA interaction, DNA methylation, protein-mRNA CSVs) with dplyr, then assemble the fold-change and p-value/q-value matrices required by the AnnotatedHeatmap constructor. Call the MultiModalGraphics annotated-heatmap function with significance-star thresholds applied to the q-value/p-value matrices, mapping auxiliary columns (e.g., CNA/mutation status) to row-annotation tracks. Assemble heatmap-color, annotation-track, and star-significance legends into a composite ComplexHeatmap layout, then export with png() or pdf(). The article reports that MultiModalGraphics interoperates with Bioconductor packages including MultiAssayExperiment, limma, voom, and iClusterPlus, so matrices can also be derived from MAE objects via optional iClusterPlus DE pipelines.

## Related tools

- **MultiModalGraphics** (Provides the AnnotatedHeatmap S4 class and createPlot/show methods for heatmaps with overlaid significance markers) — https://github.com/famanalytics0/MultiModalGraphics
- **ComplexHeatmap** (Underlying engine for rendering the heatmap, annotation tracks, and composite legend assembly)
- **dplyr** (Harmonizing feature identifiers across multi-omics input tables before matrix assembly)
- **MultiAssayExperiment** (Optional input container; heatmaps can be built from MAE objects with optional iClusterPlus DE pipelines)

## Examples

```
suppressPackageStartupMessage(library(MultiModalGraphics)); suppressPackageStartupMessage(library(ComplexHeatmap)); ah <- AnnotatedHeatmap(fc_matrix, pval_matrix); ah <- createPlot(ah); show(ah)
```

## Evaluation signals

- Rendered figure contains all expected panels (e.g., six cancer panels CESC, OV, PRAD, TGCT, UCEC, UCS) with overlaid significance stars
- CNA/mutation annotation tracks are present as row annotations alongside the heatmap body
- Legends for heatmap colors, annotation tracks, and the star-significance key are all integrated in the composite layout
- Significance stars correctly reflect the p-value/q-value cutoffs applied to the significance matrix
- Feature identifiers are consistently harmonized across all input omics tables (no unmatched rows silently dropped)

## Limitations

- For advanced features and customization, users are deferred to the ComplexHeatmap and ggplot2 documentation
- No named datasets or accession identifiers are provided in the vignette for the demonstrated biological data
- Package and dependency version numbers are not reported in the document text; sessionInfo() appears only as unevaluated code
- No changelog artifact is available

## Evidence
