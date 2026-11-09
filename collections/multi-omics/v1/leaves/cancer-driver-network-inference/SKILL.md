---
name: cancer-driver-network-inference
description: Use when the goal is to infer a context-specific cancer network from
  mutated seed genes and quantitatively compare driver-gene recovery (precision/recall
  vs IntOGen) across network inference methods under cross-validation.
when_to_use_negative:
- The goal is patient stratification from phosphoproteomic data rather than benchmarking
  driver-gene recovery from mutation seeds
- Single-cell omics data must be modeled — pyPARAGON's current version integrates
  bulk omic data only, with single-cell support planned for a future release
- The task requires handling protein isoforms or post-translational modifications
  as distinct alternative copies of hits
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_2640
tools:
- name: pyPARAGON
  role: 'Primary method: GGN construction from 2-4 node graphlets, PPR propagation,
    flux-based edge scoring, subnetwork reconstruction'
  repo: https://github.com/netlab-ku/pyPARAGON
- name: PL
  role: Baseline comparison method for driver-gene precision/recall on the same seed
    folds
- name: OI2
  role: Baseline comparison method (prize-collecting Steiner Forest) for driver-gene
    precision/recall
- name: DOMINO
  role: Baseline comparison method for driver-gene precision/recall on the same seed
    folds
- name: Python
  role: Implementation/runtime environment for pyPARAGON (virtualenv-based install)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:44b67f029a7e84036b5c0512f67a829edaaa26db332459ce02fbb43adf4563ee
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cancer-driver-network-inference

## Summary

Reconstructs cancer-type-specific subnetworks from frequently mutated seed genes using pyPARAGON's graphlet-guided network plus Personalized PageRank flux, then benchmarks driver-gene recovery (precision/recall against IntOGen) against PathLinker, Omics Integrator 2, and DOMINO.

## When to use

Use when the goal is to infer a context-specific cancer network from mutated seed genes and quantitatively compare driver-gene recovery (precision/recall vs IntOGen) across network inference methods under cross-validation.

## When NOT to use

- The goal is patient stratification from phosphoproteomic data rather than benchmarking driver-gene recovery from mutation seeds
- Single-cell omics data must be modeled — pyPARAGON's current version integrates bulk omic data only, with single-cell support planned for a future release
- The task requires handling protein isoforms or post-translational modifications as distinct alternative copies of hits

## Inputs

- Reference interactome (network with edge confidence scores)
- Most frequently mutated gene sets per cancer type (eight cancer types), used as seed nodes
- IntOGen known cancer driver gene set for validation
- Cross-validation fold assignment (5 folds per cancer type)

## Outputs

- Context-specific subnetworks inferred per fold by pyPARAGON, PL, OI2, and DOMINO
- Graphlet-guided network (GGN) of significant graphlet motifs
- Per-fold precision and recall of recovered known driver genes for each method
- Aggregated precision/recall table across 5 folds and eight cancer types (Figure 2D comparison)

## How to apply

Load the reference interactome and the most frequently mutated gene sets for each of the eight cancer types, splitting seeds into 5 cross-validation folds. Run pyPARAGON per fold: build the graphlet-guided network (GGN) by counting non-isomorphic 2-, 3-, and 4-node graphlets (G0-G8) in 100 permuted networks and retaining significant graphlets via z-test (p < 0.05, z-score > 1.65); propagate seeds with the Personalized PageRank (PPR) algorithm; weight GGN edges by flux (PageRank scores, edge confidence scores, node interaction counts); and select highly scored edges to infer the context-specific subnetwork. Run PathLinker, Omics Integrator 2, and DOMINO on the same seed folds and interactome for comparison. Score recovered nodes per method and fold against the IntOGen known cancer driver set to compute precision and recall, then aggregate across the 5 folds and eight cancer types to reproduce the Figure 2D comparison. Rationale: graphlets trim the reference interactome and counteract hub dominance, so the reference interactome is not prefiltered by a confidence threshold.

## Related tools

- **pyPARAGON** (Primary method: GGN construction from 2-4 node graphlets, PPR propagation, flux-based edge scoring, subnetwork reconstruction) — https://github.com/netlab-ku/pyPARAGON
- **PL** (Baseline comparison method for driver-gene precision/recall on the same seed folds)
- **OI2** (Baseline comparison method (prize-collecting Steiner Forest) for driver-gene precision/recall)
- **DOMINO** (Baseline comparison method for driver-gene precision/recall on the same seed folds)
- **Python** (Implementation/runtime environment for pyPARAGON (virtualenv-based install))

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- Output table contains precision and recall entries for all four methods (pyPARAGON, PL, OI2, DOMINO) across all eight cancer types and 5 folds
- pyPARAGON recovers more known IntOGen driver genes and mostly achieves higher recall and precision than PL, OI2, and DOMINO, consistent with the paper's reported performance
- GGN construction retains only significant graphlets meeting the z-test thresholds (p < 0.05, z-score > 1.65) computed over 100 permuted networks
- Inferred subnetworks contain only highly scored edges (flux-weighted by PPR scores, edge confidence, and node interaction counts)
- Cross-validation protocol is consistent: same seed folds and reference interactome used for all four methods

## Limitations

- Hub proteins may dominate the inferred network with unrelated interactions
- Reference interactomes are incomplete, biasing inferred networks toward protein complexes over metabolic pathways
- Inferred networks frequently contain generic biological processes (e.g. transcription and replication), hindering interpretation
- The tool cannot handle alternative copies of hits such as protein isoforms and post-translational modifications; this information is generalized and potentially lost in the network
- No software version stated for pyPARAGON; no single-cell omic support in the current version

## Evidence

- [methods] graphlet-guided network construction: "Graphlet module of pyPARAGON identifies, an associated region of reference network through substantially frequent non-isomorphic graphlets composed of 2-, 3-, and 4-nodes"
- [methods] PPR propagation and flux edge scoring: "pyPARAGON propagates signals from omic-hits through the personalized PageRank algorithm in re-scoring all proteins in the reference"
- [readme] GGN significance thresholds: "Using the z-test, we compared the targeted graphlet count in the reference and permuted networks (p < 0.05, z-score > 1.65). The union of significant graphlets constructs GGN."
- [abstract] driver network performance: "Furthermore, pyPARAGON exhibits promising performance in discovering cancer driver networks."
- [discussion] comparison baselines: "we compared our method with three popular tools: PL, OI2, and DOMINO"
