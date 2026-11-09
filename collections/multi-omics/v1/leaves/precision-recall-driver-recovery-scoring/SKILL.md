---
name: precision-recall-driver-recovery-scoring
description: Use when evaluating how well a network reconstruction method recovers
  known driver genes (e.g., IntOGen) from mutated-gene seed sets, requiring precision/recall
  comparison against baseline methods under k-fold cross-validation.
when_to_use_negative:
- The goal is pathway reconstruction against benchmark signaling pathways rather than
  cancer driver gene recovery — that evaluation uses a different ground-truth set.
- Only a single seed set with no held-out fold exists, so precision/recall against
  known drivers cannot be cross-validated.
- Input omic-hits are protein isoforms or post-translational modification-level hits,
  which the evaluated network tools cannot handle as alternative copies.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_2640
tools:
- name: pyPARAGON
  role: 'Method under evaluation: reconstructs context-specific subnetworks from seed
    folds via graphlet-guided network, Personalized PageRank propagation, and edge
    flux scoring'
  repo: https://github.com/netlab-ku/pyPARAGON
- name: PL
  role: Baseline network inference method (PathLinker) run on the same seed folds
    and reference interactome for precision/recall comparison
- name: OI2
  role: Baseline network inference method (Omics Integrator 2) run on the same seed
    folds and reference interactome for comparison
- name: DOMINO
  role: Baseline network inference method run on the same seed folds and reference
    interactome for comparison
- name: Python
  role: Implementation environment for pyPARAGON and the evaluation scripts (venv-based
    install)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:e612623690b15344df3c88ad29c950d2ef11e59714b1361086a6ad5864d58f15
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
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# precision-recall-driver-recovery-scoring

## Summary

Benchmarking workflow that scores nodes recovered by context-specific network inference methods (pyPARAGON, PathLinker, Omics Integrator 2, DOMINO) against the IntOGen known cancer driver set to compute per-fold precision and recall, aggregated across 5 cross-validation folds and eight cancer types.

## When to use

Use when evaluating how well a network reconstruction method recovers known driver genes (e.g., IntOGen) from mutated-gene seed sets, requiring precision/recall comparison against baseline methods under k-fold cross-validation.

## When NOT to use

- The goal is pathway reconstruction against benchmark signaling pathways rather than cancer driver gene recovery — that evaluation uses a different ground-truth set.
- Only a single seed set with no held-out fold exists, so precision/recall against known drivers cannot be cross-validated.
- Input omic-hits are protein isoforms or post-translational modification-level hits, which the evaluated network tools cannot handle as alternative copies.

## Inputs

- Reference interactome (network of protein interactions, unfiltered by confidence threshold)
- Most frequently mutated gene sets (seed nodes) for each of eight cancer types
- IntOGen known cancer driver gene set (ground-truth for scoring)
- Inferred subnetworks / recovered node sets from each method (pyPARAGON, PL, OI2, DOMINO) per cross-validation fold

## Outputs

- Per-fold precision and recall values for each method and cancer type
- Aggregated precision/recall table across 5 folds and eight cancer types for all four methods
- Figure 2D-style precision-vs-recall comparison across pyPARAGON, PathLinker, OI2, and DOMINO

## How to apply

Load the reference interactome plus the most frequently mutated gene sets for each of the eight cancer types, splitting seeds into 5 cross-validation folds so held-out seeds can be treated as ground-truth recovery targets. Run each method (pyPARAGON's graphlet-guided network construction with 100 permuted networks, z-test p < 0.05, z-score > 1.65, Personalized PageRank propagation and edge flux selection; plus PL, OI2, and DOMINO) on the same seed folds and reference interactome. For each method and fold, score the recovered nodes against the IntOGen known cancer driver set to compute precision and recall. Aggregate precision/recall across the 5 folds and eight cancer types for all four methods, then compare (e.g., Figure 2D-style comparison) to determine whether the method of interest recovers more known drivers with higher precision and recall.

## Related tools

- **pyPARAGON** (Method under evaluation: reconstructs context-specific subnetworks from seed folds via graphlet-guided network, Personalized PageRank propagation, and edge flux scoring) — https://github.com/netlab-ku/pyPARAGON
- **PL** (Baseline network inference method (PathLinker) run on the same seed folds and reference interactome for precision/recall comparison)
- **OI2** (Baseline network inference method (Omics Integrator 2) run on the same seed folds and reference interactome for comparison)
- **DOMINO** (Baseline network inference method run on the same seed folds and reference interactome for comparison)
- **Python** (Implementation environment for pyPARAGON and the evaluation scripts (venv-based install))

## Examples

```
python3 -m venv pyPARAGONenv && source pyPARAGONenv/bin/activate && pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- Output table contains precision and recall entries for all four methods (pyPARAGON, PL, OI2, DOMINO) across all eight cancer types and 5 folds.
- pyPARAGON's driver-recovery performance matches the paper's reported promising performance: more known driver genes recovered, mostly higher recall and precision than the baselines.
- Every seed fold is scored against the same IntOGen driver set, so precision values are bounded in [0,1] and comparable across methods.
- Aggregated results are reproducible under the fixed 5-fold cross-validation split of the most frequently mutated genes.

## Limitations

- Reference interactomes are incomplete, biasing inferred networks toward protein complexes over metabolic pathways, which affects which nodes can be recovered.
- Inferred networks frequently contain generic biological processes (transcription, replication), complicating biological interpretation of recovery results.
- The methods cannot handle alternative copies of hits such as protein isoforms and post-translational modifications; this information is generalized and potentially lost.
- No software version stated for pyPARAGON; the reference interactome is not prefiltered based on a confidence threshold.

## Evidence

- [discussion] evaluation vs IntOGen: "we compared our method with three popular tools: PL, OI2, and DOMINO"
- [readme] graphlet significance thresholds: "Using the z-test, we compared the targeted graphlet count in the reference and permuted networks (p < 0.05, z-score > 1.65)."
- [abstract] driver recovery finding: "Furthermore, pyPARAGON exhibits promising performance in discovering cancer driver networks."
- [methods] propagation and edge scoring: "pyPARAGON propagates signals from omic-hits through the personalized PageRank algorithm in re-scoring all proteins in the reference"
- [methods] edge selection step: "Finally, pyPARAGON infers the context specific network by selecting the highly scored edges."
- [discussion] no interactome prefiltering: "Thus, the reference interactome is not prefiltered based on a confidence threshold."
