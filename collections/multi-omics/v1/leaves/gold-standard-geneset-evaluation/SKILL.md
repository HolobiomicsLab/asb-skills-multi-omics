---
name: gold-standard-geneset-evaluation
description: Use when you have a preassembled multiplex network and curated gene sets
  and need to quantify whether functionally related genes rank highly under random
  walk with restart, including against rewired-control baselines.
when_to_use_negative:
- The gene sets are not curated/expert-annotated gold standards (e.g., arbitrary or
  unvalidated groupings) — AUROC against such sets does not establish functional predictiveness.
- The goal is network construction or layer statistics rather than validation — use
  RWR_make_multiplex and RWR_netstats instead.
- The network is a single monoplex with no functional gene sets available for cross-validation;
  RWR_CV requires curated sets to leave genes out from.
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0605
tools:
- name: RWRtoolkit
  role: Provides RWR_CV for k-fold/LOO cross-validation of gene sets on the multiplex
    and command-line scripts parallelizable on HPC systems for permutation-style testing
    against rewired controls
  repo: https://github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Core underlying functionality for constructing multiplex networks and implementing
    random walk with restart that RWR_CV builds on
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: R
  role: Runtime environment (R >= 4.1.0 required; installed via devtools::install_github)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:cf44c9a4cc31df4e5e166fd2923ebb6a952f4828727243c164fa78de6b8839e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - R
  - RWRtoolkit
  - RandomWalkRestartMH
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# gold-standard-geneset-evaluation

## Summary

Validates the functional predictiveness of a multiplex network by running RWR_CV (k-fold or LOO) on curated gold-standard gene sets (e.g., 25 MAPMAN-derived sets) against the real multiplex and against randomly rewired negative controls, using AUROC as the headline metric.

## When to use

Use when you have a preassembled multiplex network and curated gene sets and need to quantify whether functionally related genes rank highly under random walk with restart, including against rewired-control baselines.

## When NOT to use

- The gene sets are not curated/expert-annotated gold standards (e.g., arbitrary or unvalidated groupings) — AUROC against such sets does not establish functional predictiveness.
- The goal is network construction or layer statistics rather than validation — use RWR_make_multiplex and RWR_netstats instead.
- The network is a single monoplex with no functional gene sets available for cross-validation; RWR_CV requires curated sets to leave genes out from.

## Inputs

- Preassembled multiplex network RData object (e.g., Comprehensive_Network_AT_d0.5_v02.RData, 9 layers, delta = 0.5)
- Curated gold-standard gene set files based on shared MAPMAN annotations (expert-curated)
- CV parameters: method='kfold', k=5, restart probability 0.7
- Set of ~1,000 randomly rewired multiplex networks preserving per-layer node and edge counts

## Outputs

- Per-gene-set, per-fold AUROC and AUPRC summary statistics
- Mean AUROC across gene sets for the real multiplex and for rewired controls
- Results table written to rwr_cv_kfold_auroc_results.csv

## How to apply

Load the preassembled multiplex (e.g., the 9-layer comprehensive Arabidopsis thaliana multiplex, delta = 0.5, from RWRtoolkit-data) and the curated gold-standard gene set files into R. For each gene set, run RWRtoolkit's RWR_CV with method='kfold', k=5, and restart probability 0.7 against the real multiplex, collecting per-fold AUROC/AUPRC summary statistics; leave-one-out (LOO) is the alternative CV mode demonstrated in the paper. Generate ~1,000 randomly rewired multiplexes preserving node and edge counts per layer, and rerun the identical RWR_CV parameters on each rewired control to establish the random expectation (~0.49 AUROC vs. the theoretical 0.50). Aggregate per-gene-set AUROC values and compute mean AUROC across gene sets for real and rewired networks; the real network should far exceed the controls. Write per-gene-set and average results to a CSV (e.g., rwr_cv_kfold_auroc_results.csv).

## Related tools

- **RWRtoolkit** (Provides RWR_CV for k-fold/LOO cross-validation of gene sets on the multiplex and command-line scripts parallelizable on HPC systems for permutation-style testing against rewired controls) — https://github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Core underlying functionality for constructing multiplex networks and implementing random walk with restart that RWR_CV builds on) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **R** (Runtime environment (R >= 4.1.0 required; installed via devtools::install_github))

## Examples

```
Rscript run_cv.R --data Comprehensive_Network_AT_d0.5_v02.RData --geneset MAPMAN_geneset.txt --method kfold --k 5 --r 0.7 -o ./outdir
```

## Evaluation signals

- Average AUROC across all gene sets and 5-fold CV folds on the real comprehensive multiplex should reproduce the reported ~0.91.
- Mean AUROC on the 1,000 randomly rewired multiplexes should be ~0.49, near the random expectation of 0.50 — a large real-vs-control gap confirms the multiplex is functionally predictive.
- Per-gene-set AUROC/AUPRC rows must be present for all 25 MAPMAN-derived sets and all folds before aggregation.
- Rewired controls must preserve the number of nodes and edges in every layer; verify layer-wise counts match the original multiplex.

## Limitations

- Results depend on the choice of gold-standard sets; the paper's validation uses MAPMAN annotations curated by experts, so sets from other sources may yield different AUROC levels.
- RWR_CV in the RandomWalkRestartMH implementation does not correctly weight seeds per layer via tau (tau weights each seed equally across layers), which was found to have minimal effect on output rankings.
- Command-line permutation testing (e.g., 1,000 rewired networks) is designed for parallelization on high-performance cluster systems; running serially can be very slow.

## Evidence

- [discussion] k-fold RWR_CV on 25 MAPMAN sets vs rewired controls: "The example cross-validation experiment conducted on our 9-layer comprehensive Arabidopsis multiplex demonstrates that functionally related groups of genes have strong topological similarity"
- [methods] RWR_CV using curated gene sets: "we employed Random Walk with Restart Cross-Validation (RWR_CV) using a series of carefully curated gene sets"
- [methods] MAPMAN annotations curated by experts: "These gene sets were based on shared MAPMAN annotations [28] that are curated by experts"
- [abstract] CLI supports permutation testing on HPC: "The command-line interface is designed for parallelization on high-performance cluster systems, which enables high-throughput analysis such as permutation testing."
- [readme] Multiplex RData with delta 0.5 and named layers: "Contains the following network layers with unweighted edges and delta value = 0.5"
