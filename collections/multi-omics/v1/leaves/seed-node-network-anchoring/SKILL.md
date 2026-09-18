---
name: seed-node-network-anchoring
description: Use when you have omic-hit seed nodes (e.g., mutated or differentially
  active proteins) and a large reference interactome, and need to shrink it to a context-relevant
  region before network propagation or edge scoring.
when_to_use_negative:
- The input is already a small, curated pathway or context-specific subnetwork — graphlet
  significance testing would add little and the reference interactome is not prefiltered
  by confidence threshold by design
- The task requires handling alternative copies of hits such as protein isoforms or
  post-translational modifications, which pyPARAGON cannot represent distinctly
- You need dynamic or single-cell network models — the current pyPARAGON version integrates
  bulk omic data for static tumor-specific networks only
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0602
tools:
- name: pyPARAGON
  role: Implements the graphlet module that enumerates G0–G8 graphlets, runs the 100-network
    permutation z-test (p < 0.05, z-score > 1.65), and outputs the GGN edge list
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Runtime environment in which pyPARAGON's graphlet module is executed (python3
    venv setup per README)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:f49316e55e23a4bfde7572776ccef9e1fa3170d0fd316c2924525239204aa79d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# seed-node-network-anchoring

## Summary

Constructs a graphlet-guided network (GGN) — a context-relevant subnetwork of a reference interactome anchored on a set of omic-hit seed nodes — by retaining only statistically significant graphlets identified via a z-test against 100 edge-swapped/permuted networks (p < 0.05, z-score > 1.65).

## When to use

Use when you have omic-hit seed nodes (e.g., mutated or differentially active proteins) and a large reference interactome, and need to shrink it to a context-relevant region before network propagation or edge scoring.

## When NOT to use

- The input is already a small, curated pathway or context-specific subnetwork — graphlet significance testing would add little and the reference interactome is not prefiltered by confidence threshold by design
- The task requires handling alternative copies of hits such as protein isoforms or post-translational modifications, which pyPARAGON cannot represent distinctly
- You need dynamic or single-cell network models — the current pyPARAGON version integrates bulk omic data for static tumor-specific networks only

## Inputs

- Reference interactome (network edge list)
- Omic-hit seed node set (e.g., from omics data such as the CPTAC breast cancer phosphoproteomic dataset of 105 tumors)
- Permutation parameters (100 permuted networks preserving the seed node set)

## Outputs

- Graphlet-guided network GGN, G(VG, EG), as a subnetwork of the reference interactome
- ggn.edgelist — the GGN in edge-list format

## How to apply

Load the reference interactome and omic-hit seed node set with pyPARAGON's graphlet module in Python. Enumerate the nine non-isomorphic graphlets (G0–G8) composed of 2-, 3-, and 4-node connected patterns over seed-anchored regions; for graphlets larger than 2 nodes (all except G0), assign an intermediate/hidden node in one of the highest-degree orbits. Count each targeted graphlet motif in the reference network, then count graphlets in 100 permuted networks that preserve the same initial seed node set. Apply a z-test comparing the reference graphlet count against the permuted-network distribution, calling a graphlet significant at p < 0.05 and z-score > 1.65. Output the union of all significant graphlets as G(VG, EG), the GGN subnetwork in edge-list format (ggn.edgelist); this shrinks the network and eliminates highly connected nodes and their unrelated interactions.

## Related tools

- **pyPARAGON** (Implements the graphlet module that enumerates G0–G8 graphlets, runs the 100-network permutation z-test (p < 0.05, z-score > 1.65), and outputs the GGN edge list) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Runtime environment in which pyPARAGON's graphlet module is executed (python3 venv setup per README))

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- Every retained edge in the GGN belongs to a graphlet meeting the significance threshold p < 0.05 and z-score > 1.65
- The GGN equals the union of significant graphlets G(VG, EG) and is a strict subnetwork of the reference interactome
- All 100 permuted networks preserve the same initial seed node set as the reference analysis
- Exactly nine non-isomorphic graphlets (G0–G8) with 2, 3, and 4 nodes were enumerated, and each graphlet larger than G0 includes a hidden node in one of the highest-degree orbits
- Output file ggn.edgelist exists and parses as a valid edge list

## Limitations

- Hub proteins may dominate the inferred network with unrelated interactions if graphlet filtering is skipped or fails
- Incomplete knowledge in large reference interactomes biases inferred networks toward protein complexes over metabolic pathways
- Inferred networks frequently contain generic biological processes such as transcription and replication, hindering interpretation
- Alternative copies of individual hits (protein isoforms, different post-translational modifications) cannot be handled and are generalized/lost in the network

## Evidence

- [methods] G0–G8 graphlet enumeration: "Graphlet module of pyPARAGON identifies, an associated region of reference network through substantially frequent non-isomorphic graphlets composed of 2-, 3-, and 4-nodes"
- [readme] GGN significance testing: "we counted the graphlets in 100 permuted networks that had the same initial node set. Using the z-test, we compared the targeted graphlet count in the reference and permuted networks (p < 0.05,"
- [intro] Graphlets trim hubs: "We found that graphlets filter out nonspecific interactions and mitigate the dominance of highly connected nodes, thereby trimming the reference interactome"
- [readme] Hidden node assignment: "Except for G0, each graphlet includes initial nodes (red circles) and a hidden node (white circles) in one of the highest degrees of orbits."
- [intro] Motivation for network motifs: "the utilization of network motifs, in lieu of pairwise connections among proteins, may provide a more accurate representation of signaling networks"
