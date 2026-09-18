---
name: cross-validation-benchmark-of-network-methods
description: Use when comparing multiple network inference tools (e.g., pyPARAGON
  vs PL, OI2, DOMINO) on identical seed gene sets and a shared reference interactome,
  with known drivers as the validation ground truth.
when_to_use_negative:
- The goal is pathway reconstruction against curated signaling pathway maps rather
  than driver-gene recovery against a known-driver ground truth (pyPARAGON's pathway
  benchmark uses a different validation setup).
- Seed sets cannot be split into held-out folds (e.g., a single known seed list with
  no independent labels) — there is no cross-validation ground truth to score against.
- Input is single-cell omic data or requires isoform/PTM-specific seeding — pyPARAGON's
  current version integrates bulk omic data and cannot handle alternative copies of
  hits.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3311
- http://edamontology.org/topic_2640
tools:
- name: pyPARAGON
  role: 'Method under benchmark: reconstructs context-specific subnetworks per fold
    via graphlet-guided network construction, Personalized PageRank propagation, flux-based
    edge scoring, and highly scored edge selection'
  repo: https://github.com/netlab-ku/pyPARAGON
- name: PL
  role: 'Comparison baseline: run on the same seed folds and reference interactome
    for precision/recall against IntOGen drivers'
- name: OI2
  role: 'Comparison baseline (Omics Integrator 2): run on the same seed folds and
    reference interactome for precision/recall against IntOGen drivers'
- name: DOMINO
  role: 'Comparison baseline: run on the same seed folds and reference interactome
    for precision/recall against IntOGen drivers'
- name: Python
  role: Implementation/runtime environment for pyPARAGON (installed into a pyPARAGONenv
    virtual environment and run for each benchmark fold)
  repo: https://github.com/netlab-ku/pyPARAGON
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:5df472a7b4240352bb5cbe178284a7758f0e55ecea924627dae7854e2534e0b5
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3311
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cross-validation-benchmark-of-network-methods

## Summary

Benchmark context-specific network reconstruction methods head-to-head by running each tool on the same k-fold cross-validation seed splits and scoring recovered nodes against a known-driver ground truth (e.g., IntOGen) with precision and recall, as done to show pyPARAGON outperforms PathLinker, DOMINO, and Omics Integrator 2 in cancer driver recovery.

## When to use

Use when comparing multiple network inference tools (e.g., pyPARAGON vs PL, OI2, DOMINO) on identical seed gene sets and a shared reference interactome, with known drivers as the validation ground truth.

## When NOT to use

- The goal is pathway reconstruction against curated signaling pathway maps rather than driver-gene recovery against a known-driver ground truth (pyPARAGON's pathway benchmark uses a different validation setup).
- Seed sets cannot be split into held-out folds (e.g., a single known seed list with no independent labels) — there is no cross-validation ground truth to score against.
- Input is single-cell omic data or requires isoform/PTM-specific seeding — pyPARAGON's current version integrates bulk omic data and cannot handle alternative copies of hits.

## Inputs

- Reference interactome (network with edge confidence scores, not prefiltered by confidence threshold)
- Most frequently mutated gene sets per cancer type (eight cancer types), used as seed/omic-hits
- IntOGen known cancer driver gene set (ground truth for precision/recall)
- Cross-validation fold assignment (5-fold split of seed genes per cancer type)

## Outputs

- Per-fold inferred subnetworks for each method (pyPARAGON, PL, OI2, DOMINO)
- Per-method precision and recall against IntOGen drivers, aggregated across 5 folds and eight cancer types
- Comparative table/figure of precision and recall across all four methods (Figure 2D-style comparison)

## How to apply

Load the reference interactome and the most frequently mutated gene sets for each of the eight cancer types as seed sets, then split the seeds into 5 cross-validation folds so held-out seeds serve as the evaluation set. Run pyPARAGON per fold: build the graphlet-guided network (GGN) by counting non-isomorphic 2-, 3-, and 4-node graphlets (G0–G8) in 100 permuted networks with the same initial node set, retaining significant graphlets via z-test (p < 0.05, z-score > 1.65); propagate seeds with Personalized PageRank; compute edge fluxes weighting PageRank scores, edge confidence scores, and node interaction counts; and select highly scored edges to reconstruct the subnetwork. Run PathLinker (PL), Omics Integrator 2 (OI2), and DOMINO on the same folds and the same reference interactome so the comparison is controlled. For each method and fold, score recovered nodes against the IntOGen known cancer driver set to compute precision and recall, then aggregate across the 5 folds and eight cancer types (e.g., a Figure 2D precision/recall comparison). Rationale: fold-held-out seeds give an unbiased measure of each method's ability to recruit known drivers, and using the identical interactome isolates differences in propagation and edge-inference strategy.

## Related tools

- **pyPARAGON** (Method under benchmark: reconstructs context-specific subnetworks per fold via graphlet-guided network construction, Personalized PageRank propagation, flux-based edge scoring, and highly scored edge selection) — https://github.com/netlab-ku/pyPARAGON
- **PL** (Comparison baseline: run on the same seed folds and reference interactome for precision/recall against IntOGen drivers)
- **OI2** (Comparison baseline (Omics Integrator 2): run on the same seed folds and reference interactome for precision/recall against IntOGen drivers)
- **DOMINO** (Comparison baseline: run on the same seed folds and reference interactome for precision/recall against IntOGen drivers)
- **Python** (Implementation/runtime environment for pyPARAGON (installed into a pyPARAGONenv virtual environment and run for each benchmark fold)) — https://github.com/netlab-ku/pyPARAGON

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- Output table contains precision and recall entries for all four methods (pyPARAGON, PL, OI2, DOMINO) across all eight cancer types and 5 folds.
- pyPARAGON-reconstructed networks recover more known IntOGen driver genes than PathLinker, DOMINO, and Omics Integrator 2, with mostly higher recall and precision across the eight cancer types.
- Each method was run on the identical reference interactome and identical seed folds — verify no per-method differences in interactome or seed preprocessing (e.g., the reference interactome is not prefiltered by confidence threshold).
- pyPARAGON GGN construction per fold used 100 permuted networks and retained significant graphlets with z-test p < 0.05 and z-score > 1.65.
- Aggregated precision/recall values are consistent with the paper's reported promising performance in discovering cancer driver networks (Figure 2D).

## Limitations

- Reference interactomes are incomplete, biasing inferred networks toward protein complexes over metabolic pathways, which affects all compared methods.
- Inferred networks frequently contain generic biological processes (e.g., transcription and replication), hindering interpretation of the recovered subnetworks.
- Hub proteins may dominate inferred networks with unrelated interactions; pyPARAGON addresses this via flux calculation rather than hub penalization, so method-specific biases must be considered when comparing tools.
- pyPARAGON cannot handle alternative copies of hits such as protein isoforms and post-translational modifications, so seed-gene information may be generalized and lost.

## Evidence

- [discussion] Benchmark framing: "we compared our method with three popular tools: PL, OI2, and DOMINO"
- [abstract] Driver-network performance claim: "Furthermore, pyPARAGON exhibits promising performance in discovering cancer driver networks."
- [readme] pyPARAGON three-step inference procedure used per fold: "During network inference, pyPARAGON runs in three steps (Figure 1.A): i. Graphlet-guided network (GGN) construction, ii. Propagation and edge scoring via the Personalized PageRank (PPR) algorithm and"
- [readme] GGN significance thresholds: "Using the z-test, we compared the targeted graphlet count in the reference and permuted networks (p < 0.05, z-score > 1.65). The union of significant graphlets constructs GGN."
- [discussion] No interactome prefiltering — controlled comparison condition: "Thus, the reference interactome is not prefiltered based on a confidence threshold."
