---
name: subnetwork-reconstruction-from-scored-edges
description: Use when a flux-scored edge list from a Personalized PageRank + flux-calculation
  step is available and a context-specific subnetwork must be reconstructed by selecting
  highly scored edges under a tau x F threshold.
when_to_use_negative:
- No flux-scored edges exist yet — the GGN must first be built and edges scored by
  the PPR + flux-calculation step before reconstruction
- The goal is sample-specific network inference per individual (e.g. SWEET-style)
  rather than one context-specific network from aggregated omic-hits
- Community/module detection is the goal — run the Louvain-based community analysis
  module on the reconstructed network instead
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0128
tools:
- name: pyPARAGON
  role: 'performs the subnetwork reconstruction step: ranks GGN edges by flux score,
    applies the tau x F threshold, and serializes the context-specific network'
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: runtime environment for pyPARAGON (e.g. python3 -m venv pyPARAGONenv); NetworkX
    is used to write the GraphML output
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:f2c49de7d6b7678f49d68006e89e4b09a88ed5f5715554cc96c27c6bcbfcd8aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# subnetwork-reconstruction-from-scored-edges

## Summary

Reconstructs a context-specific subnetwork C(VC, EC) by ranking flux-scored edges of the graphlet-guided network (GGN) and retaining edges whose cumulative flux reaches the tau-scaled total flux threshold (tau x F), as implemented in pyPARAGON's third network-inference step.

## When to use

Use when a flux-scored edge list from a Personalized PageRank + flux-calculation step is available and a context-specific subnetwork must be reconstructed by selecting highly scored edges under a tau x F threshold.

## When NOT to use

- No flux-scored edges exist yet — the GGN must first be built and edges scored by the PPR + flux-calculation step before reconstruction
- The goal is sample-specific network inference per individual (e.g. SWEET-style) rather than one context-specific network from aggregated omic-hits
- Community/module detection is the goal — run the Louvain-based community analysis module on the reconstructed network instead

## Inputs

- flux-scored edge list from pyPARAGON's PPR + flux calculation step (edge, flux score pairs)
- graphlet-guided network (GGN) with edges EG and nodes VG
- scaling factor tau (threshold parameter)

## Outputs

- context-specific network C(VC, EC) with EC subset of EG and VC subset of VG
- context_specific_network.graphml (GraphML file written via pyPARAGON/NetworkX)

## How to apply

Load the flux-scored edge list together with the GGN produced by pyPARAGON's Personalized PageRank and flux-calculation step, in which each GGN edge was weighted considering PageRank scores of nodes, edge confidence scores, and the number of node interactions. Rank all GGN edges in descending order of flux score and accumulate fluxes from the top-ranked edge downward until the cumulative flux reaches tau x F, the tau-scaled total flux F (Eqs. 5-6). Retain the selected edges EC (a subset of the GGN edge set EG) and their incident nodes VC (a subset of VG) to form the context-specific network C(VC, EC). Serialize the result as context_specific_network.graphml using pyPARAGON/NetworkX. The rationale is that selecting highly scored edges on the trimmed GGN eliminates hub-dominated, nonspecific interactions that a raw interactome would retain.

## Related tools

- **pyPARAGON** (performs the subnetwork reconstruction step: ranks GGN edges by flux score, applies the tau x F threshold, and serializes the context-specific network) — https://github.com/netlab-ku/pyPARAGON
- **Python** (runtime environment for pyPARAGON (e.g. python3 -m venv pyPARAGONenv); NetworkX is used to write the GraphML output)

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- context_specific_network.graphml exists and parses as a valid graph file
- The edge set of the output graph is a subset of the input GGN edge set EG (EC ⊆ EG)
- The cumulative flux score of retained edges meets or exceeds tau x F, and dropping the lowest-ranked retained edge would fall below the threshold
- All nodes in the output (VC) are incident to at least one retained edge and are drawn from the GGN node set VG
- Edge count of the output is substantially smaller than the reference interactome, consistent with GGN trimming of hub-dominated, nonspecific interactions

## Limitations

- The reference interactome is not prefiltered by confidence threshold, so edge confidence enters only via the flux calculation
- Hub proteins may dominate the inferred network with unrelated interactions — the flux calculation is a countermeasure, but the tau threshold must be chosen carefully
- Inferred networks frequently contain generic biological processes (e.g. transcription and replication), hindering interpretation
- No software version is stated for pyPARAGON in the article

## Evidence

- [methods] iii. Subnetwork reconstruction with highly scored edges on GGN.: "iii. Subnetwork reconstruction with highly scored edges on GGN."
- [readme] flux calculation weights each edge of the GGN considering PageRank scores, confidence scores, and node interactions: "the flux calculation weights each edge of the GGN, considering the PageRank scores of nodes, the confidence scores of edges, and the number of node interactions."
- [methods] pyPARAGON infers the context specific network by selecting the highly scored edges: "Finally, pyPARAGON infers the context specific network by selecting the highly scored edges."
- [discussion] pyPARAGON eliminates the interactions based on the calculated edge fluxes: "pyPARAGON eliminates the interactions based on the calculated edge fluxes."
- [readme] The highly weighted edges reported in GGN construct a subnetwork or specific network: "The highly weighted edges reported in GGN construct a subnetwork or specific network."
