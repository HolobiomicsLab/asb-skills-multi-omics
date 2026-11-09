---
name: per-method-error-rate-quantification
description: Use when benchmarking scRNA-seq or SRT simulation methods and you need
  to know which methods raise execution errors, how often, and in which error categories
  (e.g. failed parameter estimation, missing/infinite values).
when_to_use_negative:
- You only need simulation accuracy/scalability metrics and no method has failed —
  error categorization adds nothing without captured error messages.
- You are evaluating a single method interactively rather than a benchmark loop across
  many methods/datasets, where per-method rate aggregation is the point.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_3308
tools:
- name: simmethods
  role: Provides the 49 simulation methods whose execution errors are captured and
    categorized
  repo: https://github.com/duohongrui/simmethods
- name: simpipe
  role: Standard pipeline that runs each simulation method on reference datasets while
    capturing errors
  repo: https://github.com/duohongrui/simpipe
- name: simpipe2docker
  role: Links the R environment to the simpipe Docker container for a consistent execution
    environment across methods
  repo: https://github.com/duohongrui/simpipe2docker
- name: tislingshot
  role: Enables trajectory-informed simulation attempts (via ti_slingshot) whose errors
    are also captured
  repo: https://github.com/dynverse/ti_slingshot
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:ee1e0785abc236de049573f81913500536e6f9fa734a4e5addb985e3e4335f82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simmethods
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# per-method-error-rate-quantification

## Summary

Systematically captures execution errors raised by each of the 49 single-cell/spatial transcriptomics simulation methods run through simpipe, categorizes the error messages, and computes per-method error rates to reveal which simulators fail and why.

## When to use

Use when benchmarking scRNA-seq or SRT simulation methods and you need to know which methods raise execution errors, how often, and in which error categories (e.g. failed parameter estimation, missing/infinite values).

## When NOT to use

- You only need simulation accuracy/scalability metrics and no method has failed — error categorization adds nothing without captured error messages.
- You are evaluating a single method interactively rather than a benchmark loop across many methods/datasets, where per-method rate aggregation is the point.

## Inputs

- 49 simulation methods available in the simmethods R package
- Reference scRNA-seq / SRT datasets (152 real reference datasets from 24 platforms in the source study)
- simpipe execution environment (local R or simpipe Docker image)
- Verbatim error messages captured from each method call

## Outputs

- execution_errors.csv with per-method error counts, assigned error categories, and per-method error rates
- Categorization of errors into the 10 categories of Additional file 2 Table S5

## How to apply

Install simmethods and simpipe from GitHub via devtools (plus tislingshot for trajectory-informed simulations), and optionally run all method calls inside the simpipe Docker image linked through simpipe2docker for a consistent environment. Wrap each of the 49 method calls (e.g. SRTsim, scDesign3, ZINB-WaVE, scDesign2, Lun, SPARSim, BASiCS) so that any raised error message is captured verbatim instead of aborting the benchmark loop. Parse the collected messages and assign each occurrence to one of the 10 error categories defined in Additional file 2 Table S5, which are dominated by the appearance of missing (infinite) values and failed estimation for genes. Finally, aggregate categorized errors per method and dataset and compute the per-method error rate as the fraction of executions that raised an error; in the source study about 61% of methods hit errors, ranging from 0.7% (SCRIP-BGP-trendedBCV, ZINB-WaVE) to 88.9% (BASiCS).

## Related tools

- **simmethods** (Provides the 49 simulation methods whose execution errors are captured and categorized) — https://github.com/duohongrui/simmethods
- **simpipe** (Standard pipeline that runs each simulation method on reference datasets while capturing errors) — https://github.com/duohongrui/simpipe
- **simpipe2docker** (Links the R environment to the simpipe Docker container for a consistent execution environment across methods) — https://github.com/duohongrui/simpipe2docker
- **tislingshot** (Enables trajectory-informed simulation attempts (via ti_slingshot) whose errors are also captured) — https://github.com/dynverse/ti_slingshot

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- Every method execution outcome is recorded: either simulated data or a verbatim captured error message — no silent aborts of the benchmark loop.
- All captured error messages map to exactly one of the 10 categories in Additional file 2 Table S5.
- Per-method error rate equals (number of erroring executions) / (total executions) for that method and lies in [0,1].
- Expected range sanity check: in the source study, error proportions ranged from 0.7% (SCRIP-BGP-trendedBCV, ZINB-WaVE) to 88.9% (BASiCS), with ~61% of methods erroring.
- execution_errors.csv contains method name, dataset, error category, count, and rate columns without missing method entries.

## Limitations

- Execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations, so rates reflect method robustness on the chosen reference data, not simulation fidelity.
- Error rates depend on the reference datasets and environment; results are not transferable without running the same 49 methods through simpipe on your own data.
- Methods with high scalability but frequent errors (e.g. BASiCS at 88.9%) cannot be assumed reliable despite speed.

## Evidence

- [results] about 61% of simulation methods encountered execution errors: [evidence span withheld — anchor status: not_found]
- [abstract] error categories dominated by missing values and failed gene estimation: "execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations"
- [methods] install methods via devtools: "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [methods] simpipe2docker links R and Docker: "use [**simpipe2docker**](https://github.com/duohongrui/simpipe2docker) package for linking R environmrnt and the Docker container"
- [readme] simpipe goal statement: "The goal of simpipe is to establish a standard pipeline to estimating parameters from real datasets, simulating single-cell RNA-seq datasets and evaluating the simulated data"
