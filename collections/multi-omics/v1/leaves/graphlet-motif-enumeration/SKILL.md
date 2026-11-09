---
name: graphlet-motif-enumeration
description: Use when a seed node set of omic-hits must be expanded into a context-specific
  subnetwork from a large reference interactome, and nonspecific interactions/hub
  bias should be filtered topologically before propagation.
when_to_use_negative:
- The input is already a small, curated context-specific network where topological
  filtering would discard true interactions
- The analysis requires isoform- or PTM-specific hits, which pyPARAGON cannot handle
  as alternative copies of individual proteins
- Single-cell omic data are the input; the current pyPARAGON version integrates bulk
  omic data only
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0128
tools:
- name: pyPARAGON
  role: Provides the graphlet module that enumerates G0–G8 graphlets, counts motifs,
    runs the 100-permutation z-test, and outputs the GGN edge list
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Runtime environment for pyPARAGON's graphlet module (installed via pip into
    a pyPARAGONenv virtual environment)
  repo: https://github.com/metunetlab/pyPARAGON
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:6bc874cf3ab73c27d784e971aeb6be291a33ea68dc95acdc30ad9a5dfefaca72
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# graphlet-motif-enumeration

## Summary

Enumerates nine non-isomorphic graphlets (G0–G8) of 2-, 3-, and 4-node connected patterns in a reference interactome and, by z-testing counts against 100 permuted networks, retains only statistically significant graphlets whose union defines a context-relevant subnetwork (GGN). This trims the interactome and counters hub dominance before network propagation.

## When to use

Use when a seed node set of omic-hits must be expanded into a context-specific subnetwork from a large reference interactome, and nonspecific interactions/hub bias should be filtered topologically before propagation.

## When NOT to use

- The input is already a small, curated context-specific network where topological filtering would discard true interactions
- The analysis requires isoform- or PTM-specific hits, which pyPARAGON cannot handle as alternative copies of individual proteins
- Single-cell omic data are the input; the current pyPARAGON version integrates bulk omic data only

## Inputs

- Reference interactome (network with protein nodes and confidence-scored edges)
- Seed node set of omic-hits (e.g. from bulk omic/phosphoproteomic data such as the 105 CPTAC breast cancer tumors)

## Outputs

- Graphlet-guided network GGN G(VG, EG) as an edge-list file (ggn.edgelist), containing only edges belonging to significant graphlets

## How to apply

Load the reference interactome and the omic-hit seed node set with pyPARAGON's graphlet module in Python. Enumerate the nine non-isomorphic graphlets (G0–G8) composed of 2-, 3-, and 4-node connected patterns over seed-anchored regions; for every graphlet except G0, assign an intermediate/hidden node in one of the highest-degree orbits (the most interacting node). Count each targeted graphlet motif in the reference network, then generate 100 permuted networks preserving the same initial seed node set and count graphlets in each permutation. Apply a z-test comparing the reference count against the permuted-network distribution, calling a graphlet significant at p < 0.05 and z-score > 1.65. Output the union of all significant graphlets as the GGN subnetwork in edge-list format (ggn.edgelist); this shrinks the network and eliminates highly connected nodes and their unrelated interactions.

## Related tools

- **pyPARAGON** (Provides the graphlet module that enumerates G0–G8 graphlets, counts motifs, runs the 100-permutation z-test, and outputs the GGN edge list) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Runtime environment for pyPARAGON's graphlet module (installed via pip into a pyPARAGONenv virtual environment)) — https://github.com/metunetlab/pyPARAGON

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git; python -m pyPARAGON.graphlet --interactome ref.sif --seeds hits.txt --permutations 100 --out ggn.edgelist
```

## Evaluation signals

- Every retained edge in the GGN belongs to a graphlet meeting the significance thresholds p < 0.05 and z-score > 1.65
- The GGN is exactly the union of the significant graphlets — no edges from nonsignificant graphlets survive
- GGN node/edge counts are substantially smaller than the reference interactome (network shrunk to a seed-associated region)
- Permutation count is exactly 100, and permuted networks preserve the same initial seed node set
- Graphlets with >2 nodes each contain exactly one intermediate/hidden node chosen from a highest-degree orbit

## Limitations

- The z-test thresholds (p < 0.05, z-score > 1.65, 100 permutations) are fixed defaults, not user-tunable per the described workflow
- The GGN biases inferred networks toward protein complexes rather than metabolic pathways because reference interactomes are incomplete and complexes form more topological modules
- Hub proteins may still dominate inferred networks with unrelated interactions despite graphlet trimming; flux calculation is the countermeasure beyond the GGN

## Evidence

- [readme] Figure 1B graphlet enumeration: [evidence span withheld — anchor status: not_found]
- [readme] README z-test thresholds: "To uncover motifs, we counted the graphlets in 100 permuted networks that had the same initial node set. Using the z-test, we compared the targeted graphlet count in the reference and permuted"
- [methods] Methods GGN rationale: "Graphlet module of pyPARAGON identifies, an associated region of reference network through substantially frequent non-isomorphic graphlets composed of 2-, 3-, and 4-nodes"
- [intro] Intro graphlet filtering claim: "We found that graphlets filter out nonspecific interactions and mitigate the dominance of highly connected nodes, thereby trimming the reference interactome"
- [discussion] Discussion limitation on complexes: "As a result of incomplete knowledge in large reference interactomes, protein complexes tend to form more topological modules than metabolic pathways"
