---
name: pagerank-flux-edge-scoring
description: Use when a weighted seed-node (omic-hits) set must be propagated over
  a confidence-scored reference interactome with PPR and edges prioritized by flux
  scores, e.g. before subnetwork reconstruction on a graphlet-guided network (GGN).
when_to_use_negative:
- The task needs sample-specific network inference rather than seed-propagation scoring
  (e.g. SWEET-style sample-specific networks).
- 'Input data are single-cell omics: the current pyPARAGON version integrates only
  bulk omic data for static network models.'
- The goal is to model protein isoforms or post-translational modifications explicitly
  — pyPARAGON cannot handle alternative copies of hits and this information is generalized
  (potentially lost) in the network.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0602
tools:
- name: pyPARAGON
  role: Implements the PPR propagation and PageRank-flux edge scoring (Eqs. 1-4) over
    the GGN; installed via pip from the GitHub repo
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Runtime environment (python3 -m venv pyPARAGONenv) in which pyPARAGON's propagation
    and flux-calculation utilities are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:aeebae815f6caccccbf13b3bcadae3d7fed4f9dcbb36f5f98eb0e9d6bbc0b327
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
yes_rate: 0.9090909090909091
n_claims: 11
---

# pagerank-flux-edge-scoring

## Summary

pyPARAGON's propagation and edge-scoring step: seed omic-hits are propagated over the reference interactome with Personalized PageRank (PPR), then each GGN edge is scored by directional fluxes f(u→t) = p(u)·c(e)/deg(u) and f(t→u) = p(t)·c(e)/deg(t), combined by assigning the minimum directional flux (negative log) to the edge.

## When to use

Use when a weighted seed-node (omic-hits) set must be propagated over a confidence-scored reference interactome with PPR and edges prioritized by flux scores, e.g. before subnetwork reconstruction on a graphlet-guided network (GGN).

## When NOT to use

- The task needs sample-specific network inference rather than seed-propagation scoring (e.g. SWEET-style sample-specific networks).
- Input data are single-cell omics: the current pyPARAGON version integrates only bulk omic data for static network models.
- The goal is to model protein isoforms or post-translational modifications explicitly — pyPARAGON cannot handle alternative copies of hits and this information is generalized (potentially lost) in the network.

## Inputs

- Reference interactome (network with edge confidence scores c(e))
- Weighted seed-node set of omic-hits (e.g. phosphoproteomic hits from CPTAC breast cancer samples)
- Graphlet-guided network (GGN) edges on which fluxes are computed

## Outputs

- ppr_flux_edge_scores.csv: per-edge table with PPR node scores, directional fluxes f(u→t) and f(t→u), and combined edge scores for every GGN edge
- Re-scored PPR priority values for all proteins in the reference network

## How to apply

Load the reference interactome with edge confidence scores and the weighted seed-node (omic-hits) set using pyPARAGON's Python utilities; the reference interactome is not prefiltered by a confidence threshold. Run Personalized PageRank with random walking from the weighted initial nodes to re-score every protein in the reference network, independent of the GGN module. For each edge of the GGN, compute directed fluxes f(u→t) = p(u)·c(e)/deg(u) and f(t→u) = p(t)·c(e)/deg(t) from the PPR node scores, edge confidence scores, and node degrees. Combine the two directional fluxes per edge into a single score by taking the negative log of the minimum directional flux (Eqs. 1-4) — assigning the minimum, not multiplying — which also serves as a countermeasure against hub dominance. Write all GGN edges with their PPR node scores, directional fluxes, and combined edge scores to ppr_flux_edge_scores.csv so highly scored edges can later be selected for subnetwork reconstruction.

## Related tools

- **pyPARAGON** (Implements the PPR propagation and PageRank-flux edge scoring (Eqs. 1-4) over the GGN; installed via pip from the GitHub repo) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Runtime environment (python3 -m venv pyPARAGONenv) in which pyPARAGON's propagation and flux-calculation utilities are executed)

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git && python -c "import pyPARAGON; pyPARAGON.run_ppr_flux_edge_scores()"  # writes ppr_flux_edge_scores.csv
```

## Evaluation signals

- The output ppr_flux_edge_scores.csv covers every GGN edge with finite combined flux scores.
- Combined edge scores are consistent with the negative log of the minimum directional flux, not the product of the two directional fluxes.
- Directional flux values equal p(node)·c(e)/deg(node) using the PPR scores, edge confidence scores, and node degrees.
- PPR node scores are finite and defined for every protein in the reference network, not only seed nodes (PPR re-scores all proteins).
- Highly scored edges selected downstream should reduce inclusion of nonspecific interactions and hub-dominated edges relative to pairwise scoring.

## Limitations

- Hub proteins may still dominate inferred networks with unrelated interactions; the flux calculation is only a countermeasure against the curse of hubs.
- The reference interactome is incomplete, biasing inferred networks toward protein complexes over metabolic pathways; it is not prefiltered by a confidence threshold.
- Inferred networks frequently contain generic biological processes (e.g. transcription, replication), hindering interpretation.
- The tool cannot handle alternative copies of hits such as protein isoforms and PTMs; no software version is stated for pyPARAGON.

## Evidence

- [methods] pyPARAGON propagates signals from omic-hits through the personalized PageRank algorithm in re-scoring all proteins in the reference (Figure 2). Then, the flux calculation weights each edge of the GGN: "pyPARAGON propagates signals from omic-hits through the personalized PageRank algorithm in re-scoring all proteins in the reference (Figure 2). Then, the flux calculation weights each edge of the GGN"
- [discussion] PageRank flux calculation for edge prioritization and GGN integration: "We leveraged the PageRank flux calculation for edge prioritization and integrated GGNs to successfully construct context-specific networks."
- [readme] README: propagation then flux weighting considering node scores, edge confidence, and interaction counts: "the flux calculation weights each edge of the GGN, considering the PageRank scores of nodes, the confidence scores of edges, and the number of node interactions."
- [discussion] Flux calculation as hub countermeasure: "Similarly, the flux calculation in pyPARAGON is a countermeasure against the curse of hubs beyond"
- [discussion] No confidence-threshold prefiltering of the reference interactome: "Thus, the reference interactome is not prefiltered based on a confidence threshold."
