---
name: omic-seed-node-integration
description: Use when you have a weighted seed-node (omic-hits) set (e.g., phosphoproteomic
  hits from tumors) plus a reference interactome with edge confidence scores, and
  need to propagate seed signals and score GGN edges to prioritize interactions for
  context-specific subnetwork reconstruction.
when_to_use_negative:
- The hits contain protein isoforms or post-translational modifications whose site-specific
  information must be preserved — pyPARAGON cannot handle alternative copies of hits
  and this information is generalized and potentially lost in the network
- The goal is sample-specific (per-patient single-sample) network inference, e.g.,
  the SWEET-style setting, rather than a context-specific network from an aggregated
  weighted seed set
- The input is single-cell omic data — the current pyPARAGON version integrates bulk
  omic data for static network models, with single-cell support only planned for the
  next version
edam_operation: http://edamontology.org/operation_0271
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: pyPARAGON
  role: Implements the PPR propagation from weighted omic-hits and the PageRank-flux
    edge scoring/combination over the GGN
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Runtime for running pyPARAGON's propagation and flux calculation utilities
    (venv-based installation)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:b1147748cb4d2104ed7f62319290bbeb1c9a0a0106f1ed284a6205b4d1d356e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_0271
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# omic-seed-node-integration

## Summary

Propagates a weighted set of omic-hit seed nodes across a reference interactome via Personalized PageRank (PPR), then scores every edge of the graphlet-guided network (GGN) by combining two directional PageRank-flux scores f(u→t)=p(u)·c(e)/deg(u) and f(t→u)=p(t)·c(e)/deg(t). This is pyPARAGON's second inference step, independent of the graphlet module, and it re-scores all proteins in the reference network rather than only seed nodes.

## When to use

Use when you have a weighted seed-node (omic-hits) set (e.g., phosphoproteomic hits from tumors) plus a reference interactome with edge confidence scores, and need to propagate seed signals and score GGN edges to prioritize interactions for context-specific subnetwork reconstruction.

## When NOT to use

- The hits contain protein isoforms or post-translational modifications whose site-specific information must be preserved — pyPARAGON cannot handle alternative copies of hits and this information is generalized and potentially lost in the network
- The goal is sample-specific (per-patient single-sample) network inference, e.g., the SWEET-style setting, rather than a context-specific network from an aggregated weighted seed set
- The input is single-cell omic data — the current pyPARAGON version integrates bulk omic data for static network models, with single-cell support only planned for the next version

## Inputs

- Reference interactome (protein interaction network) with edge confidence scores c(e)
- Weighted seed-node (omic-hits) set, e.g., phosphoproteomic omic hits from CPTAC breast cancer tumors

## Outputs

- PPR priority scores p(v) for every protein in the reference network
- Per-edge directional flux scores f(u→t) and f(t→u) for each GGN edge
- Combined PageRank-flux edge score per GGN edge (negative log of the minimum directional flux), e.g., ppr_flux_edge_scores.csv

## How to apply

Load the reference interactome with its edge confidence scores c(e) and the weighted seed-node (omic-hits) set using the pyPARAGON Python utilities; note the article explicitly avoids prefiltering the interactome by a confidence threshold. Run Personalized PageRank with random walking from the weighted initial nodes so that every protein in the reference network receives a re-scored priority p(v) — propagation happens before and independently of the graphlet-guided network (GGN). For each GGN edge (u,t), compute the two directed fluxes f(u→t)=p(u)·c(e)/deg(u) and f(t→u)=p(t)·c(e)/deg(t), normalizing by PPR score, edge confidence, and node interaction counts (degree); degree division also acts as a countermeasure against the curse of hubs instead of penalizing hubs a priori. Combine the two directional fluxes per edge by taking the minimum directional flux (expressed via the negative log as in Eqs. 1–4), rather than multiplying the two directions. Write every GGN edge with its PPR node scores, directional fluxes, and combined edge score (e.g., to ppr_flux_edge_scores.csv); highly scored edges then feed the subsequent subnetwork-reconstruction step.

## Related tools

- **pyPARAGON** (Implements the PPR propagation from weighted omic-hits and the PageRank-flux edge scoring/combination over the GGN) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Runtime for running pyPARAGON's propagation and flux calculation utilities (venv-based installation))

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- The output edge table covers every edge of the GGN, and each combined score is finite and derived from PPR scores, edge confidence, and node degrees
- The combined edge score equals the (negative log of the) minimum of the two directional fluxes f(u→t) and f(t→u) — verify it is not their product
- Directional fluxes decrease with node degree (deg(u), deg(t) in denominators), consistent with hub-countering normalization
- Node scores come from PPR re-scoring of all proteins in the reference, not only seed nodes, and are computed independently of the GGN module
- The highest-scored edges form a usable edge ranking for the subsequent subnetwork reconstruction step

## Limitations

- The reference interactome is not prefiltered by a confidence threshold, and incomplete reference interactomes bias inferred networks toward protein complexes over metabolic pathways
- Hub proteins may dominate inferred networks with unrelated interactions; the flux calculation is only a countermeasure, not a guarantee
- Inferred networks frequently contain generic biological processes (e.g., transcription and replication), hindering interpretation
- Alternative copies of hits (protein isoforms, different PTMs) cannot be handled; isoform/PTM information is generalized and potentially lost
- No software version is stated for pyPARAGON in the article

## Evidence

- [methods] pyPARAGON propagates signals from omic-hits through the personalized PageRank algorithm in re-scoring all proteins in the reference (Figure 2). Then, the flux calculation weights each edge of the GGN: "pyPARAGON propagates signals from omic-hits through the personalized PageRank algorithm in re-scoring all proteins in the reference (Figure 2). Then, the flux calculation weights each edge of the GGN"
- [discussion] the PageRank algorithm in pyPARAGON propagates the seed nodes before network inference, independent of GGN: "the PageRank algorithm in pyPARAGON propagates the seed nodes before network inference, independent of GGN"
- [discussion] Thus, the reference interactome is not prefiltered based on a confidence threshold.: "Thus, the reference interactome is not prefiltered based on a confidence threshold."
- [discussion] the flux calculation in pyPARAGON is a countermeasure against the curse of hubs: "Similarly, the flux calculation in pyPARAGON is a countermeasure against the curse of hubs beyond"
- [readme] By random walking from weighted initial nodes in the reference network, the Personalized PageRank algorithm assigns a priority to each node.: "By random walking from weighted initial nodes in the reference network, the Personalized PageRank algorithm assigns a priority to each node."
