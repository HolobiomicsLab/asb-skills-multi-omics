---
name: multiplex-network-cross-validation
description: Use when a constructed multiplex network needs statistical evaluation
  of whether curated gene sets (e.g., MAPMAN- or GO-derived) show topological similarity
  sufficient to recover held-out genes via random walk with restart.
when_to_use_negative:
- The goal is scoring/ranking genes relative to a fixed seed set rather than statistically
  validating gene set interconnectivity — use RWR_LOE instead of RWR_CV
- The gene set is small and LOO-style validation is intended (e.g., 30 genes where
  one gene is left out each time) rather than k-fold evaluation
- No curated gold-standard gene sets exist for the species/network, so AUROC against
  held-out genes cannot be computed
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0605
tools:
- name: RWRtoolkit
  role: Provides RWR_CV for k-fold/LOO cross-validation of gene sets on multiplex
    networks, plus multiplex construction and command-line scripts for parallelized
    permutation testing
  repo: https://github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Underlying R package providing core multiplex network construction and random
    walk with restart functionality that RWR_CV builds upon
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: R
  role: Runtime environment for executing RWRtoolkit functions (R >= 4.1.0 per README)
  repo: https://github.com/dkainer/RWRtoolkit
- name: RWRtoolkit-data
  role: Supplies preassembled Arabidopsis multiplex network RData objects (e.g., Comprehensive_Network_AT_d0.5_v02.RData)
    used as validation targets
  repo: https://github.com/dkainer/RWRtoolkit-data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:40b50254d529f3b7c2f79230916496934675889ba1b968600d7ef45306bacf68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - R
  - RWRtoolkit
  - RWRtoolkit-data
  - RandomWalkRestartMH
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# multiplex-network-cross-validation

## Summary

Validates the functional predictiveness of a multiplex biological network by running RWR cross-validation (RWR_CV, k-fold or LOO) with curated gene sets, comparing AUROC/AUPRC against randomly rewired control multiplexes to confirm that functionally related genes rank highly.

## When to use

Use when a constructed multiplex network needs statistical evaluation of whether curated gene sets (e.g., MAPMAN- or GO-derived) show topological similarity sufficient to recover held-out genes via random walk with restart.

## When NOT to use

- The goal is scoring/ranking genes relative to a fixed seed set rather than statistically validating gene set interconnectivity — use RWR_LOE instead of RWR_CV
- The gene set is small and LOO-style validation is intended (e.g., 30 genes where one gene is left out each time) rather than k-fold evaluation
- No curated gold-standard gene sets exist for the species/network, so AUROC against held-out genes cannot be computed

## Inputs

- Preassembled multiplex network RData object (e.g., Comprehensive_Network_AT_d0.5_v02.RData, 9 layers, delta = 0.5)
- Curated gold-standard gene set files (e.g., MAPMAN-derived gene sets for Arabidopsis)
- RWR_CV parameters: k-fold method with k=5 and restart probability 0.7 (or LOO for small gene sets)
- Randomly rewired control multiplexes preserving per-layer node and edge counts

## Outputs

- Per-gene-set, per-fold AUROC/AUPRC summary statistics from RWR_CV
- Mean AUROC across gene sets for the real multiplex and for rewired controls
- Results table (e.g., rwr_cv_kfold_auroc_results.csv)

## How to apply

Load a preassembled multiplex (e.g., the 9-layer comprehensive Arabidopsis thaliana multiplex with delta = 0.5 from RWRtoolkit-data) plus curated gold-standard gene set files. Run RWRtoolkit's RWR_CV with method='kfold', k=5 (or LOO for small sets) and restart value 0.7, collecting per-fold AUROC/AUPRC summary statistics for each gene set. Generate ~1,000 randomly rewired multiplex networks preserving node and edge counts per layer, and rerun the identical RWR_CV configuration on each rewired control. Aggregate per-gene-set AUROC values and compute mean AUROC for the real vs. rewired multiplexes; a real-multiplex mean near 0.91 versus ~0.49 for rewired controls (random expectation ~0.50) confirms functional predictiveness. Write per-gene-set and average results to a CSV such as rwr_cv_kfold_auroc_results.csv.

## Related tools

- **RWRtoolkit** (Provides RWR_CV for k-fold/LOO cross-validation of gene sets on multiplex networks, plus multiplex construction and command-line scripts for parallelized permutation testing) — https://github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Underlying R package providing core multiplex network construction and random walk with restart functionality that RWR_CV builds upon) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **R** (Runtime environment for executing RWRtoolkit functions (R >= 4.1.0 per README)) — https://github.com/dkainer/RWRtoolkit
- **RWRtoolkit-data** (Supplies preassembled Arabidopsis multiplex network RData objects (e.g., Comprehensive_Network_AT_d0.5_v02.RData) used as validation targets) — https://github.com/dkainer/RWRtoolkit-data

## Examples

```
Rscript run_cv.R --multiplex_x Comprehensive_Network_AT_d0.5_v02.RData --geneset_path genesets/ --method kfold --k 5 --rwr_restart 0.7 -o rwr_cv_kfold_auroc_results.csv
```

## Evaluation signals

- Average AUROC across the 25 MAPMAN-derived gene sets and 5 CV folds on the real multiplex should reproduce approximately 0.91
- Rewired control multiplexes should average approximately 0.49 AUROC (near the random expectation of 0.50)
- Per-gene-set AUROC/AUPRC statistics are collected per fold and written to rwr_cv_kfold_auroc_results.csv
- Rewired networks must preserve the number of nodes and edges in every layer of the original multiplex
- The 9-layer comprehensive Arabidopsis multiplex (delta = 0.5) is the loaded target; 'functionally related groups of genes have strong topological similarity' is the expected qualitative outcome

## Limitations

- RandomWalkRestartMH incorrectly weights each seed equally across all layers (tau had minimal effect on output rankings), so layer-weighting behavior in the base implementation is limited
- Multiplex construction in the original RandomWalkRestartMH was unoptimized and computationally exponential in network size/layer count; RWRtoolkit parallelizes it, but very large networks remain costly
- No version number was stated for RWRtoolkit or RandomWalkRestartMH in the article, complicating exact reproducibility
- Validation depends on curated gene sets (e.g., MAPMAN annotations curated by experts); results are only as good as the gold-standard sets

## Evidence

- [discussion] The example cross-validation experiment conducted on our 9-layer comprehensive Arabidopsis multiplex demonstrates that functionally related groups of genes have strong topological similarity: "The example cross-validation experiment conducted on our 9-layer comprehensive Arabidopsis multiplex demonstrates that functionally related groups of genes have strong topological similarity"
- [methods] we employed Random Walk with Restart Cross-Validation (RWR_CV) using a series of carefully curated gene sets: "we employed Random Walk with Restart Cross-Validation (RWR_CV) using a series of carefully curated gene sets"
- [methods] These gene sets were based on shared MAPMAN annotations [28] that are curated by experts: "These gene sets were based on shared MAPMAN annotations [28] that are curated by experts"
- [readme] Contains the following network layers with unweighted edges and delta value = 0.5:: "Contains the following network layers with unweighted edges and delta value = 0.5:"
- [discussion] Given that there are 30 genes within the set, 30 individual tests of RWR were run, leaving only one gene out each time.: "Given that there are 30 genes within the set, 30 individual tests of RWR were run, leaving only one gene out each time."
