---
name: z-test-frequency-analysis
description: Use when building a context-specific subnetwork around omic-hit seeds
  and you need to decide which seed-anchored graphlet motifs are genuinely enriched
  in a reference interactome versus 100 seed-preserving permuted networks.
when_to_use_negative:
- Seed node set or reference interactome is unavailable — the z-test requires graphlet
  counts in the reference versus 100 permuted networks sharing the same initial node
  set
- The goal is downstream edge prioritization or subnetwork reconstruction — Personalized
  PageRank propagation and flux calculation run independently of the graphlet module,
  so this skill does not score or select edges
- You need a confidence-prefiltered interactome — pyPARAGON's reference interactome
  is not prefiltered based on a confidence threshold, so this skill cannot substitute
  for quality-based edge filtering
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: pyPARAGON
  role: 'Implements the graphlet module: enumerates G0–G8 seed-anchored graphlets,
    counts them in the reference and in 100 seed-preserving permuted networks, and
    applies the z-test (p < 0.05, z-score > 1.65) to output the GGN edge list'
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Runtime environment in which pyPARAGON's graphlet module is installed and
    executed (python3 venv setup described in the README)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:845ac1c2921b6ea839aca849f8ca10f5227d15aaebeef1a43273ecba403b67b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# z-test-frequency-analysis

## Summary

Statistically tests which of the nine non-isomorphic 2-, 3-, and 4-node graphlets (G0–G8) anchored on omic-hit seed nodes occur significantly more often in the reference interactome than expected by chance, forming the graphlet-guided network (GGN) used by pyPARAGON.

## When to use

Use when building a context-specific subnetwork around omic-hit seeds and you need to decide which seed-anchored graphlet motifs are genuinely enriched in a reference interactome versus 100 seed-preserving permuted networks.

## When NOT to use

- Seed node set or reference interactome is unavailable — the z-test requires graphlet counts in the reference versus 100 permuted networks sharing the same initial node set
- The goal is downstream edge prioritization or subnetwork reconstruction — Personalized PageRank propagation and flux calculation run independently of the graphlet module, so this skill does not score or select edges
- You need a confidence-prefiltered interactome — pyPARAGON's reference interactome is not prefiltered based on a confidence threshold, so this skill cannot substitute for quality-based edge filtering

## Inputs

- Reference interactome (network/edge list with optional confidence scores)
- Omic-hit seed node set (e.g., from omics data such as phosphoproteomic hits)
- 100 permuted networks generated to preserve the same initial seed node set

## Outputs

- Graphlet count vectors for the reference network and each of the 100 permuted networks (G0–G8)
- Per-graphlet z-scores and p-values from the z-test
- GGN subnetwork as an edge list file (ggn.edgelist), being the union of significant graphlets

## How to apply

Load the reference interactome and the omic-hit seed node set with pyPARAGON's graphlet module (Python). Enumerate the nine non-isomorphic graphlets (G0–G8) composed of 2-, 3-, and 4-node connected patterns over the seed-anchored regions; except for G0, place an intermediate/hidden node in one of the highest-degree orbits of each graphlet. Count occurrences of each targeted graphlet motif in the reference network, then generate 100 permuted networks that preserve the same initial node set and count graphlets in each permutation. Apply a z-test comparing each targeted graphlet count in the reference against the permuted-network distribution, calling a graphlet significant at p < 0.05 and z-score > 1.65. Emit the union of all significant graphlets as the GGN subnetwork in edge-list format (ggn.edgelist); the rationale is that frequent graphlets shrink the network to a context-relevant region and eliminate highly connected nodes and their unrelated interactions.

## Related tools

- **pyPARAGON** (Implements the graphlet module: enumerates G0–G8 seed-anchored graphlets, counts them in the reference and in 100 seed-preserving permuted networks, and applies the z-test (p < 0.05, z-score > 1.65) to output the GGN edge list) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Runtime environment in which pyPARAGON's graphlet module is installed and executed (python3 venv setup described in the README))

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- Every graphlet retained in the GGN meets the significance threshold p < 0.05 and z-score > 1.65 against the permuted-network distribution
- The GGN equals the union of significant graphlets, and every retained edge belongs to a graphlet passing the threshold
- All 100 permuted networks preserve exactly the same initial seed node set as the reference run
- The GGN is a subnetwork of the reference interactome — no edge appears in ggn.edgelist that is absent from the reference
- Only the nine non-isomorphic graphlets G0–G8 (2-, 3-, and 4-node connected patterns) are counted, with a hidden node assigned to a highest-degree orbit for graphlets larger than 2 nodes

## Limitations

- Graphlet assessment is restricted to 2-, 3-, and 4-node connected patterns (G0–G8), so larger or higher-order topological context is not captured
- Incomplete knowledge in large reference interactomes means inferred regions tend to favor protein complexes over metabolic pathways
- Hub proteins may dominate the inferred network with unrelated interactions; the graphlet shrinkage is a countermeasure but the method relies on the interactome as given (no prefiltering by confidence threshold)
- The method cannot handle alternative copies of hits such as protein isoforms and different post-translational modifications, so such information is generalized and potentially lost
- Significance requires generating and counting graphlets in 100 permuted networks, a potentially costly step on large interactomes

## Evidence

- [readme] z-test comparison of graphlet counts: "Using the z-test, we compared the targeted graphlet count in the reference and permuted networks (p < 0.05, z-score > 1.65)."
- [readme] permutation design and graphlet scope: "To uncover motifs, we counted the graphlets in 100 permuted networks that had the same initial node set."
- [methods] graphlet module role in GGN construction: "Graphlet module of pyPARAGON identifies, an associated region of reference network through substantially frequent non-isomorphic graphlets composed of 2-, 3-, and 4-nodes"
- [readme] hidden-node orbit assignment rule: "Except for G<sub>0</sub>, each graphlet includes initial nodes (red circles) and a hidden node (white circles) in one of the highest degrees of orbits."
