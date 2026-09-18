---
name: reference-interactome-subnetwork-extraction
description: Use when given a reference interactome plus an omic-hit seed node set
  (e.g. phosphoproteomic hits from CPTAC breast cancer samples) and you need to prune
  the interactome to a seed-anchored, context-specific region before edge scoring
  or propagation.
when_to_use_negative:
- Input is already a small, curated context-specific pathway network — graphlet-based
  pruning is designed to shrink a large reference interactome and would add little
  value
- Analysis requires protein isoform or post-translational-modification-specific nodes
  — pyPARAGON generalizes this information and it is potentially lost in the network
- Single-cell omic data is the input — the current pyPARAGON version integrates bulk
  omic data for static networks only
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0602
tools:
- name: pyPARAGON
  role: Provides the graphlet module that enumerates G0-G8 graphlets, counts motifs
    in reference and 100 permuted networks, applies the z-test, and outputs the union
    of significant graphlets as the GGN
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Runtime environment for installing and running pyPARAGON (e.g. via python3
    -m venv pyPARAGONenv and pip install git+https://github.com/metunetlab/pyPARAGON.git)
  repo: https://github.com/metunetlab/pyPARAGON
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:9de8ffd3fbb80dd14646062e12d17d63dafe1828a824e99d5812b4118f92ba21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# reference-interactome-subnetwork-extraction

## Summary

Extract a context-relevant subnetwork from a large reference interactome using pyPARAGON's graphlet-guided network (GGN) construction, which retains only statistically significant 2-, 3-, and 4-node graphlet motifs anchored on omic-hit seeds, shrinking the network and trimming hub-driven nonspecific interactions before propagation.

## When to use

Use when given a reference interactome plus an omic-hit seed node set (e.g. phosphoproteomic hits from CPTAC breast cancer samples) and you need to prune the interactome to a seed-anchored, context-specific region before edge scoring or propagation.

## When NOT to use

- Input is already a small, curated context-specific pathway network — graphlet-based pruning is designed to shrink a large reference interactome and would add little value
- Analysis requires protein isoform or post-translational-modification-specific nodes — pyPARAGON generalizes this information and it is potentially lost in the network
- Single-cell omic data is the input — the current pyPARAGON version integrates bulk omic data for static networks only

## Inputs

- Reference interactome network (no confidence-threshold prefiltering applied)
- Omic-hit seed node set (e.g. phosphoproteomic hits from 105 CPTAC breast cancer tumors)
- Graphlet specification: nine non-isomorphic graphlets G0-G8 (2-, 3-, 4-node connected patterns)

## Outputs

- GGN subnetwork in edge-list format (ggn.edgelist)
- Union of statistically significant graphlets (p < 0.05, z-score > 1.65) as a subnetwork of the reference interactome

## How to apply

Load the reference interactome and the omic-hit seed set with pyPARAGON's graphlet module (Python). Enumerate the nine non-isomorphic graphlets (G0-G8) composed of 2-, 3-, and 4-node connected patterns over seed-anchored regions; for every graphlet except G0, assign an intermediate/hidden node in one of the highest-degree orbits. Count each targeted graphlet motif in the reference network, then count them again in 100 permuted networks that preserve the same initial seed node set. Apply a z-test comparing the reference graphlet count against the permuted-network distribution, calling a graphlet significant at p < 0.05 and z-score > 1.65. Output the union of all significant graphlets as the GGN subnetwork in edge-list format (ggn.edgelist); every retained edge must belong to a graphlet meeting both thresholds. The rationale is that graphlets filter out nonspecific interactions and mitigate the dominance of highly connected nodes, thereby trimming the reference interactome before downstream Personalized PageRank propagation and flux-based edge scoring.

## Related tools

- **pyPARAGON** (Provides the graphlet module that enumerates G0-G8 graphlets, counts motifs in reference and 100 permuted networks, applies the z-test, and outputs the union of significant graphlets as the GGN) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Runtime environment for installing and running pyPARAGON (e.g. via python3 -m venv pyPARAGONenv and pip install git+https://github.com/metunetlab/pyPARAGON.git)) — https://github.com/metunetlab/pyPARAGON

## Examples

```
python3 -m venv pyPARAGONenv && source pyPARAGONenv/bin/activate && pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- Every retained edge in the GGN belongs to a graphlet meeting both significance thresholds (p < 0.05 and z-score > 1.65)
- The GGN is a proper subnetwork of the reference interactome: its edge list (ggn.edgelist) contains no edges absent from the reference
- The GGN is exactly the union of significant graphlets — no edges from nonsignificant graphlets survive
- All 100 permuted networks preserve the same initial seed node set as the reference analysis
- The GGN is smaller than the reference interactome and excludes highly connected nodes' unrelated interactions (hub trimming is observable as reduced hub degree)

## Limitations

- Reference interactomes are incomplete, biasing inferred networks toward protein complexes over metabolic pathways
- Hub proteins may dominate the inferred network with unrelated interactions if graphlet trimming is bypassed; the flux calculation is a countermeasure against the curse of hubs beyond GGN construction
- The method cannot handle alternative copies of hits such as protein isoforms and different post-translational modifications; that information is generalized and potentially lost in the network

## Evidence

- [methods] Graphlet module of pyPARAGON identifies, an associated region of reference network through substantially frequent non-isomorphic graphlets composed of 2-, 3-, and 4-nodes: "Graphlet module of pyPARAGON identifies, an associated region of reference network through substantially frequent non-isomorphic graphlets composed of 2-, 3-, and 4-nodes"
- [readme] To uncover motifs, we counted the graphlets in 100 permuted networks that had the same initial node set. Using the z-test, we compared the targeted graphlet count in the reference and permuted networks (p < 0.05, z-score > 1.65). The union of significant graphlets constructs GGN.: "To uncover motifs, we counted the graphlets in 100 permuted networks that had the same initial node set. Using the z-test, we compared the targeted graphlet count in the reference and permuted"
- [intro] We found that graphlets filter out nonspecific interactions and mitigate the dominance of highly connected nodes, thereby trimming the reference interactome: "We found that graphlets filter out nonspecific interactions and mitigate the dominance of highly connected nodes, thereby trimming the reference interactome"
- [discussion] Thus, the reference interactome is not prefiltered based on a confidence threshold.: "Thus, the reference interactome is not prefiltered based on a confidence threshold."
