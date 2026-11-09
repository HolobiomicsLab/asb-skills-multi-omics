---
name: patient-stratification-clustering
description: Use when you have per-patient omic hits (e.g. significant phosphoproteins
  from CPTAC breast cancer tumors) plus a reference interactome, and the goal is to
  group patients by network-derived functional similarity rather than raw feature
  clustering.
when_to_use_negative:
- The input is single-cell omic data — the current pyPARAGON version integrates bulk
  omic data for static tumor-specific networks; single-cell extension is only planned
- You need to preserve protein isoform or post-translational-modification-specific
  information — pyPARAGON cannot handle alternative copies of hits and this information
  is generalized and lost in the network
- You only have raw expression/abundance tables without mapped hits and a reference
  interactome — there is no seed node set for propagation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: pyPARAGON
  role: Constructs the GGN, propagates seeds via Personalized PageRank, computes flux-weighted
    edges, and runs Louvain community detection plus hypergeometric enrichment to
    yield the biological process profiles used for patient clustering
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Execution environment (virtualenv pyPARAGONenv) in which pyPARAGON is installed
    and the cosine-similarity computation and clustering are scripted
- name: Louvain
  role: Community detection method recruited by pyPARAGON's community analysis module
    to divide each inferred tumor network into functional units for annotation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:3dfd07bd093f03bdfcf98ad5ab31bc4187ccee90c0e8aaf42693c447821eb849
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# patient-stratification-clustering

## Summary

Network-based stratification of patient tumors by inferring tumor-specific subnetworks with pyPARAGON from significant phosphoproteins (seed nodes) on a reference interactome, then clustering 105 CPTAC breast cancer tumors via cosine similarity of over-represented biological process profiles, reproducing the four-group stratification of Fig. 4A.

## When to use

Use when you have per-patient omic hits (e.g. significant phosphoproteins from CPTAC breast cancer tumors) plus a reference interactome, and the goal is to group patients by network-derived functional similarity rather than raw feature clustering.

## When NOT to use

- The input is single-cell omic data — the current pyPARAGON version integrates bulk omic data for static tumor-specific networks; single-cell extension is only planned
- You need to preserve protein isoform or post-translational-modification-specific information — pyPARAGON cannot handle alternative copies of hits and this information is generalized and lost in the network
- You only have raw expression/abundance tables without mapped hits and a reference interactome — there is no seed node set for propagation

## Inputs

- Significant phosphoproteomic hits per tumor (seed node sets) for 105 CPTAC breast cancer tumors
- Reference protein-protein interactome (not prefiltered by confidence threshold)
- Biological process annotations (e.g. GO biological processes) as prior knowledge for hypergeometric testing

## Outputs

- Patient-specific context-specific subnetworks (highest-flux edges on the GGN)
- Per-patient over-represented biological process profiles from Louvain communities
- Cosine similarity matrix between patient-specific networks
- patient_stratification_results.csv with four-group cluster assignments per tumor

## How to apply

Install pyPARAGON into a Python virtual environment and construct the graphlet-guided network (GGN) from significantly frequent non-isomorphic 2-, 3-, and 4-node graphlets, testing against 100 permuted networks with the same seed node set (z-test, p < 0.05, z-score > 1.65). For each of the 105 tumors, propagate signals from its omic-hits with the Personalized PageRank algorithm, weight each GGN edge by the flux calculation combining node PageRank scores, edge confidence scores, and node interaction counts, and reconstruct the patient-specific subnetwork from the highest-scored edges. Run Louvain community detection on each inferred network and test communities against biological process annotations with the hypergeometric test to obtain per-patient over-represented biological process profiles. Compute cosine similarities between patients from these profiles, cluster the similarity matrix (expecting four groups, consistent with Fig. 4A), and write assignments to patient_stratification_results.csv. The rationale is that graphlet-guided filtering trims nonspecific interactions and hub dominance before propagation, so patient similarity reflects context-specific signaling rather than generic interactome structure.

## Related tools

- **pyPARAGON** (Constructs the GGN, propagates seeds via Personalized PageRank, computes flux-weighted edges, and runs Louvain community detection plus hypergeometric enrichment to yield the biological process profiles used for patient clustering) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Execution environment (virtualenv pyPARAGONenv) in which pyPARAGON is installed and the cosine-similarity computation and clustering are scripted)
- **Louvain** (Community detection method recruited by pyPARAGON's community analysis module to divide each inferred tumor network into functional units for annotation)

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- Clustering of the patient cosine-similarity matrix recovers exactly four patient groups consistent with the network-based stratification reported in Fig. 4A
- patient_stratification_results.csv contains assignments for all 105 tumors
- GGN construction thresholds hold: significant graphlets identified at p < 0.05 and z-score > 1.65 against 100 permuted networks with the same seed node set
- Each patient's inferred network yields Louvain communities with significantly over-represented biological processes (hypergeometric test) before similarity computation
- The cosine similarity matrix is symmetric with diagonal ~1, built from the patients' over-represented biological process profiles

## Limitations

- Hub proteins may dominate inferred networks with unrelated interactions; the flux calculation is a countermeasure but hub dominance remains a risk
- Generic biological processes (e.g. transcription, replication) appear frequently in inferred networks, complicating interpretation; incomplete interactomes bias modules toward protein complexes over metabolic pathways
- pyPARAGON cannot handle alternative copies of hits (protein isoforms, different PTMs); isoform/PTM information is generalized and potentially lost in the network
- No software version is stated for pyPARAGON, and single-cell omic support is not yet available

## Evidence

- [abstract] network-based stratification of patient tumors by integrating phosphoproteomic data from 105 breast cancer tumors with the interactome: "we demonstrate its utility in network-based stratification of patient tumors by integrating phosphoproteomic data from 105 breast cancer tumors with the interactome"
- [discussion] Patients were clustered based on the network-based similarity between the overrepresented biological processes identified with functional modules.: "Patients were clustered based on the network-based similarity between the overrepresented biological processes identified with functional modules."
- [methods] The community analysis module recruits the Louvain community detection method based on network topology (Figure 3).: "The community analysis module recruits the Louvain community detection method based on network topology (Figure 3)."
- [readme] The communities are tested with hypergeometric tests to identify significantly overrepresented annotations.: "The communities are tested with hypergeometric tests to identify significantly overrepresented annotations."
