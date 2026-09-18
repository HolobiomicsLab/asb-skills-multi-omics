---
name: benchmark-output-verification
description: Use when a SignalingProfiler 2.0 (or CARNIVAL-based) run on the Olsen
  et al. EGF-stimulated HeLa phosphoproteomic dataset must be confirmed to reproduce
  the published mechanistic network before reuse or further analysis.
when_to_use_negative:
- Input is a non-EGF, non-HeLa dataset or a different perturbation, where the 84-node/256-edge
  and node-class benchmark figures do not apply.
- Only intermediate pipeline stages (footprint-based KSEA or phosphoscore outputs)
  are available and no optimized CARNIVAL network has been produced yet — there is
  nothing yet to verify against the benchmark.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SignalingProfiler 2.0
  role: Produces the network and edge annotations (expand_and_map_edges) whose outputs
    are compared to the published benchmark; installed from the SaccoPerfettoLab GitHub
    repository
  repo: github.com/SaccoPerfettoLab/SignalingProfiler
- name: CARNIVAL
  role: ILP optimization (cplex solver, mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192)
    generating the two-shots network whose union is the object under verification
  repo: https://saezlab.github.io/CARNIVAL/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:61b5937a28cdac6eccc00e86a59dcf1d5bf966a207e1bb18928821ea1a5d26ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - CARNIVAL
  - SignalingProfiler 2.0
  - VIPER
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# benchmark-output-verification

## Summary

Validates a SignalingProfiler 2.0 network reconstruction against the published EGF-stimulated HeLa benchmark (84 nodes / 256 edges, ~14% phosphorylation-mapped edges) by comparing inferred node classes, key recovered proteins, and the deposited NDEx network.

## When to use

Use when a SignalingProfiler 2.0 (or CARNIVAL-based) run on the Olsen et al. EGF-stimulated HeLa phosphoproteomic dataset must be confirmed to reproduce the published mechanistic network before reuse or further analysis.

## When NOT to use

- Input is a non-EGF, non-HeLa dataset or a different perturbation, where the 84-node/256-edge and node-class benchmark figures do not apply.
- Only intermediate pipeline stages (footprint-based KSEA or phosphoscore outputs) are available and no optimized CARNIVAL network has been produced yet — there is nothing yet to verify against the benchmark.

## Inputs

- SignalingProfiler 2.0 optimized network (union of the two CARNIVAL runs) as node/edge graph
- Edge annotation table from expand_and_map_edges with phosphorylation-evidence flags
- Published benchmark values (84 nodes, 256 edges, 14% phosphorylation-mapped edges; 56 kinases / 5 phosphatases / 3 TFs / 10 other proteins)
- Deposited EGF reference network on NDEx

## Outputs

- validation_summary.csv with benchmark-comparison results
- Node-count and edge-count verification report (84 nodes / 256 edges)
- List of recovered benchmark proteins (BRAF, RAF1, MAPK1-3, MAPK14, ATF7, JUN)
- Phosphorylation-mapped edge fraction (~14%)

## How to apply

After completing the two-shots CARNIVAL pipeline (run_footprint_based_analysis with analysis='ksea', phosphoscore_computation with activatory=TRUE, choose_PKN/preprocess_PKN, naive network with max_length_1=3 and max_length_2=4, default_CARNIVAL_options with mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192), check the optimized network size: 84 nodes and 256 edges, with ~14% of edges flagged by expand_and_map_edges (direct=FALSE, with_atlas=TRUE) as experimentally detected phosphorylation events. Verify inferred node composition against the benchmark: 56 kinases, 5 phosphatases, 3 transcription factors, and 10 other proteins. Confirm recovery of benchmark proteins BRAF, RAF1, MAPK1, MAPK3, MAPK14, ATF7, and JUN in the network. Cross-check the network against the EGF network deposited on NDEx, and write the comparison outcomes to validation_summary.csv. Rationale: parameter settings tuned on a training dataset exhibited comparable performance on validation datasets, so matching these published numbers indicates the pipeline was applied with correct default parameters.

## Related tools

- **SignalingProfiler 2.0** (Produces the network and edge annotations (expand_and_map_edges) whose outputs are compared to the published benchmark; installed from the SaccoPerfettoLab GitHub repository) — github.com/SaccoPerfettoLab/SignalingProfiler
- **CARNIVAL** (ILP optimization (cplex solver, mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192) generating the two-shots network whose union is the object under verification) — https://saezlab.github.io/CARNIVAL/

## Examples

```
devtools::install_github('https://github.com/SaccoPerfettoLab/SignalingProfiler/'); library(SignalingProfiler)
```

## Evaluation signals

- Network size equals the published 84 nodes and 256 edges after the union of the two CARNIVAL runs.
- Node composition matches the benchmark: 56 kinases, 5 phosphatases, 3 transcription factors, 10 other proteins.
- Fraction of edges flagged as experimentally detected phosphorylation events by expand_and_map_edges is ~14%.
- BRAF, RAF1, MAPK1, MAPK3, MAPK14, ATF7 and JUN are all present among the inferred network nodes.
- The constructed network matches the EGF network deposited on NDEx, and validation_summary.csv records all checks.

## Limitations

- Benchmark numbers are specific to the EGF-stimulated HeLa dataset with default parameters; other datasets or non-default parameters (e.g., different max_length_1/max_length_2 or connect_all settings) will not match these figures.
- Prior knowledge coverage is incomplete: regulon and causal interaction resources cover <50% (~9,000 proteins) of the UniProt-SwissProt proteome, so exact node-for-node identity with the deposited NDEx network may vary with database updates.
- SignalingProfiler is under active development and the README recommends regular reinstallation, which may shift results relative to the published benchmark.

## Evidence

- [other] Applied to the EGF-stimulated HeLa phosphoproteomics data, SignalingProfiler 2.0 produced a signaling network of 84 nodes and 256 edges, of which 14% were experimentally detected phosphorylation events.: [evidence span withheld — anchor status: not_found]
- [other] Verify the results against the published benchmark: 56 kinases, 5 phosphatases, 3 TFs and 10 other proteins inferred; an 84-node / 256-edge network with ~14% phosphorylation-mapped edges; recovery of BRAF, RAF1, MAPK1-3, MAPK14, ATF7 and JUN: [evidence span withheld — anchor status: not_found]
- [discussion] the parameter settings determined during the training process exhibited comparable performance (Fig. 6A) when applied to the validation datasets, demonstrating their broad applicability and limited: "the parameter settings determined during the training process exhibited comparable performance (Fig. 6A) when applied to the validation datasets, demonstrating their broad applicability and limited"
- [readme] devtools::install_github('https://github.com/SaccoPerfettoLab/SignalingProfiler/'): "devtools::install_github('https://github.com/SaccoPerfettoLab/SignalingProfiler/')"
- [readme] SignalingProfiler is currently under active development, as such we suggest to delete and re-install the package regularly: [evidence span withheld — anchor status: not_found]
