---
name: protein-interaction-network-scoring
description: Use when a weighted seed-node set (omic-hits) must be propagated over
  a reference interactome via Personalized PageRank and edges prioritized by flux
  scores for context-specific subnetwork reconstruction.
when_to_use_negative:
- When the goal is constructing sample-specific networks via network inference methods
  like SWEET rather than seed-propagation-based edge prioritization
- When the reference interactome has already been prefiltered by a confidence threshold
  — the article states the reference interactome is not prefiltered based on a confidence
  threshold
- When omic hits include alternative copies such as protein isoforms or post-translational
  modifications, which pyPARAGON cannot handle
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0602
tools:
- name: pyPARAGON
  role: Runs the Personalized PageRank propagation over the reference network and
    the flux calculation weighting each GGN edge by PPR scores, edge confidences,
    and node interaction counts
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Runtime environment for installing and executing pyPARAGON (virtualenv, wheel-based
    install)
  repo: https://github.com/metunetlab/pyPARAGON
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:7bb4bd840dfaacf9d39ae3ae1fce94fdd37e1633e7039507955fe57181322af7
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
yes_rate: 0.7692307692307693
n_claims: 13
---

# protein-interaction-network-scoring

## Summary

Reconstructs pyPARAGON's Personalized PageRank (PPR) propagation and PageRank-flux edge scoring: seed omic-hits are propagated over a reference interactome, then each GGN edge is scored by combining the two directional fluxes f(u→t) = p(u)·c(e)/deg(u) and f(t→u) = p(t)·c(e)/deg(t) by taking the minimum directional flux.

## When to use

Use when a weighted seed-node set (omic-hits) must be propagated over a reference interactome via Personalized PageRank and edges prioritized by flux scores for context-specific subnetwork reconstruction.

## When NOT to use

- When the goal is constructing sample-specific networks via network inference methods like SWEET rather than seed-propagation-based edge prioritization
- When the reference interactome has already been prefiltered by a confidence threshold — the article states the reference interactome is not prefiltered based on a confidence threshold
- When omic hits include alternative copies such as protein isoforms or post-translational modifications, which pyPARAGON cannot handle

## Inputs

- Reference interactome network with edge confidence scores
- Weighted seed-node set of omic-hits
- Graphlet-guided network (GGN) edges produced by the graphlet module

## Outputs

- PPR scores for every protein in the reference network
- Per-edge directional fluxes f(u→t) and f(t→u) for each GGN edge
- Combined edge flux scores (negative log of minimum directional flux), e.g., written to ppr_flux_edge_scores.csv

## How to apply

Load the reference interactome with edge confidence scores and the weighted seed-node (omic-hits) set using pyPARAGON's Python utilities. Run Personalized PageRank with random walking from the weighted initial nodes so every protein in the reference network receives a PPR score; this propagation runs independent of the graphlet module. For each edge of the graphlet-guided network (GGN), compute the directed fluxes f(u→t) = p(u)·c(e)/deg(u) and f(t→u) = p(t)·c(e)/deg(t), using the PPR node scores p(·), edge confidence scores c(e), and node degrees (number of node interactions). Combine the two directional fluxes into a single edge score via the negative log of the minimum directional flux (Eqs. 1-4) — the minimum, not a product, is the paper's stated combination rule. Write all GGN edges with their PPR scores, directional fluxes, and combined scores (e.g., to ppr_flux_edge_scores.csv); downstream subnetwork reconstruction then selects the highly scored edges.

## Related tools

- **pyPARAGON** (Runs the Personalized PageRank propagation over the reference network and the flux calculation weighting each GGN edge by PPR scores, edge confidences, and node interaction counts) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Runtime environment for installing and executing pyPARAGON (virtualenv, wheel-based install)) — https://github.com/metunetlab/pyPARAGON

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- Output table covers every GGN edge with finite combined flux scores
- Combined edge scores derive exactly from PPR node scores, edge confidence scores, and node degrees (deg(u), deg(t))
- Combined score equals the negative log of the minimum directional flux, not a product of the two directional fluxes
- Highly scored edges recovered by subsequent edge selection reproduce a context-specific subnetwork consistent with the GGN
- Directional fluxes are computable for both directions of each edge before combination

## Limitations

- Hub proteins may dominate the inferred network with unrelated interactions; flux calculation is a countermeasure against the curse of hubs beyond penalizing hubs by interaction count
- Inferred networks frequently contain generic biological processes (transcription, replication), hindering interpretation
- Incomplete reference interactomes bias inferred networks toward protein complexes over metabolic pathways
- The tool cannot handle alternative copies of hits such as protein isoforms and post-translational modifications; this information is generalized and potentially lost
- No software version is stated for pyPARAGON

## Evidence

- [methods] pyPARAGON propagates signals from omic-hits through the personalized PageRank algorithm in re-scoring all proteins in the reference (Figure 2). Then, the flux calculation weights each edge of the GGN: "pyPARAGON propagates signals from omic-hits through the personalized PageRank algorithm in re-scoring all proteins in the reference (Figure 2). Then, the flux calculation weights each edge of the GGN"
- [readme] the flux calculation weights each edge of the GGN, considering the PageRank scores of nodes, the confidence scores of edges, and the number of node interactions: "the flux calculation weights each edge of the GGN, considering the PageRank scores of nodes, the confidence scores of edges, and the number of node interactions"
- [readme] By random walking from weighted initial nodes in the reference network, the Personalized PageRank algorithm assigns a priority to each node: "By random walking from weighted initial nodes in the reference network, the Personalized PageRank algorithm assigns a priority to each node."
- [discussion] the PageRank algorithm in pyPARAGON propagates the seed nodes before network inference, independent of GGN: "the PageRank algorithm in pyPARAGON propagates the seed nodes before network inference, independent of GGN"
- [discussion] pyPARAGON eliminates the interactions based on the calculated edge fluxes.: "pyPARAGON eliminates the interactions based on the calculated edge fluxes."
