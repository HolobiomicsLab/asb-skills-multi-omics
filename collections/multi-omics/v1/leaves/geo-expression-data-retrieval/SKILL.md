---
name: geo-expression-data-retrieval
description: Use when an analysis starts from a GEO accession (e.g., GSE68077) whose
  DE/expression table must be obtained and converted into log2fc / negLog10p form
  for MultiModalGraphics volcano plotting.
when_to_use_negative:
- Input is a raw count matrix plus metadata — use ThresholdedScatterplot_table (limma/voom
  DE under the hood) instead of precomputing a GEO DE table
- Input is already a MultiAssayExperiment object — use ThresholdedScatterplot_MAE
  directly
- Data are not transcriptomic/DE (e.g., pathway feature grids for 2D grid heatmaps),
  so no fold-change/p-value thresholding applies
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: MultiModalGraphics
  role: Consumes the retrieved/prepared DE table via ThresholdedScatterplot and renders
    the faceted, count-annotated volcano plot
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: dplyr
  role: Computes log2fc/negLog10p columns and independently verifies up/down/neutral
    counts under the 0.585 and 1.301 cutoffs
- name: ggplot2
  role: Underlying engine for the faceted volcano plot returned by createPlot; used
    for custom theming of the retrieved data's figure
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:e5def217ffe64cfee335abe04030c4d16ded30ec35c0b250e60aff3b09382233
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
  - MultiModalGraphics
  - R
  - dplyr
  - ggplot2
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.2
n_claims: 10
---

# geo-expression-data-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves a GEO-accession-derived expression/differential-expression table (e.g., the GSE68077-derived extdata/scatterexample.csv shipped in the MultiModalGraphics repo) and prepares log2 fold-change and -log10(p-value) columns so the data can feed MultiModalGraphics' ThresholdedScatterplot volcano workflow.

## When to use

Use when an analysis starts from a GEO accession (e.g., GSE68077) whose DE/expression table must be obtained and converted into log2fc / negLog10p form for MultiModalGraphics volcano plotting.

## When NOT to use

- Input is a raw count matrix plus metadata — use ThresholdedScatterplot_table (limma/voom DE under the hood) instead of precomputing a GEO DE table
- Input is already a MultiAssayExperiment object — use ThresholdedScatterplot_MAE directly
- Data are not transcriptomic/DE (e.g., pathway feature grids for 2D grid heatmaps), so no fold-change/p-value thresholding applies

## Inputs

- GEO accession identifier (e.g., GSE68077) or its derived differential-expression table
- DE table with fold-change and p-value columns (CSV, e.g. extdata/scatterexample.csv from the MultiModalGraphics repository)
- Sample metadata columns: SampleType (tissue/brain region) and optional timePoint

## Outputs

- R data.frame with log2fc, negLog10p, regulation (up/down/neutral), SampleType, timePoint columns
- ThresholdedScatterplot S4 object and ggplot faceted volcano figure (thresholded_scatterplot.png)
- Summary table of per-panel up-/down-regulated gene counts

## How to apply

Obtain the DE table from the GEO accession (GSE68077 in the source task) or the package's bundled example (extdata/scatterexample.csv in famanalytics0/MultiModalGraphics); the package supports 'Precomputed DE tables: (log2FC, –log10 p, regulation, sampleType, [optional timepoint])' as a first-class input. If the raw table only carries fold-change and p-value columns, compute log2fc = log2(FC) and negLog10p = -log10(p-value) in R with dplyr/base R. Apply the fixed significance thresholds used by the task: |log2fc| > 0.585 (~1.5-fold change) and -log10p > 1.301 (p < 0.05) to classify each gene as up, down, or neutral, matching the ThresholdedScatterplot parameters highLog2fc, lowLog2fc, negLog10pValue. Pass the prepared data.frame to ThresholdedScatterplot(...) and createPlot(...), which facets by SampleType and timePoint (in the source task: tissue, brain region, and timepoints T5R1, T5R10, T10R1, T10R28, T10R42) and annotates each panel with up/down gene counts. Validate by cross-checking the per-facet annotations against dplyr::filter counts computed independently on the retrieved table.

## Related tools

- **MultiModalGraphics** (Consumes the retrieved/prepared DE table via ThresholdedScatterplot and renders the faceted, count-annotated volcano plot) — https://github.com/famanalytics0/MultiModalGraphics
- **dplyr** (Computes log2fc/negLog10p columns and independently verifies up/down/neutral counts under the 0.585 and 1.301 cutoffs)
- **ggplot2** (Underlying engine for the faceted volcano plot returned by createPlot; used for custom theming of the retrieved data's figure)

## Examples

```
cs1 <- ThresholdedScatterplot(data = read.csv("extdata/scatterexample.csv"), highLog2fc = 0.585, lowLog2fc = -0.585, negLog10pValue = 1.301); show(createPlot(cs1, title = "GSE68077 faceted volcano"))
```

## Evaluation signals

- Per-facet annotated up/down counts match independent dplyr counts using |log2fc| > 0.585 and negLog10p > 1.301 (p < 0.05)
- negLog10p threshold 1.301 maps correctly to p < 0.05 (i.e., -log10(0.05) = 1.301), confirming no sign or scale error in the computed column
- Output figure thresholded_scatterplot.png contains one panel per SampleType–timePoint cell (e.g., T5R1, T5R10, T10R1, T10R28, T10R42) with no missing facets
- Every point falls in exactly one of three regulation classes (up/down/neutral) with colors matching color1/color2/color3 arguments
- Session reproducibility: script records sessionInfo() and the exact provenance (GSE68077-derived CSV or bundled extdata/scatterexample.csv) of the input table

## Limitations

- The article/vignette itself provides 'No named datasets or accession identifiers provided for the biological / multi-omics data demonstrated', so the GSE68077 linkage comes from the task card, not the paper body
- Package/dependency version numbers are not reported in the vignette text ('session info appears only as unevaluated code'), complicating exact reproduction of the retrieval-to-plot pipeline
- No changelog exists ('No changelog found'), so behavior of ThresholdedScatterplot defaults may drift between commits of the development package

## Evidence

- [readme] Precomputed DE tables as supported input: [evidence span withheld — anchor status: not_found]
- [readme] Threshold parameters for ThresholdedScatterplot: [evidence span withheld — anchor status: external_doc]
- [readme] Annotated facet counts result: [evidence span withheld — anchor status: external_doc]
