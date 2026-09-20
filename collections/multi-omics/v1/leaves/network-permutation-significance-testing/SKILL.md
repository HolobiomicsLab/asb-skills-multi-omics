---
name: network-permutation-significance-testing
description: Use when you must decide which subnetwork edges are statistically enriched
  around a seed node set — e.g., trimming a large reference interactome to significant
  graphlets before propagation or edge inference.
when_to_use_negative:
- When the goal is sample-specific network inference from single-cell or bulk per-sample
  data — pyPARAGON's current version integrates bulk omic data only and handles no
  protein isoforms or PTMs.
- When the reference interactome has already been reduced to a small curated pathway
  and permutation testing would lack statistical power or be unnecessary.
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0602
tools:
- name: pyPARAGON
  role: Implements the graphlet module that enumerates G0–G8 graphlets, runs the 100-permutation
    z-test, and emits the GGN edge list
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Runtime environment in which pyPARAGON's graphlet and permutation-testing
    modules are executed
  repo: https://github.com/metunetlab/pyPARAGON
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:c298256fc1ea6a0175f2b0a166d91b70d105810686d8b8d736fbae926cef6be6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# network-permutation-significance-testing

## Summary

A statistical strategy for identifying context-relevant network motifs by comparing graphlet counts in a reference interactome against an ensemble of edge-swapped permuted networks, used by pyPARAGON to build its graphlet-guided network (GGN).

## When to use

Use when you must decide which subnetwork edges are statistically enriched around a seed node set — e.g., trimming a large reference interactome to significant graphlets before propagation or edge inference.

## When NOT to use

- When the goal is sample-specific network inference from single-cell or bulk per-sample data — pyPARAGON's current version integrates bulk omic data only and handles no protein isoforms or PTMs.
- When the reference interactome has already been reduced to a small curated pathway and permutation testing would lack statistical power or be unnecessary.

## Inputs

- Reference interactome (edge-list network)
- Omic-hit seed node set (e.g., phosphoproteomic hits from 105 CPTAC breast cancer tumors)
- 100 edge-swapped permuted networks preserving the initial seed node set

## Outputs

- Union of significant graphlets as the graphlet-guided network (GGN) subnetwork
- Edge-list file ggn.edgelist
- Per-graphlet significance statistics (p-value, z-score)

## How to apply

Enumerate the nine non-isomorphic graphlets (G0–G8) composed of 2-, 3-, and 4-node connected patterns over seed-anchored regions of the reference interactome, assigning an intermediate/hidden node in one of the highest-degree orbits for graphlets with more than 2 nodes (all except G0). Count occurrences of each targeted graphlet motif in the reference network, then generate 100 permuted networks that preserve the same initial (omic-hit seed) node set and count graphlets in each permutation. Apply a z-test comparing the targeted graphlet count in the reference against the permuted-network distribution, calling a graphlet significant at p < 0.05 and z-score > 1.65. The rationale is that graphlets filter out nonspecific interactions and mitigate the dominance of highly connected hubs, shrinking the reference interactome; the union of all significant graphlets is output as the GGN subnetwork (ggn.edgelist) for subsequent Personalized PageRank-flux edge scoring.

## Related tools

- **pyPARAGON** (Implements the graphlet module that enumerates G0–G8 graphlets, runs the 100-permutation z-test, and emits the GGN edge list) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Runtime environment in which pyPARAGON's graphlet and permutation-testing modules are executed) — https://github.com/metunetlab/pyPARAGON

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git; python -m pyPARAGON.graphlet --reference interactome.edgelist --seeds omic_hits.txt --permutations 100 --p 0.05 --z 1.65 --output ggn.edgelist
```

## Evaluation signals

- Every retained edge in the GGN belongs to a graphlet meeting the significance threshold p < 0.05 and z-score > 1.65.
- The GGN node/edge set is a strict subnetwork of the reference interactome, with hub-dominated nonspecific interactions eliminated relative to the input.
- All 100 permuted networks contain the identical initial seed node set as the reference, ensuring the null preserves seed membership.
- Output file ggn.edgelist is well-formed and its edges are each traceable to at least one of the nine non-isomorphic graphlets G0–G8.

## Limitations

- Reference interactomes are incomplete, biasing inferred networks toward protein complexes over metabolic pathways.
- Hub proteins may still dominate the inferred network with unrelated interactions if graphlet filtering does not eliminate them.
- No prefiltering of the reference interactome by confidence threshold is performed before the permutation test.
- The method cannot handle alternative copies of hits such as protein isoforms and post-translational modifications; this information is generalized and potentially lost.

## Evidence

- [readme] Union of significant graphlets forms GGN: "Using the z-test, we compared the targeted graphlet count in the reference and permuted networks (p < 0.05, z-score > 1.65). The union of significant graphlets constructs GGN."
- [readme] Graphlet enumeration and hidden node assignment: [evidence span withheld — anchor status: not_found]
- [readme] Permutation design preserving seed set: "To uncover motifs, we counted the graphlets in 100 permuted networks that had the same initial node set."
- [intro] Graphlets shrink network and counter hubs: "We found that graphlets filter out nonspecific interactions and mitigate the dominance of highly connected nodes, thereby trimming the reference interactome"
- [discussion] No confidence-threshold prefiltering: "Thus, the reference interactome is not prefiltered based on a confidence threshold."
