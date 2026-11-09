---
name: r-vignette-reproducible-rendering
description: Reference for reproducing an R package's documented example pipeline
  end-to-end (e.g., re-running a vignette's synthetic MultiAssayExperiment generation
  and plotting) and verifying that locally computed outputs match the vignette's rendered
  figures.
when_to_use_negative:
- The vignette ships pre-rendered figures only and you only need to view examples
  rather than re-execute and verify computed quantities
- The goal is exploratory visualization on your own real data rather than validating
  that a package's documented example reproduces its stated outputs
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3577
tools:
- name: knitr
  role: rmarkdown rendering engine used to execute the vignette code chunks reproducibly
- name: MultiModalGraphics
  role: Package under reproduction; its vignette and plotting classes (ThresholdedScatterplot,
    ThresholdedScatterplot_MAE) are re-executed
  repo: https://github.com/famanalytics0/MultiModalGraphics
- name: MultiAssayExperiment
  role: Bioconductor dependency providing the MAE container for the vignette's synthetic
    multi-assay example
- name: ggplot2
  role: Underlying plotting engine whose returned ggplot objects are inspected to
    extract annotated counts
- name: R
  role: Runtime environment in which the vignette and session info are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-025-06265-3
    title: 'MultiModalGraphics: an R package for graphical integration of multi-omics
      datasets'
  source_content_hash: sha256:d630ce09f6fa28f14eacbbd563417600355117c5cff96c333df3a3d0890ab267
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# r-vignette-reproducible-rendering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Render an R package vignette reproducibly with the knitr::rmarkdown engine so that synthetic data generation, plotting calls, and session info are re-executed rather than read from pre-rendered figures, enabling direct comparison of computed results against annotated vignette outputs.

## When to use

Reference for reproducing an R package's documented example pipeline end-to-end (e.g., re-running a vignette's synthetic MultiAssayExperiment generation and plotting) and verifying that locally computed outputs match the vignette's rendered figures.

## When NOT to use

- The vignette ships pre-rendered figures only and you only need to view examples rather than re-execute and verify computed quantities
- The goal is exploratory visualization on your own real data rather than validating that a package's documented example reproduces its stated outputs

## Inputs

- R package vignette (Rmd, rendered with knitr::rmarkdown engine) from the GitHub repository
- Package installation source (github.com/famanalytics0/MultiModalGraphics)
- Bioconductor dependency list (MultiAssayExperiment, ComplexHeatmap, ggplot2, dplyr, etc.)
- Example data constructors shipped in the vignette (e.g., synthetic 1000-gene MultiAssayExperiment or miniACC)

## Outputs

- Locally re-rendered vignette figures (e.g., thresholded_scatterplot_mae.png)
- Extracted numeric results from rendered panels (e.g., threshold_counts.csv of upregulated/downregulated/neutral counts)
- sessionInfo() output documenting package and R versions
- Comparison verdict (computed counts vs. vignette-annotated counts)

## How to apply

Install the package and its Bioconductor dependencies (MultiModalGraphics from github.com/famanalytics0/MultiModalGraphics plus MultiAssayExperiment, ComplexHeatmap, ggplot2, dplyr), then execute the vignette with the knitr::rmarkdown engine so the synthetic 1000-gene MultiAssayExperiment object and all plotting calls are regenerated from scratch. Load libraries with suppressPackageStartupMessage(library(...)) exactly as shown in the vignette, run the demonstrated class constructors (e.g., ThresholdedScatterplot / ThresholdedScatterplot_MAE with explicit highLog2fc, lowLog2fc, negLog10pValue cutoffs), and extract computed quantities (e.g., up/down/neutral feature counts) from the rendered panels. Compare those extracted counts against the counts annotated in the vignette's own rendered figures to confirm the implementation matches the package output, and record sessionInfo() for reproducibility. Export the final figure (e.g., thresholded_scatterplot_mae.png) and tabular results (e.g., threshold_counts.csv) so the reproduction is auditable.

## Related tools

- **knitr** (rmarkdown rendering engine used to execute the vignette code chunks reproducibly)
- **MultiModalGraphics** (Package under reproduction; its vignette and plotting classes (ThresholdedScatterplot, ThresholdedScatterplot_MAE) are re-executed) — https://github.com/famanalytics0/MultiModalGraphics
- **MultiAssayExperiment** (Bioconductor dependency providing the MAE container for the vignette's synthetic multi-assay example)
- **ggplot2** (Underlying plotting engine whose returned ggplot objects are inspected to extract annotated counts)
- **R** (Runtime environment in which the vignette and session info are executed)

## Examples

```
remotes::install_github("famanalytics0/MultiModalGraphics"); knitr::knit("ThresholdedScatterplot_Example.Rmd")
```

## Evaluation signals

- Extracted per-panel upregulated/downregulated/neutral counts from the locally rendered figure match the counts annotated in the vignette's own rendered figures
- The synthetic MultiAssayExperiment object is regenerated with the documented dimensions (1000 genes) rather than loaded from a cached artifact
- Volcano panels render as faceted log2 fold-change vs. -log10(p-value) plots with up/down/neutral classes separated by the specified cutoffs (e.g., highLog2fc=1.0, lowLog2fc=-1.0, negLog10pValue=1.301, i.e., p < 0.05)
- sessionInfo() output is captured and reports the expected package versions (e.g., MultiModalGraphics 0.99.5)
- Figure and count outputs are written to named files (thresholded_scatterplot_mae.png, threshold_counts.csv) as the task specifies

## Limitations

- The vignette's sessionInfo() appears only as unevaluated code, so exact package/dependency versions are not stated in the rendered text and must be captured at run time
- The vignette claims to include examples (data generation, scatterplots, informative heatmaps, multifeature grids) but no such example content appears in the document body, so reproduction may depend on the repository README/manual instead
- Advanced features and customization are deferred to external ComplexHeatmap and ggplot2 documentation rather than documented in the vignette

## Evidence

- [other] Execute the 'ThresholdedScatterplot Example' vignette (rendered with the knitr::rmarkdown engine) to generate the synthetic 1000-gene MultiAssayExperiment object used for demonstration.: [evidence span withheld — anchor status: not_found]
- [other] Compare the computed counts against the counts annotated in the vignette's rendered figures to confirm the implementation matches the package output: [evidence span withheld — anchor status: not_found]
- [discussion] Vignette claims to include examples ... but no such example content appears in the document body: [evidence span withheld — anchor status: not_found]
