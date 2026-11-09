---
name: dockerized-r-benchmark-execution
description: Use when benchmarking the 49 simmethods simulation methods for scRNA-seq/SRT
  data and you need identical dependency environments across runs, or when reproducing
  per-method execution-error rates.
when_to_use_negative:
- Docker Desktop cannot be installed or the daemon fails the test_docker_installation
  checks (pull, run, or volume mounting) — install simmethods and simpipe directly
  in R instead
- The goal is scoring the accuracy/fidelity of the simulated data itself rather than
  reproducing execution errors and environment-consistent per-method error rates
- Only a single, already-tested simulation method is needed for one-off data generation,
  where container overhead adds no benchmarking value
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: simpipe2docker
  role: Links the R environment and the Docker container and verifies the Docker installation
    before benchmark execution
  repo: https://github.com/duohongrui/simpipe2docker
- name: simpipe
  role: Standard pipeline whose parameter-estimation, simulation, and evaluation calls
    for the 49 methods are executed in the containerized environment
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Supplies the collection of 49 simulation methods being benchmarked inside
    the container
  repo: https://github.com/duohongrui/simmethods
- name: Docker
  role: Container runtime hosting the pulled simpipe image; installation, daemon,
    version, pull/run, and volume-mount checks precede execution
  repo: https://www.docker.com/
- name: tislingshot
  role: Installed first so simpipe can simulate datasets with trajectory information
    during the benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:a94c56b169deabc59c27f57355409d77d3a8130a47c60d27b588611648d4093e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simpipe2docker
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# dockerized-r-benchmark-execution

## Summary

Executes the simmethods/simpipe simulation benchmark for scRNA-seq and spatially resolved transcriptomics inside the duohongrui/simpipe Docker image, linked to R via simpipe2docker, so all 49 simulation methods run in one consistent dependency environment with errors captured verbatim.

## When to use

Use when benchmarking the 49 simmethods simulation methods for scRNA-seq/SRT data and you need identical dependency environments across runs, or when reproducing per-method execution-error rates.

## When NOT to use

- Docker Desktop cannot be installed or the daemon fails the test_docker_installation checks (pull, run, or volume mounting) — install simmethods and simpipe directly in R instead
- The goal is scoring the accuracy/fidelity of the simulated data itself rather than reproducing execution errors and environment-consistent per-method error rates
- Only a single, already-tested simulation method is needed for one-off data generation, where container overhead adds no benchmarking value

## Inputs

- Docker Desktop installation with a running daemon (verified via simpipe2docker::test_docker_installation(detailed = TRUE))
- duohongrui/simpipe:latest image pulled from Docker Hub
- R session with devtools, simmethods, simpipe, and simpipe2docker installed from GitHub
- Real scRNA-seq/SRT reference datasets for parameter estimation (the benchmark used 152 reference datasets from 24 platforms)
- Calls to the 49 simmethods simulation methods (e.g., SRTsim, scDesign3, ZINB-WaVE, scDesign2, BASiCS, SPARSim)

## Outputs

- Containerized, environment-consistent execution of parameter estimation, simulation, and evaluation across all 49 methods
- Verbatim captured error messages for every failed method execution
- execution_errors.csv with per-method error counts, 10-category error classifications, and per-method error rates

## How to apply

Install Docker Desktop, verify it with simpipe2docker::test_docker_installation(detailed = TRUE), then pull the image via 'docker pull duohongrui/simpipe:latest' or babelwhale::pull_container. In R, install devtools packages duohongrui/simmethods, duohongrui/simpipe, and duohongrui/simpipe2docker, adding dynverse/ti_slingshot if trajectory-informed simulations will be attempted. Use simpipe2docker to link the R environment and the Docker container, then run parameter estimation, simulation, and evaluation for each of the 49 methods on the reference datasets inside the container. Wrap every method call so raised error messages are captured verbatim rather than aborting the benchmark loop, and assign each error to one of the 10 categories in Additional file 2 Table S5, which are dominated by failed parameter estimation for genes and appearance of missing (infinite) values. Aggregate categorized errors per method and dataset, compute per-method error rates as the fraction of executions that raised an error, and write execution_errors.csv; expect roughly 61% of methods to error, ranging from 0.7% (SCRIP-BGP-trendedBCV, ZINB-WaVE) to 88.9% (BASiCS).

## Related tools

- **simpipe2docker** (Links the R environment and the Docker container and verifies the Docker installation before benchmark execution) — https://github.com/duohongrui/simpipe2docker
- **simpipe** (Standard pipeline whose parameter-estimation, simulation, and evaluation calls for the 49 methods are executed in the containerized environment) — https://github.com/duohongrui/simpipe
- **simmethods** (Supplies the collection of 49 simulation methods being benchmarked inside the container) — https://github.com/duohongrui/simmethods
- **Docker** (Container runtime hosting the pulled simpipe image; installation, daemon, version, pull/run, and volume-mount checks precede execution) — https://www.docker.com/
- **tislingshot** (Installed first so simpipe can simulate datasets with trajectory information during the benchmark)

## Examples

```
simpipe2docker::test_docker_installation(detailed = TRUE); babelwhale::pull_container(container_id = "duohongrui/simpipe:latest")
```

## Evaluation signals

- simpipe2docker::test_docker_installation(detailed = TRUE) returns TRUE with checks for a running daemon, version >1.0, image pull/run capability, and temporary volume mounting
- All 49 method executions run in the same pulled duohongrui/simpipe:latest image with no environment-induced divergence between runs
- Every failed execution yields a verbatim captured error message with no silent aborts of the benchmark loop
- Categorized errors map onto the 10 categories of Additional file 2 Table S5, dominated by failed parameter estimation for genes and appearance of missing (infinite) values
- Per-method error rates fall in the reported 0.7% (SCRIP-BGP-trendedBCV, ZINB-WaVE) to 88.9% (BASiCS) range when rerun on the same reference datasets

## Limitations

- Containerized execution requires Docker Desktop and a running daemon; failures in pull, run, or volume-mount checks block this route entirely
- Dockerization is optional — the article presents simpipe2docker linking as an alternative, and users without Docker must rely on direct local installs of simmethods/simpipe
- Capturing errors does not resolve them: about 61% of the 49 methods hit execution errors, mainly failed parameter estimations and missing/infinite values, with BASiCS failing up to 88.9% of executions

## Evidence

- [readme] simpipe2docker links Docker containers and R environment and performs simulations for single-cell and spatial transcriptomics data.: [evidence span withheld — anchor status: external_doc]
- [methods] use [**simpipe2docker**](https://github.com/duohongrui/simpipe2docker) package for linking R environmrnt and the Docker container: "use [**simpipe2docker**](https://github.com/duohongrui/simpipe2docker) package for linking R environmrnt and the Docker container"
- [readme] babelwhale::pull_container(container_id = "duohongrui/simpipe:latest"): [evidence span withheld — anchor status: external_doc]
- [abstract] execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations: "execution errors are mainly caused by failed parameter estimations and appearance of missing or infinite values in calculations"
- [methods] if you want to use simpipe to simulate datasets with **trajectory information**, please install **tislingshot** first: "if you want to use simpipe to simulate datasets with **trajectory information**, please install **tislingshot** first"
