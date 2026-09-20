---
name: random-walk-restart-gene-ranking
description: Use when a curated seed gene set (e.g., MAPMAN- or GO-derived) must be
  scored or validated for functional interconnectivity on a multi-layer multiplex
  network, and left-out genes should be ranked by RWR proximity (AUROC/AUPRC).
when_to_use_negative:
- When the question is layer-by-layer topology description rather than seed-based
  gene ranking/validation (use RWR_netstats instead).
- When the network is a single aggregated summary graph, since it loses the unique
  topological information carried by each layer that RWR on the multiplex exploits.
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: RWRtoolkit
  role: Provides RWR_CV (k-fold/LOO cross-validation), RWR_LOE, RWR_make_multiplex,
    and command-line scripts for RWR gene ranking on multiplex networks
  repo: github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Core functionality for constructing multiplex networks and implementing random
    walks with restart (compute.adjacency.matrix, compute.transition.matrix, Random.Walk.Restart.Multiplex)
  repo: github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: R
  role: Environment in which the RWR analysis and cross-validation workflow is run
    (R >= 4.1.0 required)
- name: dkainer/RWRtoolkit-data
  role: Hosts preassembled Arabidopsis multiplex RData objects (e.g., Comprehensive_Network_AT_d0.5_v02.RData)
    used as RWR_CV input
  repo: github.com/dkainer/RWRtoolkit-data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:39379ab13672659f28e7b96d38b96ea1233cc46e5d80781c4c14fd6fbf53a86f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - RWRtoolkit
  - RandomWalkRestartMH
  - dkainer/RWRtoolkit-data
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 20
---

# random-walk-restart-gene-ranking

## Summary

Rank genes by topological connectivity to seed gene sets using Random Walk with Restart (RWR) on multiplex networks, with RWRtoolkit/R functions such as RWR_CV and RWR_LOE, and validate ranking power via k-fold cross-validation (e.g., k=5) against rewired network controls.

## When to use

Use when a curated seed gene set (e.g., MAPMAN- or GO-derived) must be scored or validated for functional interconnectivity on a multi-layer multiplex network, and left-out genes should be ranked by RWR proximity (AUROC/AUPRC).

## When NOT to use

- When the question is layer-by-layer topology description rather than seed-based gene ranking/validation (use RWR_netstats instead).
- When the network is a single aggregated summary graph, since it loses the unique topological information carried by each layer that RWR on the multiplex exploits.

## Inputs

- Preassembled multiplex network RData object (e.g., Comprehensive_Network_AT_d0.5_v02.RData with 9 unweighted layers at delta=0.5)
- Curated gold-standard gene set files (e.g., 25 MAPMAN-derived gene sets)
- flist file listing network layers as {file_path, name, group} triples
- Seed gene set file (e.g., geneset1.txt with ATG locus IDs)

## Outputs

- Per-gene-set, per-fold AUROC/AUPRC summary statistics
- Aggregated mean AUROC for real vs. rewired multiplexes (e.g., rwr_cv_kfold_auroc_results.csv)
- RWR ranking scores of genes with respect to seed sets (RWR_LOE)

## How to apply

Build the multiplex with RWRtoolkit's RWR_make_multiplex/create.multiplex from an flist of network layers, then run RWR_CV on each curated gene set (e.g., method='kfold', k=5, restart probability 0.7) against the multiplex (such as the 9-layer comprehensive Arabidopsis thaliana multiplex with delta=0.5), collecting AUROC/AUPRC per fold. As a negative control, generate ~1,000 randomly rewired multiplexes preserving node and edge counts per layer and rerun the same k-fold RWR_CV on them. Rationale: functional gene groups should have strong topological similarity, so real-network AUROC (reported ~0.91) should far exceed the rewired-control expectation (~0.49-0.50). Aggregate per-gene-set AUROC values, compute means across gene sets for real vs. rewired multiplexes, and write results to a CSV such as rwr_cv_kfold_auroc_results.csv. For interactive seed-vs-other-gene ranking, use RWR_LOE with a seed set file; underlying machinery is compute.adjacency.matrix, compute.transition.matrix, and Random.Walk.Restart.Multiplex from RandomWalkRestartMH.

## Related tools

- **RWRtoolkit** (Provides RWR_CV (k-fold/LOO cross-validation), RWR_LOE, RWR_make_multiplex, and command-line scripts for RWR gene ranking on multiplex networks) — github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Core functionality for constructing multiplex networks and implementing random walks with restart (compute.adjacency.matrix, compute.transition.matrix, Random.Walk.Restart.Multiplex)) — github.com/alberto-valdeolivas/RandomWalkRestartMH
- **R** (Environment in which the RWR analysis and cross-validation workflow is run (R >= 4.1.0 required))
- **dkainer/RWRtoolkit-data** (Hosts preassembled Arabidopsis multiplex RData objects (e.g., Comprehensive_Network_AT_d0.5_v02.RData) used as RWR_CV input) — github.com/dkainer/RWRtoolkit-data

## Examples

```
Rscript $LIBPATHS/RWRtoolkit/scripts/run_loe.R --data $LIBPATHS/RWRtoolkit/example_data/string_interactions.Rdata --seed_geneset geneset1.txt --tau "1.0,1.0" -o ./outdir
```

## Evaluation signals

- Average AUROC across the 25 MAPMAN-derived gene sets and 5 folds should reproduce ~0.91 on the real comprehensive multiplex.
- Mean AUROC across ~1,000 randomly rewired multiplexes should be ~0.49, near the random expectation of 0.50.
- RWR_CV output must contain per-gene-set, per-fold AUROC/AUPRC summary statistics and be written to the results CSV.
- The multiplex object must retain all layers (9 layers, delta=0.5) as defined in the flist before running RWR_CV.
- Rewired controls must preserve the number of nodes and edges in every layer.

## Limitations

- RandomWalkRestartMH's tau did not correctly weight seeds per layer in the original implementation, so restart/seed-weighting behavior depends on the updated RWRtoolkit base.
- Multiplex construction in the unoptimized RandomWalkRestartMH scales poorly (exponentially increasing computational load with network size and layer count); use RWRtoolkit's parallelized construction.
- No version number is stated for RWRtoolkit or RandomWalkRestartMH in the article, complicating reproducibility pinning.
- Installation requires R >= 4.1.0 and can hit Bioconductor SSL certificate errors needing explicit CA certificate configuration.

## Evidence

- [discussion] RWRtoolkit uses RWR to explore and score topological connectivity from multiple lines of evidence between seed genes and other genes.: "RWRtoolkit uses RWR to explore and score topological connectivity from multiple lines of evidence between seed genes and other genes."
- [methods] we employed Random Walk with Restart Cross-Validation (RWR_CV) using a series of carefully curated gene sets: "we employed Random Walk with Restart Cross-Validation (RWR_CV) using a series of carefully curated gene sets"
- [discussion] we have included RWR_LOE and RWR_CV as applications within KBase and provided preassembled Arabidopsis thaliana multiplex networks.: "we have included RWR_LOE and RWR_CV as applications within KBase and provided preassembled Arabidopsis thaliana multiplex networks."
- [readme] RWRtoolkit enables easy use of RandomWalk with Restart on multiplex networks. These functions are an extension to the RandomWalkRestartMH R package.: "RWRtoolkit enables easy use of RandomWalk with Restart on multiplex networks. These functions are an extension to the"
- [discussion] The example cross-validation experiment conducted on our 9-layer comprehensive Arabidopsis multiplex demonstrates that functionally related groups of genes have strong topological similarity: "The example cross-validation experiment conducted on our 9-layer comprehensive Arabidopsis multiplex demonstrates that functionally related groups of genes have strong topological similarity"
