---
name: network-rewiring-permutation-testing
description: Use when you have a curated gold-standard gene set collection (e.g.,
  25 MAPMAN-derived gene sets) and a multiplex network, and you need a statistical
  null to show RWR_CV AUROC exceeds what rewired controls give.
when_to_use_negative:
- The gene sets are not curated gold standards (e.g., arbitrary or unvalidated gene
  lists), since the AUROC comparison against the random expectation of 0.50 is only
  meaningful for functionally coherent sets
- You need strictly topology-preserving nulls (degree sequence, layer-specific edge
  motifs) beyond matching node and edge counts per layer — the rewired controls here
  only preserve the number of nodes and edges in every layer
- You want leave-one-out (LOO) RWR_CV instead of k-fold; this skill's reproduction
  targets (AUROC ~0.91 vs ~0.49) are specific to the k-fold (k=5) configuration
edam_operation: http://edamontology.org/operation_0294
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: RWRtoolkit
  role: Runs RWR_CV (method='kfold', k=5, restart 0.7) on the real and rewired multiplexes
    and aggregates AUROC results; CLI supports HPC parallelization for the 1,000-permutation
    testing
  repo: https://github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Provides the core multiplex construction and random-walk-with-restart functionality
    underlying RWRtoolkit's cross-validation
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: R
  role: Environment in which the multiplex, gene sets, rewired controls, and RWR_CV
    analyses are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:cecc26fa6dec672b857e1180ca4ae0eec199785a6e323d243be602473b8db311
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - R
  - RWRtoolkit
  - RandomWalkRestartMH
  edam_operation: http://edamontology.org/operation_0294
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# network-rewiring-permutation-testing

## Summary

Permutation-testing skill for judging whether RWR-based gene-function predictions from a multiplex network (e.g., the 9-layer comprehensive Arabidopsis thaliana multiplex, delta = 0.5) are better than chance, by comparing RWR_CV AUROC on the real multiplex against AUROC on many randomly rewired null multiplexes that preserve node and edge counts per layer.

## When to use

Use when you have a curated gold-standard gene set collection (e.g., 25 MAPMAN-derived gene sets) and a multiplex network, and you need a statistical null to show RWR_CV AUROC exceeds what rewired controls give.

## When NOT to use

- The gene sets are not curated gold standards (e.g., arbitrary or unvalidated gene lists), since the AUROC comparison against the random expectation of 0.50 is only meaningful for functionally coherent sets
- You need strictly topology-preserving nulls (degree sequence, layer-specific edge motifs) beyond matching node and edge counts per layer — the rewired controls here only preserve the number of nodes and edges in every layer
- You want leave-one-out (LOO) RWR_CV instead of k-fold; this skill's reproduction targets (AUROC ~0.91 vs ~0.49) are specific to the k-fold (k=5) configuration

## Inputs

- Preassembled 9-layer comprehensive Arabidopsis thaliana multiplex network RData object (Comprehensive_Network_AT_d0.5_v02.RData, delta = 0.5, unweighted edges)
- MAPMAN-derived gold-standard gene set files (25 curated gene sets)
- Rewiring configuration: 1,000 permutations preserving node and edge counts per layer
- RWR_CV parameters: method='kfold', k=5, restart probability 0.7

## Outputs

- Per-gene-set AUROC/AUPRC summary statistics per CV fold for the real multiplex
- Per-gene-set AUROC distributions across the 1,000 rewired control multiplexes
- rwr_cv_kfold_auroc_results.csv with per-gene-set and average AUROC values for real vs. rewired networks

## How to apply

Load the preassembled 9-layer comprehensive Arabidopsis multiplex (Comprehensive_Network_AT_d0.5_v02.RData, delta = 0.5) and the MAPMAN-derived gene set files in R via RWRtoolkit. Run RWR_CV with method='kfold', k=5, and restart value 0.7 on each gene set against the real multiplex, collecting AUROC/AUPRC per fold. Generate 1,000 randomly rewired multiplex networks that preserve the number of nodes and edges in every layer, then repeat the identical RWR_CV k-fold analysis on each gene set against each rewired multiplex. Aggregate per-gene-set AUROC values and compute the mean AUROC across gene sets for real vs. rewired multiplexes; the CLI interface is designed for parallelization on HPC systems for exactly this kind of permutation testing. Write per-gene-set and average AUROC results to rwr_cv_kfold_auroc_results.csv and compare against the reported reproduction targets (real ~0.91, rewired ~0.49).

## Related tools

- **RWRtoolkit** (Runs RWR_CV (method='kfold', k=5, restart 0.7) on the real and rewired multiplexes and aggregates AUROC results; CLI supports HPC parallelization for the 1,000-permutation testing) — https://github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Provides the core multiplex construction and random-walk-with-restart functionality underlying RWRtoolkit's cross-validation) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **R** (Environment in which the multiplex, gene sets, rewired controls, and RWR_CV analyses are executed)

## Examples

```
RWRtoolkit::RWR_CV(data = multiplex, geneset = mapman_geneset, method = 'kfold', k = 5, restart = 0.7, outdir = 'outdir')
```

## Evaluation signals

- Average AUROC across the 25 MAPMAN gene sets and 5 CV folds on the real comprehensive multiplex reproduces the reported value of approximately 0.91
- Mean AUROC across the 1,000 rewired control multiplexes averages approximately 0.49, near the random expectation of 0.50
- Each of the 1,000 rewired multiplexes preserves the number of nodes and edges in every one of the 9 layers
- rwr_cv_kfold_auroc_results.csv contains both per-gene-set and average AUROC values for the real multiplex and the rewired controls
- Per-fold AUROC/AUPRC summary statistics are collected for every gene set and every CV fold before aggregation

## Limitations

- Rewired controls preserve only node and edge counts per layer, not finer topological structure, so the null may not capture all network-structure effects
- The reproduction targets are tied to specific parameters (k-fold with k=5, restart value 0.7, delta = 0.5 comprehensive multiplex); other settings have no reported expected AUROC
- No version number is stated for RWRtoolkit or RandomWalkRestartMH, and the underlying RandomWalkRestartMH tau weighting behavior has been flagged as an implementation concern
- The 1,000-permutation workload assumes access to a high-performance cluster system for parallelized execution

## Evidence

- [abstract] The command-line interface is designed for parallelization on high-performance cluster systems, which enables high-throughput analysis such as permutation testing.: "The command-line interface is designed for parallelization on high-performance cluster systems, which enables high-throughput analysis such as permutation testing."
- [methods] we employed Random Walk with Restart Cross-Validation (RWR_CV) using a series of carefully curated gene sets: "we employed Random Walk with Restart Cross-Validation (RWR_CV) using a series of carefully curated gene sets"
- [discussion] The example cross-validation experiment conducted on our 9-layer comprehensive Arabidopsis multiplex demonstrates that functionally related groups of genes have strong topological similarity: "The example cross-validation experiment conducted on our 9-layer comprehensive Arabidopsis multiplex demonstrates that functionally related groups of genes have strong topological similarity"
- [readme] Contains the following network layers with unweighted edges and delta value = 0.5:: "Contains the following network layers with unweighted edges and delta value = 0.5:"
