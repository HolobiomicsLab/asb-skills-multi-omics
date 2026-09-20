---
name: context-specific-network-inference
description: Use when a seed set of omic-hits (e.g., mutated or phosphoproteomic hits
  from 105 CPTAC breast cancer tumors) must be converted into a context-specific subnetwork
  over a reference interactome such as HIPPIE or ConsensusPathDB.
when_to_use_negative:
- Input is single-cell omic data — pyPARAGON's current version integrates bulk omic
  data for static tumor-specific networks only; single-cell support is planned for
  the next version
- Hits include protein isoforms or site-specific post-translational modifications
  that must be preserved — this information is generalized and potentially lost in
  the network
- The goal is reconstructing metabolic pathways from an incomplete interactome — inferred
  networks are biased toward protein complexes, which form more topological modules
  than metabolic pathways
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: pyPARAGON
  role: Core tool executing GGN construction, PPR propagation, flux-based edge scoring,
    and subnetwork reconstruction
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Implementation/execution environment (venv setup, pyPARAGONenv; NetworkX used
    for graph serialization)
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:e1c6caaeeed1e29e34649565ffcd85f3d84afa51b0281638746780c5b8ac6e15
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
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# context-specific-network-inference

## Summary

Infers a context-specific signaling subnetwork from a large reference interactome using pyPARAGON, which combines graphlet-guided network (GGN) construction, Personalized PageRank (PPR) propagation, and PageRank-flux edge scoring to select highly scored edges. This hybrid approach trims nonspecific interactions and hub dominance while retaining network-level (not pairwise) connections.

## When to use

Use when a seed set of omic-hits (e.g., mutated or phosphoproteomic hits from 105 CPTAC breast cancer tumors) must be converted into a context-specific subnetwork over a reference interactome such as HIPPIE or ConsensusPathDB.

## When NOT to use

- Input is single-cell omic data — pyPARAGON's current version integrates bulk omic data for static tumor-specific networks only; single-cell support is planned for the next version
- Hits include protein isoforms or site-specific post-translational modifications that must be preserved — this information is generalized and potentially lost in the network
- The goal is reconstructing metabolic pathways from an incomplete interactome — inferred networks are biased toward protein complexes, which form more topological modules than metabolic pathways

## Inputs

- Reference interactome (e.g., HIPPIE v2.2/v2.3 or ConsensusPathDB) with edge confidence scores
- Seed node set of omic-hits (e.g., mutated genes, phosphoproteomic hits from CPTAC breast cancer tumors)
- Graphlet-guided network (GGN) edge list with flux scores
- Scaling factor tau for the flux-threshold criterion

## Outputs

- Context-specific subnetwork C(VC, EC) where EC is a subset of GGN edges and VC the incident nodes
- Serialized network file (context_specific_network.graphml)
- Optional Louvain communities with hypergeometric overrepresentation of GO/KEGG annotations

## How to apply

Run pyPARAGON's three-step inference: (i) build the GGN from nine non-isomorphic graphlets of 2, 3, and 4 nodes, counting motifs against 100 permuted networks with a z-test (p < 0.05, z-score > 1.65) and taking the union of significant graphlets; (ii) propagate seed omic-hits with the Personalized PageRank algorithm independently of the GGN, then compute a flux for each GGN edge using node PageRank scores, edge confidence scores, and node interaction counts; (iii) rank GGN edges by descending flux and accumulate scores until the cumulative flux reaches the tau-scaled total threshold (tau x F, Eqs. 5-6), retaining the selected edges EC and incident nodes VC as C(VC, EC). The rationale is that graphlets filter nonspecific interactions and counter the 'curse of hubs' without a confidence-threshold prefilter. Serialize the result (e.g., context_specific_network.graphml via pyPARAGON/NetworkX) and optionally feed it to the Louvain community-detection module for hypergeometric overrepresentation of biological processes or pathways.

## Related tools

- **pyPARAGON** (Core tool executing GGN construction, PPR propagation, flux-based edge scoring, and subnetwork reconstruction) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Implementation/execution environment (venv setup, pyPARAGONenv; NetworkX used for graph serialization))

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- context_specific_network.graphml exists and parses as a valid graph file
- The output edge set EC is a subset of the input GGN edge set EG, and VC is a subset of VG
- The cumulative flux score of retained edges meets the tau x F threshold criterion (Eqs. 5-6)
- The inferred network is smaller than the reference interactome, with highly connected hubs and their unrelated interactions eliminated
- Against curated NetPath gold-standard node/edge sets, node-level and edge-level AUPRC/F1 exceed those of PathLinker, Omics Integrator 2, and DOMINO

## Limitations

- Hub proteins may dominate the inferred network with unrelated interactions; the flux calculation is a countermeasure, but hub effects are a known failure mode
- The reference interactome is not prefiltered by confidence threshold, and incomplete interactome knowledge biases inferred networks toward protein complexes over metabolic pathways; generic processes such as transcription and replication can appear frequently
- No software version is stated for pyPARAGON, complicating reproducibility

## Evidence

- [methods] Finally, pyPARAGON infers the context specific network by selecting the highly scored edges.: "Finally, pyPARAGON infers the context specific network by selecting the highly scored edges."
- [readme] the flux calculation weights each edge of the GGN, considering the PageRank scores of nodes, the confidence scores of edges, and the number of node interactions.: "the flux calculation weights each edge of the GGN, considering the PageRank scores of nodes, the confidence scores of edges, and the number of node interactions."
- [discussion] pyPARAGON eliminates the interactions based on the calculated edge fluxes.: "pyPARAGON eliminates the interactions based on the calculated edge fluxes."
- [intro] We found that graphlets filter out nonspecific interactions and mitigate the dominance of highly connected nodes, thereby trimming the reference interactome: "We found that graphlets filter out nonspecific interactions and mitigate the dominance of highly connected nodes, thereby trimming the reference interactome"
