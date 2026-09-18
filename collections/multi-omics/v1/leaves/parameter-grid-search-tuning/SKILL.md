---
name: parameter-grid-search-tuning
description: Use when benchmarking a network-inference or propagation tool against
  current competitors and each tool's parameters must be optimized fairly over shared
  seed gene sets and reference interactomes before scoring recall/precision.
when_to_use_negative:
- Input omic data are single-cell or contain alternative copies of hits (protein isoforms,
  PTMs) — pyPARAGON handles bulk omic data and generalizes such information
- No curated gold-standard node/edge sets exist for the target pathways, since AUPRC/F1-based
  grid-search selection cannot be computed
- The reference interactome should be prefiltered by confidence threshold — pyPARAGON
  explicitly does not prefilter on confidence
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: pyPARAGON
  role: 'Tool being tuned: graphlet-guided network construction, PPR propagation,
    flux edge scoring, subnetwork reconstruction with grid-searched parameters'
  repo: https://github.com/netlab-ku/pyPARAGON
- name: PL
  role: Baseline (PathLinker) run on the same seed sets and interactomes with grid-searched
    parameters
- name: OI2
  role: Baseline (Omics Integrator 2) run on the same seed sets and interactomes with
    grid-searched parameters
- name: DOMINO
  role: Baseline run on the same seed sets and interactomes with grid-searched parameters
- name: Python
  role: Runtime environment for pyPARAGON (python3 -m venv pyPARAGONenv)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:21d5e4201d9097fb99f31ed56f36f588a14abb9368f1639f91d524fba4c4592f
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
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# parameter-grid-search-tuning

## Summary

Grid-search the free parameters of pyPARAGON (graphlet-guided network construction, Personalized PageRank propagation, flux-based edge scoring) and its baselines (PathLinker, Omics Integrator 2, DOMINO) so each tool is tuned on the same seed sets and interactomes before computing node- and edge-level AUPRC/F1 against curated NetPath gold standards.

## When to use

Use when benchmarking a network-inference or propagation tool against state-of-the-art competitors and each tool's parameters must be optimized fairly over shared seed gene sets and reference interactomes before scoring recall/precision.

## When NOT to use

- Input omic data are single-cell or contain alternative copies of hits (protein isoforms, PTMs) — pyPARAGON handles bulk omic data and generalizes such information
- No curated gold-standard node/edge sets exist for the target pathways, since AUPRC/F1-based grid-search selection cannot be computed
- The reference interactome should be prefiltered by confidence threshold — pyPARAGON explicitly does not prefilter on confidence

## Inputs

- Reference interactomes: ConsensusPathDB, HIPPIE v2.2, HIPPIE v2.3
- NetPath pathway seed gene sets (omic-hits)
- Curated NetPath gold-standard node and edge sets
- Grid-search parameter ranges for pyPARAGON, PathLinker, Omics Integrator 2, and DOMINO

## Outputs

- Grid-searched optimal parameter settings per tool/interactome/pathway combination
- Node-level and edge-level AUPRC and F1 metrics per tool/interactome/pathway
- Aggregated comparison panels reproducing Fig. 2B-C (node propagation and edge inference)

## How to apply

Load the reference interactomes (ConsensusPathDB, HIPPIE v2.2, HIPPIE v2.3) and the NetPath pathway seed gene sets, then run pyPARAGON — GGN construction over 9 non-isomorphic 2-4 node graphlets, PPR propagation from omic-hits, flux edge scoring, and high-edge subnetwork reconstruction — while grid-searching its parameters over the allowed ranges. Apply the same grid-search procedure to the baselines PL, OI2, and DOMINO so no method is advantaged by hand-tuned settings. Rationale: graphlet significance is fixed by a z-test against 100 permuted networks (p < 0.05, z-score > 1.65), so tuning targets the remaining free parameters of propagation and edge selection rather than the graphlet filter. For every tool/interactome/pathway combination, compute node-level and edge-level AUPRC and F1 against the curated NetPath gold-standard node and edge sets, then aggregate metrics across pathways and interactomes to render comparison panels (Fig. 2B-C). Correctness is judged by whether pyPARAGON achieves the highest node-propagation and edge-inference AUPRC/F1 relative to PL, OI2, and DOMINO under its grid-searched optimum.

## Related tools

- **pyPARAGON** (Tool being tuned: graphlet-guided network construction, PPR propagation, flux edge scoring, subnetwork reconstruction with grid-searched parameters) — https://github.com/netlab-ku/pyPARAGON
- **PL** (Baseline (PathLinker) run on the same seed sets and interactomes with grid-searched parameters)
- **OI2** (Baseline (Omics Integrator 2) run on the same seed sets and interactomes with grid-searched parameters)
- **DOMINO** (Baseline run on the same seed sets and interactomes with grid-searched parameters)
- **Python** (Runtime environment for pyPARAGON (python3 -m venv pyPARAGONenv))

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- Grid search must be applied to all four tools (pyPARAGON, PL, OI2, DOMINO) on the identical seed sets and interactomes — a tuned pyPARAGON compared against untuned baselines is invalid
- Graphlet module parameters are fixed by the z-test against 100 permuted networks (p < 0.05, z-score > 1.65), not tuned
- Metrics must be computed at both node-propagation and edge-inference levels as AUPRC and F1 against the NetPath gold standards
- Aggregated results across pathways and interactomes should reproduce the Fig. 2B-C comparison panels
- Expected outcome: pyPARAGON attains the highest node and edge AUPRC/F1; in F1 terms, pyPARAGON and PL are better at propagation, pyPARAGON and OI2 better at network inference

## Limitations

- No software version is stated for pyPARAGON, complicating reproducibility of grid-searched optima
- Reference interactomes are incomplete, biasing inferred networks toward protein complexes over metabolic pathways
- Hub proteins may dominate inferred networks with unrelated interactions (the flux calculation only partially counters this)
- Tool cannot handle alternative copies of hits such as protein isoforms and post-translational modifications

## Evidence

- [other] run pyPARAGON — graphlet-guided network construction over 9 non-isomorphic graphlets (2-4 nodes, tested against 100 permuted networks by z-test with p < 0.05 and z-score > 1.65), Personalized PageRank propagation: [evidence span withheld — anchor status: not_found]
- [other] Run the three baselines PL, OI2, and DOMINO on the same seed sets and interactomes with grid-searched parameters.: [evidence span withheld — anchor status: not_found]
- [other] pyPARAGON outperformed PathLinker, Omics Integrator 2, and DOMINO at both the node and edge levels for inferring all NetPath signaling pathways: [evidence span withheld — anchor status: not_found]
- [readme] pyPARAGON is available at https://github.com/netlab-ku/pyPARAGON.: "pyPARAGON is available at https://github.com/netlab-ku/pyPARAGON."
- [abstract] Through comprehensive evaluations on benchmark signaling pathways, we demonstrate that pyPARAGON outperforms state-of-the-art approaches in node propagation and edge inference.: "Through comprehensive evaluations on benchmark signaling pathways, we demonstrate that pyPARAGON outperforms state-of-the-art approaches in node propagation and edge inference."
