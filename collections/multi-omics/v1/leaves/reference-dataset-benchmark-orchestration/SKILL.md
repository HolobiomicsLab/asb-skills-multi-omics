---
name: reference-dataset-benchmark-orchestration
description: Use when many (tens of) simulation or analysis methods must be run uniformly
  over a panel of real reference datasets in R, with per-execution error capture and
  per-method success/error-rate aggregation.
when_to_use_negative:
- Only a single simulation method on a single dataset is needed — the orchestration
  overhead (Docker, error wrapping, aggregation) is unnecessary
- The methods of interest are not among the 49 collected in simmethods and have no
  simpipe wrapper
- The goal is evaluating a downstream analysis tool rather than benchmarking data-simulation
  methods themselves
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_2229
tools:
- name: simpipe
  role: Standard pipeline that executes simulation methods on reference datasets and
    evaluates simulated data; provides the benchmark loop and Docker image
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: R package bundling the 49 simulation methods that are orchestrated by simpipe
  repo: https://github.com/duohongrui/simmethods
- name: simpipe2docker
  role: Links the R environment to the simpipe Docker container for a consistent execution
    environment
  repo: https://github.com/duohongrui/simpipe2docker
- name: tislingshot
  role: Required before simpipe can simulate datasets with trajectory information
    (dynverse/ti_slingshot)
  repo: https://github.com/dynverse/ti_slingshot/package/
- name: simshiny
  role: Shiny companion for exploring simulation results
  repo: https://github.com/duohongrui/simshiny
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:9eff0e58f3e455b8b4b5bdb98f4d0079a216de2257a188b1556591d16dd72977
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
  - simshiny
  - tislingshot
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# reference-dataset-benchmark-orchestration

## Summary

Orchestrates a benchmark in which 49 single-cell/spatial transcriptomics simulation methods from simmethods are executed through the simpipe standard pipeline (optionally inside the simpipe Docker container) on real reference datasets, capturing per-execution errors and computing per-method performance rates.

## When to use

Use when many (tens of) simulation or analysis methods must be run uniformly over a panel of real reference datasets in R, with per-execution error capture and per-method success/error-rate aggregation.

## When NOT to use

- Only a single simulation method on a single dataset is needed — the orchestration overhead (Docker, error wrapping, aggregation) is unnecessary
- The methods of interest are not among the 49 collected in simmethods and have no simpipe wrapper
- The goal is evaluating a downstream analysis tool rather than benchmarking data-simulation methods themselves

## Inputs

- Real reference scRNA-seq and/or SRT datasets (152 datasets from 24 platforms) in R-compatible formats accepted by simpipe
- List of simulation method IDs from the simmethods collection (49 methods, e.g. BASiCS, BEARscc, dyngen, SRTsim, scDesign3, ZINB-WaVE)
- R session with devtools, simmethods, simpipe, and optionally tislingshot and simpipe2docker

## Outputs

- Per-method error counts, error category assignments, and per-method error rates (e.g. execution_errors.csv)
- Simulated datasets per method/dataset combination with ground truth for downstream accuracy, functionality, scalability, and usability evaluation
- Verbatim captured error messages per execution for categorization

## How to apply

Install simmethods and simpipe from GitHub via devtools (and tislingshot if trajectory-informed simulations are needed), optionally pull the duohongrui/simpipe Docker image and link it to R with simpipe2docker for a consistent execution environment. Run each of the 49 collected simulation methods (e.g. BASiCS, SRTsim, scDesign3, ZINB-WaVE, scDesign2, Lun, SPARSim, scDesign3-tree, Phenopath, Simple, MFA) through simpipe on the 152 reference datasets from 24 platforms, wrapping every call so raised error messages are captured verbatim instead of aborting the loop. Classify captured errors into categories such as failed parameter estimation for genes and appearance of missing or infinite values, then aggregate per-method execution counts and compute per-method error rates (ranging from 0.7% to 88.9% in the study). The rationale is that a standard pipeline plus containerization removes environment variability, so differences in accuracy, functionality, scalability, and usability are attributable to the methods rather than the runtime.

## Related tools

- **simpipe** (Standard pipeline that executes simulation methods on reference datasets and evaluates simulated data; provides the benchmark loop and Docker image) — https://github.com/duohongrui/simpipe
- **simmethods** (R package bundling the 49 simulation methods that are orchestrated by simpipe) — https://github.com/duohongrui/simmethods
- **simpipe2docker** (Links the R environment to the simpipe Docker container for a consistent execution environment) — https://github.com/duohongrui/simpipe2docker
- **tislingshot** (Required before simpipe can simulate datasets with trajectory information (dynverse/ti_slingshot)) — https://github.com/dynverse/ti_slingshot/package/
- **simshiny** (Shiny companion for exploring simulation results) — https://github.com/duohongrui/simshiny

## Examples

```
devtools::install_github("duohongrui/simmethods"); devtools::install_github("duohongrui/simpipe")
```

## Evaluation signals

- Every method execution either returns a simulated dataset or a captured verbatim error message — no silent failures or aborted runs
- Each error message maps to one of the 10 predefined error categories (e.g. failed parameter estimation for genes, appearance of missing/infinite values)
- Per-method error rates fall within plausible ranges and match reported extremes (e.g. 0.7% for SCRIP-BGP-trendedBCV and ZINB-WaVE up to 88.9% for BASiCS)
- All 49 simmethods IDs are represented in the output table with counts summed over the 152 reference datasets from 24 platforms
- Docker-backed runs (simpipe2docker) reproduce R-native runs without environment-dependent discrepancy in error patterns

## Limitations

- About 61% of the 49 simulation methods encountered execution errors, so many method–dataset combinations yield no simulated data
- Trajectory-informed simulations additionally require installing tislingshot first
- Some high-scalability methods (Phenopath, Lun, Simple, MFA) cannot generate realistic simulated data, so orchestration success does not imply simulation fidelity

## Evidence

- [methods] installation of simmethods: "devtools::install_github("duohongrui/simmethods")
devtools::install_github("duohongrui/simpipe")"
- [methods] docker linking: "use [**simpipe2docker**](https://github.com/duohongrui/simpipe2docker) package for linking R environmrnt and the Docker container"
- [methods] tislingshot prerequisite: "if you want to use simpipe to simulate datasets with **trajectory information**, please install **tislingshot** first"
- [abstract] benchmark scope: "We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability using 152 reference datasets derived from 24"
- [abstract] error causes: "execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations"
- [readme] simpipe2docker install: "devtools::install_github("duohongrui/simpipe2docker")"
