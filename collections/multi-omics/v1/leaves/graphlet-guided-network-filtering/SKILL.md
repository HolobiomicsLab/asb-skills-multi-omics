---
name: graphlet-guided-network-filtering
description: Use when integrating omic-hits (e.g., phosphoproteomic seed proteins)
  with a large reference interactome to infer a context-specific subnetwork, and nonspecific
  hub-dominated pairwise interactions must be filtered out.
when_to_use_negative:
- Input is already a small curated pathway rather than a large reference interactome
  with hub-dominated nonspecific interactions — the graphlet trimming step offers
  no benefit
- The omic data are single-cell or require explicit modeling of protein isoforms and
  post-translational modifications, which the current bulk-data, node-collapsed pyPARAGON
  version cannot represent
- No omic-hits seed node set is available — graphlet significance testing and PPR
  flux scoring both require seed nodes
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0092
tools:
- name: pyPARAGON
  role: 'Implements the full filtering pipeline: GGN construction, PPR-based edge
    scoring with flux calculation, and tau-threshold subnetwork reconstruction'
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Runtime environment (virtualenv-based installation; pyPARAGON is a pip-installable
    Python package using NetworkX for graph serialization)
  repo: https://github.com/metunetlab/pyPARAGON
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:70cf0315e3fdaeaddadc105dbe0a4dc0c004287c8831436c6c7556e56af984ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# graphlet-guided-network-filtering

## Summary

pyPARAGON's network-inference filtering strategy: it shrinks a reference interactome to a Graphlet-guided Network (GGN) built from statistically enriched 2-, 3-, and 4-node graphlets, then ranks GGN edges by PageRank-flux scores and retains the top edges until cumulative flux reaches tau × total flux F to reconstruct a context-specific subnetwork.

## When to use

Use when integrating omic-hits (e.g., phosphoproteomic seed proteins) with a large reference interactome to infer a context-specific subnetwork, and nonspecific hub-dominated pairwise interactions must be filtered out.

## When NOT to use

- Input is already a small curated pathway rather than a large reference interactome with hub-dominated nonspecific interactions — the graphlet trimming step offers no benefit
- The omic data are single-cell or require explicit modeling of protein isoforms and post-translational modifications, which the current bulk-data, node-collapsed pyPARAGON version cannot represent
- No omic-hits seed node set is available — graphlet significance testing and PPR flux scoring both require seed nodes

## Inputs

- Reference interactome (network with edge confidence scores)
- Omic-hits seed node set (e.g., phosphoproteomic hits from breast cancer tumors)
- Graphlet count parameters: 100 permuted networks, p < 0.05, z-score > 1.65
- Scaling factor tau for the flux threshold (tau × total flux F)
- Flux-scored GGN edge list produced by the PPR + flux-calculation step

## Outputs

- Graphlet-guided Network (GGN) — union of significant graphlet motifs
- Context-specific subnetwork C(VC, EC) serialized as context_specific_network.graphml (GraphML via NetworkX), with EC ⊆ EG and VC ⊆ VG

## How to apply

First construct the GGN: scan the reference network for substantially frequent non-isomorphic graphlets composed of 2-, 3-, and 4-nodes (nine graphlets G0–G8, each with a hidden node in one of the highest-degree orbits), counting graphlets in 100 permuted networks with the same initial node set and keeping motifs significant at p < 0.05 with z-score > 1.65 by z-test; the union of significant graphlets constructs the GGN, which trims the interactome and mitigates hub dominance. Next, propagate signals from the omic-hits seed set with the Personalized PageRank (PPR) algorithm and weight each GGN edge by a flux calculation that considers node PageRank scores, edge confidence scores, and the number of node interactions. Finally, rank GGN edges in descending flux score, accumulate flux from the top edge downward until the cumulative flux reaches the tau-scaled threshold (tau × F, Eqs. 5-6), and retain the selected edges (EC) and their incident nodes (VC) as the context-specific network C(VC, EC), serialized with NetworkX to context_specific_network.graphml.

## Related tools

- **pyPARAGON** (Implements the full filtering pipeline: GGN construction, PPR-based edge scoring with flux calculation, and tau-threshold subnetwork reconstruction) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Runtime environment (virtualenv-based installation; pyPARAGON is a pip-installable Python package using NetworkX for graph serialization)) — https://github.com/metunetlab/pyPARAGON

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- context_specific_network.graphml exists and parses as a valid GraphML graph file
- The output edge set EC is a subset of the input GGN edge set (EC ⊆ EG) and node set VC ⊆ VG
- The cumulative flux score of retained edges satisfies the tau × F threshold (Eqs. 5-6)
- Retained graphlet motifs were significant against 100 permuted networks at p < 0.05 with z-score > 1.65
- Hub-related unrelated interactions are reduced relative to the full reference interactome, reflecting graphlet-based trimming

## Limitations

- Reference interactomes are incomplete, so inferred networks are biased toward protein complexes over metabolic pathways
- Hub proteins may still dominate the inferred network with unrelated interactions despite flux-based countering
- The tool cannot handle alternative copies of hits such as protein isoforms and post-translational modifications; that information is generalized and potentially lost in the network
- No software version is stated for pyPARAGON, complicating reproducibility

## Evidence

- [readme] Graphlet significance testing: "Using the z-test, we compared the targeted graphlet count in the reference and permuted networks (p < 0.05, z-score > 1.65). The union of significant graphlets constructs GGN."
- [readme] Graphlet module identifies frequent graphlets: "Graphlet module of pyPARAGON identifies, an associated region of reference network through substantially frequent non-isomorphic graphlets composed of 2-, 3-, and 4-nodes"
- [readme] Flux calculation weights GGN edges: "the flux calculation weights each edge of the GGN, considering the PageRank scores of nodes, the confidence scores of edges, and the number of node interactions"
- [methods] Subnetwork reconstruction step: "Finally, pyPARAGON infers the context specific network by selecting the highly scored edges."
- [intro] Graphlets filter nonspecific interactions: "We found that graphlets filter out nonspecific interactions and mitigate the dominance of highly connected nodes, thereby trimming the reference interactome"
- [discussion] Flux-based edge elimination: "pyPARAGON eliminates the interactions based on the calculated edge fluxes."
