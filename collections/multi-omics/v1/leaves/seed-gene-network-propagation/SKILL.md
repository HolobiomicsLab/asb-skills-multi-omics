---
name: seed-gene-network-propagation
description: Use when you have a set of seed genes (omic-hits such as frequently mutated
  genes per cancer type) and a reference interactome, and need a context-specific
  subnetwork whose recovered nodes can be scored against known drivers (IntOGen).
when_to_use_negative:
- Input includes protein isoforms or post-translational modification information that
  must be preserved — pyPARAGON cannot handle alternative copies of individual hits
  and this information is generalized and potentially lost
- Single-cell omic data — the current pyPARAGON version integrates bulk omic data
  for static network models only
- The goal is metabolic pathway reconstruction — incomplete reference interactomes
  bias inferred networks toward protein complexes over metabolic pathways
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0602
tools:
- name: pyPARAGON
  role: 'Primary tool: builds the GGN from significant graphlets, propagates seeds
    with Personalized PageRank, computes edge fluxes, and selects high-scoring edges
    to infer the context-specific subnetwork'
  repo: https://github.com/netlab-ku/pyPARAGON
- name: PL
  role: Baseline comparison method run on the same seed folds and reference interactome
    for precision/recall benchmarking
- name: OI2
  role: Baseline comparison method (prize-collecting Steiner Forest) run on the same
    folds for benchmarking
- name: DOMINO
  role: Baseline comparison method run on the same folds for benchmarking
- name: Python
  role: Execution environment; pyPARAGON is installed as a Python package via pip
    (python3 -m venv pyPARAGONenv)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:581344831b3fed45001511a4546e7085647eb0a3168dcabfb571a7a6b93b5f45
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
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# seed-gene-network-propagation

## Summary

Propagate seed genes (e.g., frequently mutated cancer genes) through a reference interactome using pyPARAGON's hybrid of graphlet-guided network construction and Personalized PageRank flux, then select highly scored edges to infer a context-specific subnetwork and benchmark its driver-gene recovery against PathLinker, DOMINO, and Omics Integrator 2.

## When to use

Use when you have a set of seed genes (omic-hits such as frequently mutated genes per cancer type) and a reference interactome, and need a context-specific subnetwork whose recovered nodes can be scored against known drivers (IntOGen).

## When NOT to use

- Input includes protein isoforms or post-translational modification information that must be preserved — pyPARAGON cannot handle alternative copies of individual hits and this information is generalized and potentially lost
- Single-cell omic data — the current pyPARAGON version integrates bulk omic data for static network models only
- The goal is metabolic pathway reconstruction — incomplete reference interactomes bias inferred networks toward protein complexes over metabolic pathways

## Inputs

- Reference interactome (network of protein-protein interactions, not prefiltered by confidence threshold)
- Seed gene sets: most frequently mutated genes per cancer type (eight cancer types, from Pan-Cancer Atlas-derived data)
- Known cancer driver gene set (IntOGen) for scoring recovered nodes
- Cross-validation fold assignment of seeds (5-fold)

## Outputs

- Context-specific subnetwork per fold (nodes and highly scored edges on the GGN)
- Per-fold precision and recall of recovered nodes against the IntOGen driver set
- Aggregated precision/recall comparison table for pyPARAGON, PL, OI2, and DOMINO (reproducing Figure 2D)

## How to apply

Split the seed genes into 5 cross-validation folds and run pyPARAGON on each fold against the reference interactome. First construct the graphlet-guided network (GGN) by counting nine non-isomorphic graphlets (G0–G8) of 2-, 3-, and 4-node compositions in 100 permuted networks with the same initial node set, retaining significant graphlets via z-test with p < 0.05 and z-score > 1.65; the union of significant graphlets forms the GGN and shrinks the interactome, eliminating hub-dominated unrelated interactions. Independently of the GGN, propagate signals from the omic-hits with the Personalized PageRank algorithm to re-score all proteins, then compute edge fluxes on the GGN weighting PageRank node scores, edge confidence scores, and node interaction counts. Select the highly scored edges to reconstruct the context-specific subnetwork. For benchmarking, run PathLinker (PL), Omics Integrator 2 (OI2), and DOMINO on the same folds and interactome, score recovered nodes against the IntOGen known driver set to compute per-fold precision and recall, aggregate across folds and the eight cancer types, and compare methods as in Figure 2D.

## Related tools

- **pyPARAGON** (Primary tool: builds the GGN from significant graphlets, propagates seeds with Personalized PageRank, computes edge fluxes, and selects high-scoring edges to infer the context-specific subnetwork) — https://github.com/netlab-ku/pyPARAGON
- **PL** (Baseline comparison method run on the same seed folds and reference interactome for precision/recall benchmarking)
- **OI2** (Baseline comparison method (prize-collecting Steiner Forest) run on the same folds for benchmarking)
- **DOMINO** (Baseline comparison method run on the same folds for benchmarking)
- **Python** (Execution environment; pyPARAGON is installed as a Python package via pip (python3 -m venv pyPARAGONenv))

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- Output table contains precision and recall entries for all four methods (pyPARAGON, PL, OI2, DOMINO) across all eight cancer types
- pyPARAGON-reconstructed networks recover more known IntOGen driver genes and mostly achieve higher recall and precision than PL, OI2, and DOMINO across the eight cancer types
- GGN construction retains only significant graphlets meeting the z-test thresholds (p < 0.05, z-score > 1.65) computed over 100 permuted networks
- 5-fold cross-validation is used consistently for every method on the same seed folds and reference interactome
- Precision/recall values aggregate cleanly across the 5 folds per cancer type before cross-method comparison

## Limitations

- Hub proteins may dominate the inferred network with unrelated interactions (partially mitigated by flux calculation and GGN trimming)
- Reference interactomes are incomplete, so inferred networks favor protein complexes over metabolic pathways
- Inferred networks frequently contain generic biological processes (e.g., transcription and replication), hindering interpretation
- No software version stated for pyPARAGON; the reference interactome is not prefiltered by a confidence threshold

## Evidence

- [other] pyPARAGON-reconstructed networks recover more known driver genes and mostly achieve higher recall and precision: [evidence span withheld — anchor status: not_found]
- [readme] GGN graphlet counting thresholds: "we counted the graphlets in 100 permuted networks that had the same initial node set. Using the z-test, we compared the targeted graphlet count in the reference and permuted networks (p < 0.05,"
- [methods] Propagation and edge scoring via PPR and flux calculation: "pyPARAGON propagates signals from omic-hits through the personalized PageRank algorithm in re-scoring all proteins in the reference"
- [methods] Subnetwork reconstruction: "Finally, pyPARAGON infers the context specific network by selecting the highly scored edges."
- [discussion] No confidence prefiltering of interactome: "Thus, the reference interactome is not prefiltered based on a confidence threshold."
- [abstract] Promising driver-network performance: "Furthermore, pyPARAGON exhibits promising performance in discovering cancer driver networks."
