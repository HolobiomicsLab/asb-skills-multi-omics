---
name: cosine-similarity-network-comparison
description: Use when multiple sample-specific networks (e.g. pyPARAGON subnetworks
  per tumor) must be compared and grouped, via functional-module annotation profiles
  such as over-represented biological processes.
when_to_use_negative:
- Input is raw phosphoproteomic abundance data rather than inferred patient-specific
  networks with annotated functional modules — the GGN/PPR/flux inference pipeline
  must run first
- The goal is comparison of networks at the sequence/interaction level rather than
  via functional annotation profiles (e.g. direct graph-structure similarity of edges)
- Single-cell omic data are the input, since pyPARAGON integrates bulk omic data for
  static tumor-specific network models
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3524
tools:
- name: pyPARAGON
  role: Infers each patient-specific subnetwork (GGN construction, PPR propagation,
    flux-based edge scoring) and runs Louvain community detection plus hypergeometric
    testing to produce the over-represented biological process profiles that are compared
    by cosine similarity
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Louvain
  role: Community detection method that divides each inferred network into functional
    units whose over-represented annotations feed the similarity computation
- name: Python
  role: Environment in which pyPARAGON is installed (venv) and in which cosine similarities
    are computed and patients clustered
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:992d8b133b3f616f13072be1f0abb0dbb2bbd6ab064622146b44620442614b7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - Louvain
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cosine-similarity-network-comparison

## Summary

Compares patient-specific context-subnetworks inferred by pyPARAGON by computing cosine similarities between their over-represented biological process profiles, then clustering the similarity matrix — the step that turns per-tumor networks into the four-group stratification of 105 CPTAC breast cancer tumors (Fig. 4A).

## When to use

Use when multiple sample-specific networks (e.g. pyPARAGON subnetworks per tumor) must be compared and grouped, via functional-module annotation profiles such as over-represented biological processes.

## When NOT to use

- Input is raw phosphoproteomic abundance data rather than inferred patient-specific networks with annotated functional modules — the GGN/PPR/flux inference pipeline must run first
- The goal is comparison of networks at the sequence/interaction level rather than via functional annotation profiles (e.g. direct graph-structure similarity of edges)
- Single-cell omic data are the input, since pyPARAGON integrates bulk omic data for static tumor-specific network models

## Inputs

- Per-patient pyPARAGON context-specific subnetworks inferred from the CPTAC breast cancer phosphoproteomic dataset (105 tumors, significant phosphoproteins as seed node set) and a reference interactome
- Per-patient over-represented biological process profiles obtained from hypergeometric testing of Louvain communities against GO biological process annotations

## Outputs

- Pairwise patient-by-patient cosine similarity matrix based on over-represented biological process profiles
- Cluster assignments stratifying the 105 breast cancer tumors into four groups, written to patient_stratification_results.csv

## How to apply

First infer each patient-specific subnetwork with pyPARAGON: build the graphlet-guided network (GGN) from significantly frequent non-isomorphic 2-, 3-, and 4-node graphlets (z-test, p < 0.05, z-score > 1.65 against 100 permuted networks), propagate omic-hits with Personalized PageRank, weight GGN edges by the flux calculation combining node PageRank scores, edge confidence scores, and node interaction counts, and reconstruct each subnetwork from the highest-scored edges. Run Louvain community detection on each inferred network and test communities against biological process annotations with the hypergeometric test to obtain each patient's over-represented biological process profile. Encode these profiles (e.g. as binary/significance vectors over biological processes), compute all pairwise cosine similarities between patient profiles, and cluster patients on the resulting similarity matrix. The rationale is that patients whose networks show the same over-represented biological processes share tumor-specific signaling — producing the four-group stratification reported in Figure 4A, written to patient_stratification_results.csv.

## Related tools

- **pyPARAGON** (Infers each patient-specific subnetwork (GGN construction, PPR propagation, flux-based edge scoring) and runs Louvain community detection plus hypergeometric testing to produce the over-represented biological process profiles that are compared by cosine similarity) — https://github.com/netlab-ku/pyPARAGON
- **Louvain** (Community detection method that divides each inferred network into functional units whose over-represented annotations feed the similarity computation)
- **Python** (Environment in which pyPARAGON is installed (venv) and in which cosine similarities are computed and patients clustered)

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- Clustering the patient-by-patient cosine similarity matrix recovers exactly four patient groups consistent with the network-based stratification reported in Figure 4A
- Each patient's network yields significant Louvain communities annotated by hypergeometric test against biological processes; networks without any over-represented processes indicate a failed inference upstream
- Cosine similarity values are in [-1, 1] and the matrix is symmetric with unit diagonal (self-similarity = 1)
- Cluster assignments are written to patient_stratification_results.csv and every one of the 105 tumors receives exactly one cluster label

## Limitations

- Inferred networks frequently contain generic biological processes such as transcription and replication, which can dominate annotation profiles and hinder interpretation
- Incomplete reference interactomes bias inferred networks toward protein complexes rather than metabolic pathways, affecting the biological-process profiles being compared
- pyPARAGON cannot handle alternative copies of hits (protein isoforms, different post-translational modifications), so isoform/PTM information is generalized and potentially lost before comparison
- Biological interpretation of context-specific networks through causal relations, modules, and biological processes is challenging; no software version is stated for pyPARAGON

## Evidence

- [abstract] we demonstrate its utility in network-based stratification of patient tumors by integrating phosphoproteomic data from 105 breast cancer tumors: "we demonstrate its utility in network-based stratification of patient tumors by integrating phosphoproteomic data from 105 breast cancer tumors"
- [discussion] Patients were clustered based on the network-based similarity between the overrepresented biological processes identified with functional modules.: "Patients were clustered based on the network-based similarity between the overrepresented biological processes identified with functional modules."
- [methods] The community analysis module recruits the Louvain community detection method based on network topology (Figure 3).: "The community analysis module recruits the Louvain community detection method based on network topology (Figure 3)."
- [readme] Using the z-test, we compared the targeted graphlet count in the reference and permuted networks (p < 0.05, z-score > 1.65).: "Using the z-test, we compared the targeted graphlet count in the reference and permuted networks (p < 0.05, z-score > 1.65)."
