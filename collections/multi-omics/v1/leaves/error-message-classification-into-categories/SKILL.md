---
name: error-message-classification-into-categories
description: Use when benchmarking single-cell simulation methods and you need to
  capture, categorize, and quantify runtime errors per method — e.g., computing per-method
  error rates across 152 reference datasets in a robustness audit.
when_to_use_negative:
- The analysis goal is scoring simulation accuracy, functionality, scalability, or
  usability rather than robustness to execution failures
- A single method failed during interactive use and needs debugging rather than systematic
  taxonomy-based categorization
- The simulations completed without raising errors, so there is no message to classify
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: simpipe
  role: Standard pipeline that runs the 49 simulation methods on reference datasets
    and lets every raised error message be captured verbatim inside the benchmark
    loop
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: R collection of the 49 single-cell/SRT simulation methods whose execution
    errors are captured and categorized
  repo: https://github.com/duohongrui/simmethods
- name: simpipe2docker
  role: Links the R session to the simpipe Docker container so error behavior is consistent
    across all 49 methods
  repo: https://github.com/duohongrui/simpipe2docker
- name: tislingshot
  role: Enables trajectory-informed simulation attempts so errors from trajectory
    scenarios are also captured and classified
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:b6c97415c1dc1c2375d20f3de4728cc77b2e50a603aac4d434f1a1027ce0a857
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simpipe
  tools:
  - Docker
  - Lun
  - MFA
  - Phenopath
  - R
  - SPARSim
  - SRTsim
  - Simple
  - ZINB-WaVE
  - devtools
  - scDesign2
  - scDesign3
  - scDesign3-tree
  - simmethods
  - simpipe
  - simpipe2docker
  - tislingshot
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7368421052631579
n_claims: 19
---

# error-message-classification-into-categories

## Summary

Captures verbatim execution errors raised by the 49 single-cell/spatial transcriptomics simulation methods in simmethods when run through simpipe, classifies each message into one of 10 predefined categories (Additional file 2 Table S5), and computes per-method error rates (paper range 0.7% for SCRIP-BGP-trendedBCV/ZINB-WaVE to 88.9% for BASiCS).

## When to use

Use when benchmarking single-cell simulation methods and you need to capture, categorize, and quantify runtime errors per method — e.g., computing per-method error rates across 152 reference datasets in a robustness audit.

## When NOT to use

- The analysis goal is scoring simulation accuracy, functionality, scalability, or usability rather than robustness to execution failures
- A single method failed during interactive use and needs debugging rather than systematic taxonomy-based categorization
- The simulations completed without raising errors, so there is no message to classify

## Inputs

- 49 single-cell/SRT simulation methods from the simmethods R package, executed via simpipe
- 152 real reference scRNA-seq / SRT datasets from 24 platforms used as simulation inputs
- Verbatim error messages captured from each wrapped method execution
- 10-category error taxonomy defined in Additional file 2 Table S5

## Outputs

- execution_errors.csv listing per-method error counts, assigned error categories, and per-method error rates
- Category label (one of 10) assigned to each captured error occurrence

## How to apply

Install the R stack with devtools::install_github('duohongrui/simmethods') and devtools::install_github('duohongrui/simpipe') — plus dynverse/ti_slingshot for trajectory-informed simulations — and prefer the duohongrui/simpipe Docker container linked via simpipe2docker so that captured errors reflect the methods rather than local library-version drift. Run each of the 49 methods on the 152 reference datasets (24 platforms) through simpipe, wrapping every call in an error handler that stores the raised message verbatim instead of aborting the benchmark loop. Parse the collected messages and assign each occurrence to one of the 10 categories defined in Additional file 2 Table S5, dominated by 'appearance of missing (infinite) values' and 'failed parameter estimation for genes'. Aggregate category counts per method and dataset, then compute each method's error rate as the fraction of executions that raised an error, reproducing paper-scale values such as 0.7% for SCRIP-BGP-trendedBCV and ZINB-WaVE versus 88.9% for BASiCS. Write the per-method error counts, error categories, and error rates to execution_errors.csv.

## Related tools

- **simpipe** (Standard pipeline that runs the 49 simulation methods on reference datasets and lets every raised error message be captured verbatim inside the benchmark loop) — https://github.com/duohongrui/simpipe
- **simmethods** (R collection of the 49 single-cell/SRT simulation methods whose execution errors are captured and categorized) — https://github.com/duohongrui/simmethods
- **simpipe2docker** (Links the R session to the simpipe Docker container so error behavior is consistent across all 49 methods) — https://github.com/duohongrui/simpipe2docker
- **tislingshot** (Enables trajectory-informed simulation attempts so errors from trajectory scenarios are also captured and classified)

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- Every failed execution receives exactly one of the 10 categories from Additional file 2 Table S5; no message is left unclassified or double-counted
- Per-method error rates reproduce the paper's ordering: ~0.7% for SCRIP-BGP-trendedBCV and ZINB-WaVE up to 88.9% for BASiCS, with ~61% of the 49 methods affected
- Denominators equal total executions per method across the 152 reference datasets, so each rate is reconstructable from execution_errors.csv
- Captured messages are stored verbatim so category assignments can be re-audited; a benchmark run that aborts on the first error signals misapplication
- execution_errors.csv covers all 49 methods with counts, categories, and rates and no missing rows

## Limitations

- About 61% of the 49 simulation methods raise execution errors, so a large share of executions yield no simulated data for downstream accuracy or scalability evaluation
- Classification is bound to the fixed 10-category scheme of Additional file 2 Table S5; error types outside the scheme need manual extension
- Which errors are raised depends on the execution environment — the simpipe Docker image (via simpipe2docker) is recommended because local R library versions can change error behavior

## Evidence

- [abstract] execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations: "execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations"
- [methods] devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe"): "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [methods] if you want to use simpipe to simulate datasets with **trajectory information**, please install **tislingshot** first: "if you want to use simpipe to simulate datasets with **trajectory information**, please install **tislingshot** first"
- [readme] simpipe2docker links Docker containers and R environment and performs simulations for single-cell and spatial transcriptomics data.: [evidence span withheld — anchor status: external_doc]
- [full_text] errors were classified into 10 categories dominated by the appearance of missing (infinite) values and failed estimation for genes: [evidence span withheld — anchor status: not_found]
- [full_text] About 61% of the simulation methods encountered execution errors, with error proportions ranging from 0.7% (SCRIP-BGP-trendedBCV and ZINB-WaVE) to 88.9% (BASiCS): [evidence span withheld — anchor status: not_found]
