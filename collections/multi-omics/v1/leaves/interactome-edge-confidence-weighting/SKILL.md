---
name: interactome-edge-confidence-weighting
description: Use when you have a reference interactome with edge confidence scores,
  a weighted omic-hits seed set, and a GGN, and you need to rank edges to reconstruct
  a context-specific subnetwork.
when_to_use_negative:
- You want to prefilter the reference interactome by an edge-confidence cutoff before
  analysis — pyPARAGON explicitly does not prefilter on confidence; confidences enter
  only during flux scoring
- Your hits require isoform- or PTM-level resolution — the tool cannot handle alternative
  copies of individual hits (protein isoforms, different post-translational modifications)
  and this information is generalized/lost in the network
- Your data are single-cell omics — the released pyPARAGON integrates bulk omic data
  for static networks; single-cell support is only planned for a future version
edam_operation: http://edamontology.org/operation_0277
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: pyPARAGON
  role: Implements the PPR propagation and PageRank-flux edge scoring over the reference
    interactome and GGN
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Runtime environment for installing and running pyPARAGON (venv, pip, setup.py
    bdist_wheel)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:c375698881108cea2e6a82501f992675f6c64abfa3b26a1fd7c70caa9dbecd35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_0277
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# interactome-edge-confidence-weighting

## Summary

Score every edge of pyPARAGON's Graphlet-guided network (GGN) by combining Personalized PageRank node scores with edge confidence values and node degrees via directional PageRank-flux terms, assigning the minimum directional flux (negative-log transformed) as the single edge score.

## When to use

Use when you have a reference interactome with edge confidence scores, a weighted omic-hits seed set, and a GGN, and you need to rank edges to reconstruct a context-specific subnetwork.

## When NOT to use

- You want to prefilter the reference interactome by an edge-confidence cutoff before analysis — pyPARAGON explicitly does not prefilter on confidence; confidences enter only during flux scoring
- Your hits require isoform- or PTM-level resolution — the tool cannot handle alternative copies of individual hits (protein isoforms, different post-translational modifications) and this information is generalized/lost in the network
- Your data are single-cell omics — the released pyPARAGON integrates bulk omic data for static networks; single-cell support is only planned for a future version

## Inputs

- Reference interactome (nodes plus edges carrying confidence scores c(e))
- Weighted seed-node set derived from omic-hits (e.g. phosphoproteomic hits)
- Graphlet-guided network (GGN) edges built from significant 2-, 3-, and 4-node graphlets (p < 0.05, z-score > 1.65)

## Outputs

- PPR scores for every protein in the reference network
- Directional flux values f(u->t) and f(t->u) per GGN edge
- Combined edge scores (negative log of the minimum directional flux)
- ppr_flux_edge_scores.csv table covering every GGN edge with node scores, directional fluxes, and combined edge scores

## How to apply

First run Personalized PageRank over the full reference interactome, random walking from the weighted initial (omic-hits) nodes so every protein is re-scored; this propagation is independent of the graphlet module and of GGN construction. Then, for each edge of the GGN, compute the two directional fluxes f(u->t) = p(u)*c(e)/deg(u) and f(t->u) = p(t)*c(e)/deg(t), where p are the PPR node scores, c(e) are the reference edge confidence scores, and deg() are node interaction counts (the flux calculation acts as a countermeasure against the curse of hubs, instead of explicitly penalizing hubs). Combine the two directional fluxes into one edge score by taking the negative log of the minimum directional flux (Eqs. 1-4) — note this is a minimum, not a product, of the two directions. Note that the reference interactome is not prefiltered by confidence threshold; confidence scores only enter at edge-scoring time. Write all GGN edges with their PPR node scores, directional fluxes, and combined edge scores to ppr_flux_edge_scores.csv, from which highly scored edges are subsequently selected for subnetwork reconstruction.

## Related tools

- **pyPARAGON** (Implements the PPR propagation and PageRank-flux edge scoring over the reference interactome and GGN) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Runtime environment for installing and running pyPARAGON (venv, pip, setup.py bdist_wheel))

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- Output table ppr_flux_edge_scores.csv covers every GGN edge with finite combined flux scores (no missing or null entries)
- Each combined edge score is reproducible as the negative log of the minimum of the two directional fluxes, not their product or sum
- Directional fluxes recomputed from the stored PPR scores, edge confidences c(e), and node degrees match the reported values
- PPR scores sum appropriately over the reference network and rank seed-proximal proteins higher than distant ones
- Downstream edge selection from the scored table yields a connected, context-specific subnetwork consistent with the highly scored edges of the GGN

## Limitations

- Hub proteins may dominate the inferred network with unrelated interactions (flux calculation mitigates but does not eliminate this)
- Reference interactomes are incomplete, biasing inferred networks toward protein complexes over metabolic pathways
- Tool cannot handle alternative copies of hits such as protein isoforms and PTMs; that information is generalized and potentially lost
- No single-cell omic support in the current version; extension is only planned
- No software version is stated for pyPARAGON, complicating reproducibility

## Evidence

- [methods] pyPARAGON propagates signals from omic-hits through the personalized PageRank algorithm in re-scoring all proteins in the reference (Figure 2). Then, the flux calculation weights each edge of the GGN: "pyPARAGON propagates signals from omic-hits through the personalized PageRank algorithm in re-scoring all proteins in the reference (Figure 2). Then, the flux calculation weights each edge of the GGN"
- [discussion] the PageRank algorithm in pyPARAGON propagates the seed nodes before network inference, independent of GGN: "the PageRank algorithm in pyPARAGON propagates the seed nodes before network inference, independent of GGN"
- [readme] By random walking from weighted initial nodes in the reference network, the Personalized PageRank algorithm assigns a priority to each node. The computation of fluxes between interacting nodes scored edges in the reference.: "By random walking from weighted initial nodes in the reference network, the Personalized PageRank algorithm assigns a priority to each node."
- [discussion] Thus, the reference interactome is not prefiltered based on a confidence threshold.: "Thus, the reference interactome is not prefiltered based on a confidence threshold."
- [discussion] the flux calculation in pyPARAGON is a countermeasure against the curse of hubs: "The prize-collecting Steiner Forest algorithm penalizes hubs based on the number of interactions. Similarly, the flux calculation in pyPARAGON is a countermeasure against the curse of hubs"
