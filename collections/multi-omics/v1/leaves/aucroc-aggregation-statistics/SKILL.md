---
name: aucroc-aggregation-statistics
description: 'Use when validating that a multiplex network carries functional signal:
  you have curated gene sets, an RWR_CV cross-validation result table, and (optionally)
  rewired control networks, and need mean AUROC/AUPRC summaries to compare real vs.
  randomized rankings.'
when_to_use_negative:
- Gene sets are so small that k-fold splits would leave fewer than one gene per test
  — the article's LOO discussion (30 genes, 30 leave-one-out tests) is the appropriate
  mode for small sets instead
- No curated gold-standard gene sets exist for the species/network, since RWR_CV scoring
  depends on expert-curated sets such as MAPMAN annotations
- The analysis goal is neighborhood visualization or seed-to-node ranking (RWR_LOE
  / RWR_ShortestPaths) rather than statistical validation of network interconnectivity
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RWRtoolkit
  role: Provides RWR_CV (kfold method) for computing per-fold AUROC/AUPRC on the multiplex;
    used directly in R to run the cross-validation and aggregate results
  repo: https://github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Underlying package supplying core multiplex construction and random-walk functionality
    that RWRtoolkit extends
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: R
  role: Execution environment for RWRtoolkit, the multiplex objects, and the aggregation
    of AUROC results into CSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:44d13fbdc3d5df45dd99906c9b525b023cb7fefa7de8b34352eee4cb032e1604
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - R
  - RWRtoolkit
  - RandomWalkRestartMH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# aucroc-aggregation-statistics

## Summary

Aggregates per-gene-set, per-fold AUROC (and AUPRC) values from k-fold RWR_CV runs on the 9-layer comprehensive Arabidopsis thaliana multiplex (delta = 0.5) to quantify how well the network ranks held-out MAPMAN seed genes, benchmarked against ~1,000 node/edge-preserving rewired control multiplexes.

## When to use

Use when validating that a multiplex network carries functional signal: you have curated gene sets, an RWR_CV cross-validation result table, and (optionally) rewired control networks, and need mean AUROC/AUPRC summaries to compare real vs. randomized rankings.

## When NOT to use

- Gene sets are so small that k-fold splits would leave fewer than one gene per test — the article's LOO discussion (30 genes, 30 leave-one-out tests) is the appropriate mode for small sets instead
- No curated gold-standard gene sets exist for the species/network, since RWR_CV scoring depends on expert-curated sets such as MAPMAN annotations
- The analysis goal is neighborhood visualization or seed-to-node ranking (RWR_LOE / RWR_ShortestPaths) rather than statistical validation of network interconnectivity

## Inputs

- Preassembled 9-layer comprehensive Arabidopsis thaliana multiplex network RData object with delta = 0.5 (e.g., Comprehensive_Network_AT_d0.5_v02.RData)
- Curated MAPMAN-derived gold-standard gene set files (25 sets)
- Randomly rewired multiplex networks preserving node and edge counts per layer (~1,000 controls)

## Outputs

- Per-gene-set and average AUROC results table (rwr_cv_kfold_auroc_results.csv)
- Per-fold AUROC/AUPRC summary statistics from RWR_CV k-fold runs
- Mean AUROC for the real multiplex vs. mean AUROC for rewired controls

## How to apply

Load the preassembled 9-layer comprehensive Arabidopsis multiplex (delta = 0.5, Comprehensive_Network_AT_d0.5_v02.RData) and the 25 curated MAPMAN-derived gold-standard gene set files into R. Run RWRtoolkit's RWR_CV with method='kfold', k=5, and restart value 0.7 against the real multiplex, collecting AUROC and AUPRC summary statistics per fold. Generate ~1,000 randomly rewired multiplexes preserving node and edge counts per layer, and run the identical RWR_CV k-fold configuration (k=5, r=0.7) against each. Aggregate per-gene-set AUROC values and compute the mean across gene sets separately for the real multiplex and the rewired controls. Write per-gene-set and average AUROC results to rwr_cv_kfold_auroc_results.csv; the real network's mean AUROC should sit near 0.91 while rewired controls should average near 0.49, i.e., the random expectation of 0.50.

## Related tools

- **RWRtoolkit** (Provides RWR_CV (kfold method) for computing per-fold AUROC/AUPRC on the multiplex; used directly in R to run the cross-validation and aggregate results) — https://github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Underlying package supplying core multiplex construction and random-walk functionality that RWRtoolkit extends) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **R** (Execution environment for RWRtoolkit, the multiplex objects, and the aggregation of AUROC results into CSV)

## Examples

```
RWRtoolkit::RWR_CV(multiplex_object = comprehensive_multiplex, geneset_path = "mapman_geneset.tsv", method = "kfold", k = 5, r = 0.7, outdir = "./rwr_cv_out")
```

## Evaluation signals

- Mean AUROC across the 25 MAPMAN gene sets and 5 CV folds on the real comprehensive multiplex reproduces the reported ~0.91
- Mean AUROC across the 1,000 rewired control multiplexes averages ~0.49, matching the random expectation of 0.50
- Per-fold output contains both AUROC and AUPRC summary statistics from RWR_CV with method='kfold', k=5, restart 0.7
- rwr_cv_kfold_auroc_results.csv contains one aggregated AUROC entry per gene set plus the overall average for real and rewired networks
- Rewired multiplexes preserve the number of nodes and edges in every layer, so control AUROC distributions should be tightly centered near 0.5

## Limitations

- The RandomWalkRestartMH implementation weighted each seed equally across all layers rather than per-layer via tau, so seed-weighting parameters should not be trusted for ranking-sensitive results
- No version number was stated for RWRtoolkit or RandomWalkRestartMH, complicating reproducibility of the AUROC values
- Running RWR_CV against 1,000 rewired multiplexes is a high-throughput permutation-style workload; the command-line interface is required for parallelization on HPC systems
- Multiplex construction in RandomWalkRestartMH is computationally expensive as network size and layer count increase, so preassembled RData multiplexes should be reused

## Evidence

- [discussion] The example cross-validation experiment conducted on our 9-layer comprehensive Arabidopsis multiplex demonstrates that functionally related groups of genes have strong topological similarity: "The example cross-validation experiment conducted on our 9-layer comprehensive Arabidopsis multiplex demonstrates that functionally related groups of genes have strong topological similarity"
- [methods] we employed Random Walk with Restart Cross-Validation (RWR_CV) using a series of carefully curated gene sets: "we employed Random Walk with Restart Cross-Validation (RWR_CV) using a series of carefully curated gene sets"
- [methods] These gene sets were based on shared MAPMAN annotations [28] that are curated by experts: "These gene sets were based on shared MAPMAN annotations [28] that are curated by experts"
- [readme] RWRtoolkit enables easy use of RandomWalk with Restart on multiplex networks. These functions are an extension to the RandomWalkRestartMH R package: "RWRtoolkit enables easy use of RandomWalk with Restart on multiplex networks."
