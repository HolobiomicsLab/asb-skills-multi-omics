---
name: cna-mutation-annotation-track-composition
description: Use when building a multi-omics annotated heatmap that must carry significance
  markers (stars from p/q-value cutoffs) plus mutation/CNA row-annotation tracks across
  multiple cancer panels from fold-change and p-value matrices.
when_to_use_negative:
- Input is a raw expression/count matrix needing differential expression first — use
  the package's matrix+metadata or MultiAssayExperiment heatmap pipeline instead of
  precomputed-FC/p-value assembly
- The goal is a faceted volcano plot (ThresholdedScatterplot) rather than a heatmap
  with annotation tracks
- Only a single omics modality is involved and no mutation/CNA annotation overlay
  is needed — a plain ComplexHeatmap call is simpler
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: MultiModalGraphics
  role: Provides the AnnotatedHeatmap S4 class and constructor used to assemble the
    composite significance-starred heatmap with annotation tracks
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: ComplexHeatmap
  role: Underlying engine for heatmap rendering, row-annotation tracks, and legend
    assembly in the composite layout
- name: dplyr
  role: Harmonizes feature identifiers across the four extdata CSV tables (mutation/CNA,
    miRNA-mRNA, DNA methylation, protein-mRNA) before matrix assembly
- name: R
  role: Runtime environment for the whole workflow, including extdata CSV ingestion
    and png() export
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:ae16f0905ed7557b990434e12a77e3de2c2683d419aee1656434a1c135deb43f
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
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cna-mutation-annotation-track-composition

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Composes a pan-cancer composite AnnotatedHeatmap (MultiModalGraphics/ComplexHeatmap) in which significance stars overlay fold-change/q-value matrices while CNA and mutation status are mapped to row-annotation tracks, reproducing the article's Fig. 1 across six TCGA cancer types (CESC, OV, PRAD, TGCT, UCEC, UCS).

## When to use

Use when building a multi-omics annotated heatmap that must carry significance markers (stars from p/q-value cutoffs) plus mutation/CNA row-annotation tracks across multiple cancer panels from fold-change and p-value matrices.

## When NOT to use

- Input is a raw expression/count matrix needing differential expression first — use the package's matrix+metadata or MultiAssayExperiment heatmap pipeline instead of precomputed-FC/p-value assembly
- The goal is a faceted volcano plot (ThresholdedScatterplot) rather than a heatmap with annotation tracks
- Only a single omics modality is involved and no mutation/CNA annotation overlay is needed — a plain ComplexHeatmap call is simpler

## Inputs

- Pan_cancer_CESC_Mutated_CNV_STR_VART.csv (mutation/CNA/STR/VART table from MultiModalGraphics extdata)
- Pan_cancer_miRNA-mRNA_interaction_data.csv
- Pan_Cancer_DNA_methylation.csv
- Pan_cancer_protein-mRNA_combined_data.csv
- Fold-change and p-value/q-value matrices keyed by shared feature identifiers
- Cancer-panel selection: CESC, OV, PRAD, TGCT, UCEC, UCS

## Outputs

- Composite AnnotatedHeatmap figure (fig1_annotated_heatmap.png) with six cancer panels, significance stars, CNA/mutation row-annotation tracks, and integrated legends

## How to apply

Load MultiModalGraphics, ComplexHeatmap, and dplyr, then read the four packaged extdata CSVs (Pan_cancer_CESC_Mutated_CNV_STR_VART.csv, Pan_cancer_miRNA-mRNA_interaction_data.csv, Pan_Cancer_DNA_methylation.csv, Pan_cancer_protein-mRNA_combined_data.csv) and harmonize feature identifiers across the mutation/CNA, miRNA-mRNA, DNA methylation, and protein-mRNA tables with dplyr. Assemble the fold-change and p-value/q-value matrices required by the AnnotatedHeatmap constructor and subset to the six cancer panels CESC, OV, PRAD, TGCT, UCEC, and UCS. Construct the AnnotatedHeatmap object, mapping CNA/mutation columns to row-annotation tracks and applying significance-star thresholds to the q-value/p-value matrices (thresholds such as pvalue_cutoff and fc_cutoff are user-settable per the README's customization tips). Package the legends — heatmap colors, annotation tracks, and the star-significance key — into the composite layout via ComplexHeatmap legend assembly, then render to fig1_annotated_heatmap.png. Validate that all six cancer panels appear with overlaid stars, annotation tracks, and integrated legends matching the article's Fig. 1 composition.

## Related tools

- **MultiModalGraphics** (Provides the AnnotatedHeatmap S4 class and constructor used to assemble the composite significance-starred heatmap with annotation tracks) — https://github.com/famanalytics0/MultiModalGraphics
- **ComplexHeatmap** (Underlying engine for heatmap rendering, row-annotation tracks, and legend assembly in the composite layout)
- **dplyr** (Harmonizes feature identifiers across the four extdata CSV tables (mutation/CNA, miRNA-mRNA, DNA methylation, protein-mRNA) before matrix assembly)
- **R** (Runtime environment for the whole workflow, including extdata CSV ingestion and png() export)

## Examples

```
suppressPackageStartupMessage(library(MultiModalGraphics)); suppressPackageStartupMessage(library(ComplexHeatmap)); suppressPackageStartupMessage(library(dplyr)); hm <- AnnotatedHeatmap(fc_matrix, pvalue_matrix); png('fig1_annotated_heatmap.png'); draw(hm)
```

## Evaluation signals

- Rendered figure contains all six cancer panels (CESC, OV, PRAD, TGCT, UCEC, UCS)
- Significance stars appear overlaid on cells according to the applied q-value/p-value thresholds
- CNA/mutation row-annotation tracks are present and correctly aligned to heatmap rows
- Composite layout includes integrated legends for heatmap colors, annotation tracks, and the star-significance key
- Figure matches the composition of the article's Fig. 1 (pan-cancer AnnotatedHeatmap)

## Limitations

- The vignette claims heatmap examples but no such example content appears in the document body; advanced customization is deferred to ComplexHeatmap/ggplot2 documentation
- No named datasets or accession identifiers are provided in the vignette text; data must come from the GitHub extdata CSVs
- Package and dependency versions are not reported (sessionInfo appears only as unevaluated code); README lists version 0.99.5
- Thresholds (highLog2fc, lowLog2fc, pvalue_cutoff, fc_cutoff) and color ramps are user-settable, so star placement depends on chosen cutoffs

## Evidence

- [readme] README AnnotatedHeatmap data sources: "**Informative heatmaps** with overlaid significance markers from:
  - Fold-change & p-value matrices"
- [readme] README thresholds customization: [evidence span withheld — anchor status: external_doc]
- [abstract] Article abstract on package purpose: "We introduce MultiModalGraphics, an R package designed specifically for creating annotated scatterplots and heatmaps of multi-omics and high-dimensional biological data."
- [abstract] Article abstract on embedded statistics: "The package allows seamless embedding of statistical summaries such as fold-changes, p-values, q-values, and standard deviations"
- [discussion] Vignette library loading: "suppressPackageStartupMessage(library(MultiModalGraphics))"
- [discussion] Vignette missing heatmap example caveat: [evidence span withheld — anchor status: not_found]
