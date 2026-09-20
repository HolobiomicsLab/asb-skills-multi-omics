---
name: edge-flux-prioritization
description: Use when you have a flux-scored GGN from the Personalized PageRank and
  flux-calculation step and must reconstruct a context-specific subnetwork by thresholding
  edges with the tau x F criterion.
when_to_use_negative:
- The input is a raw reference interactome without GGN construction and PPR-based
  flux scoring, since edge selection requires flux-weighted GGN edges
- The goal is module/annotation analysis only — the community analysis module (Louvain
  + hypergeometric test) can run independently of this network inference step
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0602
tools:
- name: pyPARAGON
  role: Performs the flux-scored edge ranking, tau x F threshold selection, and reconstruction
    of the context-specific subnetwork C(VC, EC)
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Runtime environment for pyPARAGON (e.g. python3 -m venv pyPARAGONenv) used
    to serialize the subnetwork with NetworkX
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:d339c6f4c470c5c720bdf4d4292b93f5a4687f9a317faeecfd4d5d71d01d70e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Edge flux prioritization

## Summary

Ranks the edges of pyPARAGON's graphlet-guided network (GGN) by their PageRank-flux scores and selects the top-ranked edges until the cumulative flux reaches the tau-scaled total flux threshold (tau x F), yielding the context-specific subnetwork C(VC, EC).

## When to use

Use when you have a flux-scored GGN from the Personalized PageRank and flux-calculation step and must reconstruct a context-specific subnetwork by thresholding edges with the tau x F criterion.

## When NOT to use

- The input is a raw reference interactome without GGN construction and PPR-based flux scoring, since edge selection requires flux-weighted GGN edges
- The goal is module/annotation analysis only — the community analysis module (Louvain + hypergeometric test) can run independently of this network inference step

## Inputs

- Flux-scored edge list of the graphlet-guided network (GGN)
- GGN produced by the Personalized PageRank and flux-calculation step
- Scaling factor tau and total flux F

## Outputs

- Context-specific subnetwork C(VC, EC) serialized as context_specific_network.graphml

## How to apply

Load the flux-scored edge list and the GGN produced by the graphlet-guided network construction and Personalized PageRank (PPR) flux-calculation steps. Rank all GGN edges in descending order of their calculated flux scores, where flux weights each edge using node PageRank scores, edge confidence scores, and the number of node interactions. Accumulate edge fluxes from the top-ranked edge downward until the cumulative flux reaches the tau-scaled total flux threshold (tau x F, Eqs. 5-6). Retain the selected edges (EC) and their incident nodes (VC) as EC subset of EG and VC subset of VG to infer the context-specific network C(VC, EC), then serialize the result to context_specific_network.graphml using pyPARAGON/NetworkX.

## Related tools

- **pyPARAGON** (Performs the flux-scored edge ranking, tau x F threshold selection, and reconstruction of the context-specific subnetwork C(VC, EC)) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Runtime environment for pyPARAGON (e.g. python3 -m venv pyPARAGONenv) used to serialize the subnetwork with NetworkX)

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- context_specific_network.graphml exists and parses as a valid graph file via pyPARAGON/NetworkX
- The output edge set EC is a subset of the input GGN edge set EG
- The cumulative flux score of the selected edges satisfies the tau x F criterion (Eqs. 5-6)
- All nodes in the reconstructed network are incident to at least one selected edge (VC is the incident node set of EC)
- Highly scored (top-ranked) GGN edges are preferentially retained over low-flux edges

## Limitations

- Hub proteins may dominate the inferred network with unrelated interactions; flux calculation is a countermeasure against the curse of hubs but does not eliminate the risk
- Inferred networks frequently contain generic biological processes (e.g. transcription and replication), hindering interpretation
- The reference interactome is not prefiltered based on a confidence threshold, and incomplete interactome knowledge biases networks toward protein complexes over metabolic pathways

## Evidence

- [methods] Subnetwork reconstruction with highly scored edges on GGN.: "iii. Subnetwork reconstruction with highly scored edges on GGN."
- [methods] Finally, pyPARAGON infers the context specific network by selecting the highly scored edges.: "Finally, pyPARAGON infers the context specific network by selecting the highly scored edges."
- [other] pyPARAGON ranks GGN edges by flux score, sums the scores up to a threshold of tau x total flux F: [evidence span withheld — anchor status: not_found]
- [readme] The highly weighted edges reported in GGN construct a subnetwork or specific network.: "The highly weighted edges reported in GGN construct a subnetwork or specific network."
- [discussion] pyPARAGON eliminates the interactions based on the calculated edge fluxes.: "pyPARAGON eliminates the interactions based on the calculated edge fluxes."
