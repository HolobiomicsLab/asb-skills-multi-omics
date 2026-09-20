---
name: personalized-pagerank-network-propagation
description: Use when a weighted set of omic hit proteins must be propagated over
  a reference interactome to re-score all proteins and prioritize edges by directional
  PageRank-flux before selecting a context-specific subnetwork.
when_to_use_negative:
- Input contains protein isoforms or post-translational modifications as alternative
  copies of hits — the tool generalizes this information and it is potentially lost
  in the network
- The goal is a dynamic or single-cell network model — pyPARAGON integrates bulk omic
  data for static tumor-specific networks; single-cell support is only planned in
  a future version
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3557
tools:
- name: pyPARAGON
  role: Runs Personalized PageRank propagation from the omic-hits seed set and performs
    the flux calculation that weights each GGN edge
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Runtime environment in which pyPARAGON is installed and executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:aae2745663ab223629f4312c22ab2f0f9a5e23304a35fccd74fc6ae6af38fa8f
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
  - http://edamontology.org/topic_3557
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# personalized-pagerank-network-propagation

## Summary

Propagates a weighted seed-node set (omic-hits) over a reference interactome with the Personalized PageRank algorithm and then scores each GGN edge via PageRank-flux, the mechanism pyPARAGON uses before subnetwork reconstruction.

## When to use

Use when a weighted set of omic hit proteins must be propagated over a reference interactome to re-score all proteins and prioritize edges by directional PageRank-flux before selecting a context-specific subnetwork.

## When NOT to use

- Input contains protein isoforms or post-translational modifications as alternative copies of hits — the tool generalizes this information and it is potentially lost in the network
- The goal is a dynamic or single-cell network model — pyPARAGON integrates bulk omic data for static tumor-specific networks; single-cell support is only planned in a future version

## Inputs

- Reference interactome with edge confidence scores
- Weighted seed-node set of omic-hits (e.g. phosphoproteomic hits from CPTAC breast cancer samples)
- Graphlet-guided network (GGN) edges produced by the graphlet module

## Outputs

- PPR priority scores for every node in the reference network
- Directional fluxes f(u->t) and f(t->u) per GGN edge
- Combined (minimum-flux, negative log) edge score per GGN edge, written as ppr_flux_edge_scores.csv

## How to apply

Load the reference interactome with per-edge confidence scores and the weighted omic-hits seed set, and run Personalized PageRank by random walking from the weighted initial nodes so every protein in the reference network is re-scored. For each edge of the graphlet-guided network (GGN), compute the two directed fluxes f(u->t) = p(u)*c(e)/deg(u) and f(t->u) = p(t)*c(e)/deg(t) using the PPR node scores, edge confidence scores, and node interaction counts (degree). Combine the two directional fluxes per edge into a single edge score by taking the minimum directional flux (Eqs. 1-4), rather than multiplying them. The degree-normalization term acts as a countermeasure against the curse of hubs, so highly connected proteins do not dominate the inferred network. Note that the reference interactome is not prefiltered by a confidence threshold, and the propagation runs independently of the graphlet module.

## Related tools

- **pyPARAGON** (Runs Personalized PageRank propagation from the omic-hits seed set and performs the flux calculation that weights each GGN edge) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Runtime environment in which pyPARAGON is installed and executed)

## Examples

```
python3 -m venv pyPARAGONenv; source venv/bin/activate; pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- ppr_flux_edge_scores.csv covers every GGN edge with finite combined flux scores derived from the PPR scores, edge confidences, and degrees
- Combined edge score equals the negative log of the minimum directional flux, not a product of the two directional fluxes
- Each directional flux equals p(node)*c(edge)/deg(node) with the PPR score, edge confidence, and degree of the source node
- PPR scores are assigned to every protein in the reference network via random walking from weighted initial nodes
- Subsequently selecting the most highly scored edges should yield a context-specific subnetwork whose hub dominance is mitigated

## Limitations

- Reference interactomes are incomplete, biasing inferred networks toward protein complexes over metabolic pathways
- Hub proteins may dominate the inferred network with unrelated interactions; the flux calculation is a countermeasure beyond hub penalization
- Inferred networks frequently contain generic biological processes such as transcription and replication, hindering interpretation
- The tool cannot handle alternative copies of hits such as protein isoforms and different post-translational modifications
- No software version is stated for pyPARAGON

## Evidence

- [methods] Propagation and edge scoring via the Personalized PageRank (PPR) algorithm and flux calculation: "ii. Propagation and edge scoring via  the Personalized PageRank (PPR) algorithm and flux calculation,"
- [readme] PPR propagation then flux weighting: "pyPARAGON propagates signals from omic-hits through the personalized PageRank algorithm in re-scoring all proteins in the reference (Figure 2). Then, the flux calculation weights each edge of the GGN"
- [readme] Flux inputs: PageRank, confidence, degree: "the flux calculation weights each edge of the GGN, considering the PageRank scores of nodes, the confidence scores of edges, and the number of node interactions."
- [discussion] Propagation independent of GGN: "the PageRank algorithm in pyPARAGON propagates the seed nodes before network inference, independent of GGN"
- [discussion] No confidence prefiltering: "Thus, the reference interactome is not prefiltered based on a confidence threshold."
- [discussion] Flux as hub countermeasure: "the flux calculation in pyPARAGON is a countermeasure against the curse of hubs beyond"
