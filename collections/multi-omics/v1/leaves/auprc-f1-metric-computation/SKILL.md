---
name: auprc-f1-metric-computation
description: Use when benchmarking network propagation or network inference methods
  against a curated pathway gold standard, where both node recovery (propagation)
  and edge recovery (inference) must be quantified separately.
when_to_use_negative:
- When evaluating a single method without baselines and only qualitative assessment
  is needed — AUPRC/F1 benchmarking requires a gold standard and comparable ranked
  predictions.
- When the pathway of interest lacks a curated gold-standard node/edge set (e.g.,
  pathways not represented in NetPath); metrics would be computed against an unreliable
  reference.
- When evaluating patient stratification or module-level outputs rather than node/edge
  reconstruction — the article assesses those via functional modules and clustering,
  not AUPRC/F1.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0605
tools:
- name: pyPARAGON
  role: Method under benchmarking; supplies ranked nodes (PPR propagation) and scored
    edges (flux calculation) for AUPRC/F1 computation
  repo: https://github.com/netlab-ku/pyPARAGON
- name: PL
  role: Baseline network propagation/inference tool compared at node and edge levels
- name: OI2
  role: Baseline prize-collecting Steiner forest network inference tool compared at
    node and edge levels
- name: DOMINO
  role: Baseline active-module network inference tool compared at node and edge levels
- name: Python
  role: Environment for running pyPARAGON and computing AUPRC/F1 metrics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:31af94b2a076c0c8b0d7047f83462ba2d03ee5a6b1ce3fbc99f920b470506880
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - DOMINO
  - OI2
  - PL
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# auprc-f1-metric-computation

## Summary

Computes node-level and edge-level AUPRC and F1 scores to benchmark context-specific network reconstruction tools (e.g., pyPARAGON vs. PL, OI2, DOMINO) against curated gold-standard node and edge sets from NetPath signaling pathways.

## When to use

Use when benchmarking network propagation or network inference methods against a curated pathway gold standard, where both node recovery (propagation) and edge recovery (inference) must be quantified separately.

## When NOT to use

- When evaluating a single method without baselines and only qualitative assessment is needed — AUPRC/F1 benchmarking requires a gold standard and comparable ranked predictions.
- When the pathway of interest lacks a curated gold-standard node/edge set (e.g., pathways not represented in NetPath); metrics would be computed against an unreliable reference.
- When evaluating patient stratification or module-level outputs rather than node/edge reconstruction — the article assesses those via functional modules and clustering, not AUPRC/F1.

## Inputs

- Inferred/selected node set or ranked node scores from a network propagation method
- Inferred/selected edge set or ranked edge scores (e.g., PageRank-flux weighted edges) from a network inference method
- Curated gold-standard node set (e.g., NetPath pathway genes)
- Curated gold-standard edge set (e.g., NetPath signaling interactions)
- Predictions from comparison tools (PL, Omics Integrator 2, DOMINO) on identical seed sets and reference interactomes (ConsensusPathDB, HIPPIE v2.2, HIPPIE v2.3)

## Outputs

- Node-level AUPRC and F1 per tool/interactome/pathway combination
- Edge-level AUPRC and F1 per tool/interactome/pathway combination
- Aggregated metric comparison across pathways and interactomes (node- and edge-level benchmark panels)

## How to apply

For each tool/interactome/pathway combination, run the network reconstruction method on the same seed sets and reference interactomes (e.g., ConsensusPathDB, HIPPIE v2.2, HIPPIE v2.3) with grid-searched parameters. Compare the inferred network against the curated gold-standard node set (for propagation) and gold-standard edge set (for inference), computing AUPRC by ranking predicted nodes/edges and sweeping the reconstruction threshold. Compute F1 as the harmonic mean of precision and recall at each threshold to identify the best-operating-point performance. Aggregate metrics across pathways and interactomes to render comparative panels (e.g., the node-level and edge-level comparison panels of Fig. 2B-C in the pyPARAGON paper). Validate that the claimed ranking (pyPARAGON highest on both node-propagation and edge-inference AUPRC/F1 relative to PL, OI2, and DOMINO) is reproduced.

## Related tools

- **pyPARAGON** (Method under benchmarking; supplies ranked nodes (PPR propagation) and scored edges (flux calculation) for AUPRC/F1 computation) — https://github.com/netlab-ku/pyPARAGON
- **PL** (Baseline network propagation/inference tool compared at node and edge levels)
- **OI2** (Baseline prize-collecting Steiner forest network inference tool compared at node and edge levels)
- **DOMINO** (Baseline active-module network inference tool compared at node and edge levels)
- **Python** (Environment for running pyPARAGON and computing AUPRC/F1 metrics)

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- pyPARAGON achieves the highest node-propagation and edge-inference AUPRC/F1 relative to PL, OI2, and DOMINO on NetPath pathways, reproducing Fig. 2B-C
- Metrics are computed per tool/interactome/pathway combination with grid-searched parameters, so no tool is disadvantaged by untuned settings
- Node-level and edge-level metrics are reported separately: pyPARAGON and PL lead at propagation, pyPARAGON and OI2 lead at network inference in F1 comparison
- Gold-standard node and edge sets are drawn from curated NetPath signaling pathways, so every predicted node/edge has a verifiable truth label

## Limitations

- Findings are limited to benchmark signaling pathways with curated gold standards (NetPath); performance on pathways lacking complete annotations may be biased
- Reference interactomes are incomplete, so inferred networks bias toward protein complexes over metabolic pathways, which can distort edge-level ground truth comparisons
- Inferred networks may contain generic biological processes, complicating whether metric gains reflect biologically meaningful edge recovery
- Results are context-specific to the three reference interactomes and grid-searched parameter settings used in the benchmark

## Evidence

- [abstract] pyPARAGON outperformed PathLinker, Omics Integrator 2, and DOMINO at both the node and edge levels for inferring all NetPath signaling pathways: "Through comprehensive evaluations on benchmark signaling pathways, we demonstrate that pyPARAGON outperforms state-of-the-art approaches in node propagation and edge inference."
- [methods] For each tool/interactome/pathway combination, compute node-level and edge-level AUPRC and F1 against the curated NetPath gold-standard node and edge sets.: "Finally, pyPARAGON infers the context specific network by selecting the highly scored edges."
- [discussion] pyPARAGON and PL were better at propagation, while pyPARAGON and OI2 were better at network inference in F1 comparison.: "we compared our method with three popular tools: PL, OI2, and DOMINO"
- [readme] Grid-searched parameters were used for pyPARAGON and the three baselines on the same seed sets and interactomes.: "pyPARAGON infers a context specific subnetwork from a given reference network, utilizing omic-hits as a seed node set"
