---
name: omics-network-propagation
description: Use when you have a set of significant omic hits (e.g., phosphoproteomic
  seed nodes from 105 breast cancer tumors) and a reference interactome, and need
  sample-specific networks plus functional modules for stratification.
when_to_use_negative:
- Input is single-cell omic data — the current version integrates bulk omic data only;
  single-cell extension is planned for the next version
- Analysis requires preserving protein isoform or PTM-specific information — pyPARAGON
  cannot handle alternative copies of hits, so isoform/PTM information is generalized
  and potentially lost
- Goal is metabolic pathway reconstruction — incomplete reference interactomes bias
  inferred networks toward protein complexes over metabolic pathways
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3520
tools:
- name: pyPARAGON
  role: 'Core tool: builds the GGN, runs Personalized PageRank propagation and flux-based
    edge scoring, reconstructs context-specific subnetworks, and performs Louvain
    community detection with hypergeometric annotation testing'
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: 'Runtime environment: pyPARAGON is installed via pip into a Python virtual
    environment (python3 -m venv pyPARAGONenv)'
- name: Louvain
  role: Community detection method recruited by pyPARAGON's community analysis module
    based on network topology
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:d1e004c84632e0054bfff0933b6ed715b65b0897c26491c075eb155e1b9a8f71
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - Louvain
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# omics-network-propagation

## Summary

Propagate omic hits through a reference interactome using pyPARAGON, which combines graphlet-guided network (GGN) construction with Personalized PageRank flux scoring to infer context-specific subnetworks from seed node sets such as significant phosphoproteins.

## When to use

Use when you have a set of significant omic hits (e.g., phosphoproteomic seed nodes from 105 breast cancer tumors) and a reference interactome, and need sample-specific networks plus functional modules for stratification.

## When NOT to use

- Input is single-cell omic data — the current version integrates bulk omic data only; single-cell extension is planned for the next version
- Analysis requires preserving protein isoform or PTM-specific information — pyPARAGON cannot handle alternative copies of hits, so isoform/PTM information is generalized and potentially lost
- Goal is metabolic pathway reconstruction — incomplete reference interactomes bias inferred networks toward protein complexes over metabolic pathways

## Inputs

- Seed node set of omic-hits (e.g., significant phosphoproteins per tumor from CPTAC breast cancer phosphoproteomic data)
- Reference interactome (protein-protein interaction network with edge confidence scores)
- Prior knowledge annotations (e.g., GO biological processes, KEGG pathways) for module testing

## Outputs

- Graphlet-guided network (GGN) of significant graphlet motifs
- Context-specific subnetwork reconstructed from highest flux-scored edges
- Louvain communities with hypergeometric-test-significant over-represented biological processes/pathways per sample
- Patient-specific networks usable for similarity-based patient stratification

## How to apply

Install pyPARAGON into a Python virtual environment (pip install git+https://github.com/metunetlab/pyPARAGON.git). Load the seed node set of omic hits (e.g., significant phosphoproteins from CPTAC breast cancer tumors) and the reference interactome, then construct the GGN from substantially frequent non-isomorphic graphlets of 2-, 3-, and 4-nodes, comparing graphlet counts against 100 permuted networks with the same seed set via a z-test (p < 0.05, z-score > 1.65). Independently of the graphlet module, propagate signals from omic-hits with the Personalized PageRank algorithm, then weight each GGN edge by the flux calculation combining node PageRank scores, edge confidence scores, and node interaction counts, and reconstruct the subnetwork from the highest-scored edges. Run Louvain community detection on the inferred network and test communities against prior knowledge (biological processes, pathways) with a hypergeometric test to identify significantly over-represented annotations. The rationale is that graphlets filter out nonspecific interactions and counter the dominance of highly connected hub nodes, while PageRank flux prioritizes edges relevant to the seed hits before any reconstruction.

## Related tools

- **pyPARAGON** (Core tool: builds the GGN, runs Personalized PageRank propagation and flux-based edge scoring, reconstructs context-specific subnetworks, and performs Louvain community detection with hypergeometric annotation testing) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Runtime environment: pyPARAGON is installed via pip into a Python virtual environment (python3 -m venv pyPARAGONenv))
- **Louvain** (Community detection method recruited by pyPARAGON's community analysis module based on network topology)

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- GGN construction z-test against 100 permuted networks with the same seed node set must yield significant graphlets at p < 0.05 and z-score > 1.65
- Inferred subnetwork should contain only highly flux-scored edges from the GGN, with hub-dominated unrelated interactions reduced relative to the raw interactome
- On benchmark signaling pathways, node propagation and edge inference should outperform comparison tools PL, OI2, and DOMINO
- For the 105-tumor breast cancer application, cosine similarities between patient networks based on over-represented biological processes should support clustering into four patient groups consistent with Fig. 4A
- Hypergeometric tests on Louvain communities should return significantly over-represented biological processes for each inferred patient network

## Limitations

- No prefiltering of the reference interactome by confidence threshold; hub proteins may still dominate inferred networks with unrelated interactions
- Cannot handle alternative copies of hits such as protein isoforms and post-translational modifications; this information is generalized and potentially lost in the network
- Reference interactomes are incomplete, so inferred networks bias toward protein complexes over metabolic pathways
- Generic biological processes (e.g., transcription and replication) can appear frequently in inferred networks, hindering interpretation; no software version is stated for pyPARAGON

## Evidence

- [methods] pyPARAGON propagates signals from omic-hits through the personalized PageRank algorithm in re-scoring all proteins in the reference: "pyPARAGON propagates signals from omic-hits through the personalized PageRank algorithm in re-scoring all proteins in the reference"
- [readme] GGN z-test thresholds: "Using the z-test, we compared the targeted graphlet count in the reference and permuted networks (p < 0.05, z-score > 1.65). The union of significant graphlets constructs GGN."
- [abstract] abstract performance claim: "Through comprehensive evaluations on benchmark signaling pathways, we demonstrate that pyPARAGON outperforms state-of-the-art approaches in node propagation and edge inference."
- [readme] flux calculation components: "the flux calculation weights each edge of the GGN, considering the PageRank scores of nodes, the confidence scores of edges, and the number of node interactions."
- [discussion] patient clustering on over-represented processes: "Patients were clustered based on the network-based similarity between the overrepresented biological processes identified with functional modules."
