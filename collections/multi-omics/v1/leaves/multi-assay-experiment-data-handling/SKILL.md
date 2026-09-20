---
name: multi-assay-experiment-data-handling
description: Use when the input is a MultiAssayExperiment (MAE) object and the goal
  is a thresholded, faceted volcano scatterplot with per-panel upregulated/downregulated/neutral
  feature counts rather than plotting a precomputed DE table or raw matrix.
when_to_use_negative:
- Input is already a precomputed DE table — use the plain ThresholdedScatterplot class
  on the DE table instead of the MAE path.
- Input is a raw expression/count matrix plus metadata without multi-assay structure
  — use ThresholdedScatterplot_table, which runs voom+limma per SampleType–timePoint
  cell.
- The goal is a heatmap with significance markers rather than a volcano scatterplot
  — use AnnotatedHeatmap or CompositeFeatureHeatmap instead.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: MultiModalGraphics
  role: Provides the ThresholdedScatterplot_MAE class and createPlot() method that
    consume MAE objects and produce threshold-annotated faceted volcano plots
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: MultiAssayExperiment
  role: Bioconductor container supplying the multi-omics input object consumed by
    ThresholdedScatterplot_MAE
- name: ggplot2
  role: Rendering engine underlying the faceted volcano plot; theme and legend customization
    (theme_minimal(), legend_position, legend_labels) is passed through createPlot()
- name: limma
  role: Differential expression backend run per SampleType-timePoint cell when DE
    is computed from MAE/matrix inputs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:948c453c479bf334f21494a07b8405ab9da7c2d2618e4fdce056bf0d25905ea3
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
  - MultiAssayExperiment
  - MultiModalGraphics
  - R
  - ggplot2
  - knitr
  - limma
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# multi-assay-experiment-data-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs and visualizes faceted volcano plots (log2 fold-change vs. -log10 p-value) directly from MultiAssayExperiment objects using MultiModalGraphics' ThresholdedScatterplot_MAE class, with fold-change/p-value cutoffs and per-panel up/down/neutral counts embedded as annotations.

## When to use

Use when the input is a MultiAssayExperiment (MAE) object and the goal is a thresholded, faceted volcano scatterplot with per-panel upregulated/downregulated/neutral feature counts rather than plotting a precomputed DE table or raw matrix.

## When NOT to use

- Input is already a precomputed DE table — use the plain ThresholdedScatterplot class on the DE table instead of the MAE path.
- Input is a raw expression/count matrix plus metadata without multi-assay structure — use ThresholdedScatterplot_table, which runs voom+limma per SampleType–timePoint cell.
- The goal is a heatmap with significance markers rather than a volcano scatterplot — use AnnotatedHeatmap or CompositeFeatureHeatmap instead.

## Inputs

- MultiAssayExperiment (MAE) object (e.g. miniACC from MultiAssayExperiment, or a synthetic 1000-gene MAE)
- Assay name within the MAE (e.g. 'RNASeq2GeneNorm')
- Metadata column names: groupColumn, sampleType, timepoint
- Cutoff parameters: highLog2fc, lowLog2fc, negLog10pValue (e.g. 1.0, -1.0, 1.301), pvalue_cutoff, fc_cutoff
- Optional BPPARAM parallel backend (e.g. MulticoreParam(workers = 2))

## Outputs

- Faceted volcano scatterplot (log2 fold-change vs. -log10 p-value) with per-panel upregulated/downregulated/neutral feature counts annotated
- ThresholdedScatterplot_MAE S4/ggplot object for further customization or export via pdf()/png()
- Extracted per-panel threshold counts (e.g. written to threshold_counts.csv)
- Final rendered figure (e.g. thresholded_scatterplot_mae.png)

## How to apply

Install MultiModalGraphics from GitHub (famanalytics0/MultiModalGraphics) together with the MultiAssayExperiment Bioconductor dependency. Construct a ThresholdedScatterplot_MAE object by supplying the MAE object plus assayName (e.g. 'RNASeq2GeneNorm'), groupColumn, sampleType, and timepoint column names, dataType='auto', and an optional var_quantile (0.75 in the README example) to filter low-variance features; optionally enable parallel=TRUE with a BPPARAM such as MulticoreParam(workers=2). Then call createPlot() with color1/color2/color3 for down/neutral/up classes, supplying statistical cutoffs (highLog2fc, lowLog2fc, negLog10pValue, e.g. |log2FC| > 1 and -log10(p) > 1.301 for p < 0.05) so features are thresholded into upregulated, downregulated, and neutral classes per panel. Rationale: the MAE input path lets the package run DE internally and stitch results into the same annotated faceted volcano produced from precomputed DE tables, keeping per-panel counts embedded in the figure; counts can be extracted to CSV and cross-checked against the vignette's rendered figures.

## Related tools

- **MultiModalGraphics** (Provides the ThresholdedScatterplot_MAE class and createPlot() method that consume MAE objects and produce threshold-annotated faceted volcano plots) — https://github.com/famanalytics0/MultiModalGraphics
- **MultiAssayExperiment** (Bioconductor container supplying the multi-omics input object consumed by ThresholdedScatterplot_MAE)
- **ggplot2** (Rendering engine underlying the faceted volcano plot; theme and legend customization (theme_minimal(), legend_position, legend_labels) is passed through createPlot())
- **limma** (Differential expression backend run per SampleType-timePoint cell when DE is computed from MAE/matrix inputs)

## Examples

```
cs3 <- ThresholdedScatterplot_MAE(mae=miniACC, assayName="RNASeq2GeneNorm", groupColumn="C1A.C1B", sampleType="pathologic_stage", timepoint="MethyLevel", var_quantile=0.75, parallel=TRUE, BPPARAM=MulticoreParam(workers=2)); cs3 <- createPlot(cs3, color1="darkred", color2="grey70", color3="darkblue")
```

## Evaluation signals

- Rendered panels show three thresholded classes (upregulated, downregulated, neutral) colored by color1/color2/color3, with counts of 'up' and 'down' annotated in each facet.
- Recomputed counts (e.g. features with log2FC > 1, < -1, and -log10(p) > 1.301) match the counts annotated in the rendered figure and agree with the vignette's figures.
- The MAE path (ThresholdedScatterplot_MAE) produces the same faceted log2FC vs -log10(p) panel layout as the precomputed-table path when run on equivalent inputs.
- Per-panel counts can be extracted reproducibly to threshold_counts.csv and compared across runs.
- Final figure exports cleanly (e.g. thresholded_scatterplot_mae.png) with axis labels log2 fold-change and -log10(p-value).

## Limitations

- Advanced features and customization are not documented in the vignette; readers are deferred to external package documentation (ComplexHeatmap, ggplot2).
- Package and dependency version numbers are not reported in the vignette text; session info appears only as unevaluated code.
- No named datasets or accession identifiers are provided for the biological/multi-omics data demonstrated in the vignette.
- With parallel=TRUE the user must supply a BPPARAM backend; README examples use MulticoreParam(workers = 2), and low-variance genes are filtered by var_quantile (0.75) which changes which features appear.

## Evidence

- [other] The ThresholdedScatterplot class automatically applies fold-change and p-value cutoffs and annotates, within each panel, the number of upregulated, downregulated, and neutral features.: [evidence span withheld — anchor status: not_found]
- [readme] README: createPlot annotates up/down counts in each facet: [evidence span withheld — anchor status: external_doc]
