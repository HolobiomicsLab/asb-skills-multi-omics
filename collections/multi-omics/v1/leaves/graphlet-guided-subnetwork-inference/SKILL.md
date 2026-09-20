---
name: graphlet-guided-subnetwork-inference
description: Use when you have a set of omic-hits (e.g., mutated genes or significant
  phosphoproteins) plus a reference interactome and need a context-specific subnetwork
  that trims hub-dominated, nonspecific interactions.
when_to_use_negative:
- Input data are single-cell omics — pyPARAGON's current version integrates bulk omic
  data for static tumor-specific networks; single-cell support is only planned
- Seed hits are protein isoforms or distinct post-translational modifications of the
  same protein — alternative copies of individual hits cannot be handled and the isoform/PTM
  information is generalized and potentially lost in the network
- The goal is metabolic pathway reconstruction — incomplete reference interactomes
  bias inferred networks toward protein complexes over metabolic pathways
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3557
tools:
- name: pyPARAGON
  role: 'Main tool: constructs the GGN, runs Personalized PageRank propagation and
    flux-based edge scoring, and reconstructs the context-specific subnetwork'
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: 'Runtime environment: pyPARAGON is installed as a Python package into a virtual
    environment (python3 -m venv pyPARAGONenv)'
- name: Louvain
  role: Community detection method used by pyPARAGON's community analysis module to
    divide the inferred network into functional units
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:a87ec076768ca55eb3da1633bafc1ebc804c1e40b73ab24b2cbdfdd2d30f445f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - DOMINO
  - Louvain
  - OI2
  - PL
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3557
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# graphlet-guided-subnetwork-inference

## Summary

Infer a context-specific subnetwork from a reference interactome using pyPARAGON, which builds a graphlet-guided network (GGN) from frequent non-isomorphic 2-, 3-, and 4-node graphlets and then propagates omic-hits with Personalized PageRank and flux-based edge scoring to select highly scored edges.

## When to use

Use when you have a set of omic-hits (e.g., mutated genes or significant phosphoproteins) plus a reference interactome and need a context-specific subnetwork that trims hub-dominated, nonspecific interactions.

## When NOT to use

- Input data are single-cell omics — pyPARAGON's current version integrates bulk omic data for static tumor-specific networks; single-cell support is only planned
- Seed hits are protein isoforms or distinct post-translational modifications of the same protein — alternative copies of individual hits cannot be handled and the isoform/PTM information is generalized and potentially lost in the network
- The goal is metabolic pathway reconstruction — incomplete reference interactomes bias inferred networks toward protein complexes over metabolic pathways

## Inputs

- Reference interactome (network with confidence-scored edges, not prefiltered by confidence threshold)
- Seed node set of omic-hits (e.g., most frequently mutated genes per cancer type or significant phosphoproteins from CPTAC breast cancer samples)
- Optional prior-knowledge annotation sets for module testing (e.g., biological processes from GOA, KEGG pathways)

## Outputs

- Graphlet-guided network (GGN) of significant graphlet motifs
- Context-specific subnetwork inferred from highly scored GGN edges
- Optional Louvain communities and hypergeometric-tested overrepresented biological processes/pathways per module

## How to apply

Run pyPARAGON in three steps. (1) GGN construction: count non-isomorphic graphlets (G0–G8, composed of 2-, 3-, and 4-nodes, each with an intermediate node at a highest-degree orbit) in the reference network and in 100 permuted networks with the same seed node set, then retain significant graphlets via z-test (p < 0.05, z-score > 1.65); their union forms the GGN that shrinks the network and eliminates highly connected nodes and their unrelated interactions. (2) Propagation and edge scoring: propagate signals from omic-hits with the Personalized PageRank algorithm to re-score all proteins in the reference, then weight each GGN edge with the flux calculation combining node PageRank scores, edge confidence scores, and the number of node interactions. (3) Subnetwork reconstruction: select the highly scored edges on the GGN to output the context-specific network. The reference interactome is not prefiltered by a confidence threshold; the flux calculation serves as the countermeasure against hub dominance. Optionally, run Louvain community detection on the inferred network and test communities against prior knowledge (e.g., biological processes, pathways) with a hypergeometric test to identify significant modules.

## Related tools

- **pyPARAGON** (Main tool: constructs the GGN, runs Personalized PageRank propagation and flux-based edge scoring, and reconstructs the context-specific subnetwork) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Runtime environment: pyPARAGON is installed as a Python package into a virtual environment (python3 -m venv pyPARAGONenv))
- **Louvain** (Community detection method used by pyPARAGON's community analysis module to divide the inferred network into functional units)

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- GGN construction uses exactly 100 permuted networks with the same seed node set and retains graphlets with z-test p < 0.05 and z-score > 1.65
- The union of significant non-isomorphic 2-, 3-, and 4-node graphlets (G0–G8) forms the GGN; inferred edges must lie within the GGN
- Reconstructed benchmark signaling pathways (e.g., via 5-fold cross-validation on mutated-gene seeds) should recover more IntOGen driver genes with higher precision and recall than PL, OI2, and DOMINO
- On CPTAC breast cancer phosphoproteomic seeds (105 tumors), cosine similarities of overrepresented biological process profiles should cluster tumors into four groups (Fig. 4A stratification)
- Edge flux weights must combine node PageRank scores, edge confidence scores, and node interaction counts

## Limitations

- Hub proteins may still dominate the inferred network with unrelated interactions; the flux calculation is only a countermeasure against the curse of hubs
- Inferred networks frequently contain generic biological processes (e.g., transcription, replication), and protein complexes tend to form more topological modules than metabolic pathways due to incomplete reference interactomes
- The tool cannot handle alternative copies of hits such as protein isoforms and post-translational modifications; this information is generalized and potentially lost in the network
- No single-cell omic support in the current version; a single-cell extension is only planned
- No software version is stated for pyPARAGON, complicating reproducibility

## Evidence

- [readme] pyPARAGON propagates signals from omic-hits through the personalized PageRank algorithm in re-scoring all proteins in the reference (Figure 2). Then, the flux calculation weights each edge of the GGN: "pyPARAGON propagates signals from omic-hits through the personalized PageRank algorithm in re-scoring all proteins in the reference (Figure 2). Then, the flux calculation weights each edge of the GGN"
- [readme] Graphlet module of pyPARAGON identifies, an associated region of reference network through substantially frequent non-isomorphic graphlets composed of 2-, 3-, and 4-nodes: "Graphlet module of pyPARAGON identifies, an associated region of reference network through substantially frequent non-isomorphic graphlets composed of 2-, 3-, and 4-nodes"
- [readme] Using the z-test, we compared the targeted graphlet count in the reference and permuted networks (p < 0.05, z-score > 1.65). The union of significant graphlets constructs GGN.: "Using the z-test, we compared the targeted graphlet count in the reference and permuted networks (p < 0.05, z-score > 1.65). The union of significant graphlets constructs GGN."
- [discussion] Thus, the reference interactome is not prefiltered based on a confidence threshold.: "Thus, the reference interactome is not prefiltered based on a confidence threshold."
- [methods] The community analysis module recruits the Louvain community detection method based on network topology (Figure 3).: "The community analysis module recruits the Louvain community detection method based on network topology (Figure 3)."
