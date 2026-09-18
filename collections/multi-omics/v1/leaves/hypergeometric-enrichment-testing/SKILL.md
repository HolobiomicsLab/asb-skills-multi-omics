---
name: hypergeometric-enrichment-testing
description: Use when louvain modules from pyPARAGON-inferred subnetworks (e.g., 105
  CPTAC breast cancer tumor networks) must be annotated with GO biological processes
  or KEGG pathways via a hypergeometric test to support patient stratification.
when_to_use_negative:
- Input is single-cell omic data — the current pyPARAGON version integrates bulk omic
  data only; single-cell support is planned for the next version
- The goal is raw omics analysis before a context-specific network exists — the hypergeometric
  test operates on modules of an inferred subnetwork, not directly on expression or
  phosphoproteomics tables
- No suitable prior-knowledge annotation source (GOA, KEGG, TTP) is available for
  the organism or entity type in the modules
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: pyPARAGON
  role: Community analysis module performs Louvain community detection and hypergeometric
    over-representation testing of modules against GO biological processes and KEGG
    pathways
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Runtime environment for installing and executing pyPARAGON (python3 venv,
    pip wheel install)
- name: KEGG
  role: Pathway annotation source used as the prior-knowledge set in the hypergeometric
    test for the Fig. 4D cluster frequency analysis
merged_aliases:
- hypergeometric-biological-process-enrichment-testing
merged_alias_records:
- alias: hypergeometric-biological-process-enrichment-testing
  slug: hypergeometric-biological-process-enrichment-testing
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:4b8f65185eb826cc3c5bc37e5a125d1ec3712d96f05c23875ad30d0528c358fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - KEGG
  - Louvain
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# hypergeometric-enrichment-testing

## Summary

Within pyPARAGON's community analysis module, Louvain-detected functional modules of inferred context-specific subnetworks are tested against prior-knowledge annotations (GO biological processes, KEGG pathways, TTP drugs) with the hypergeometric test to identify significantly over-represented terms, enabling breast cancer patient stratification and pathway-frequency analysis (Fig. 4D).

## When to use

Use when Louvain modules from pyPARAGON-inferred subnetworks (e.g., 105 CPTAC breast cancer tumor networks) must be annotated with GO biological processes or KEGG pathways via a hypergeometric test to support patient stratification.

## When NOT to use

- Input is single-cell omic data — the current pyPARAGON version integrates bulk omic data only; single-cell support is planned for the next version
- The goal is raw omics analysis before a context-specific network exists — the hypergeometric test operates on modules of an inferred subnetwork, not directly on expression or phosphoproteomics tables
- No suitable prior-knowledge annotation source (GOA, KEGG, TTP) is available for the organism or entity type in the modules

## Inputs

- Patient-cluster-specific subnetworks from pyPARAGON network inference (105 breast cancer tumors, CPTAC phosphoproteomic seed hits on a reference interactome)
- Inferred context-specific networks or arbitrary networks with functional modules from Louvain community detection
- Prior-knowledge annotation sets: GOA biological processes, KEGG pathways, TTP drug knowledge

## Outputs

- Significantly over-represented GO biological processes or KEGG pathways per Louvain module
- Aggregated over-representation frequency table per patient cluster (e.g., pathway_overrepresentation_table.csv reproducing Fig. 4D)
- Over-represented biological-process profiles per patient network used for cosine-similarity-based patient stratification

## How to apply

Run Louvain community detection on each inferred context-specific subnetwork (built from CPTAC breast cancer phosphoproteomic seed nodes via the GGN, personalized PageRank, and flux edge scoring) to partition it into functional modules. For each community, apply a hypergeometric test against a chosen prior-knowledge source — biological processes in GOA, pathways in KEGG, or drug knowledge in TTP — to flag significantly over-represented annotations. Aggregate significant terms per patient network; over-represented biological process profiles form the basis of patient clustering via cosine similarity between networks, and KEGG hits are tabulated per patient cluster to reproduce the Fig. 4D over-representation frequency table. For the Ras-signaling check, extract the module connecting cluster phosphoproteins through KRAS, NRAS, HRAS, RHOA, and RHOD (Fig. 4E). The same community-analysis module can be run independently of the network-inference module on any network with biological annotations.

## Related tools

- **pyPARAGON** (Community analysis module performs Louvain community detection and hypergeometric over-representation testing of modules against GO biological processes and KEGG pathways) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Runtime environment for installing and executing pyPARAGON (python3 venv, pip wheel install))
- **KEGG** (Pathway annotation source used as the prior-knowledge set in the hypergeometric test for the Fig. 4D cluster frequency analysis)

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- The output table (e.g., pathway_overrepresentation_table.csv) exists as a structured CSV of pathway over-representation frequencies per patient cluster
- Cell cycle and PI3K/Akt signaling pathways appear most frequently over-represented across breast cancer patient clusters, with reduced frequency in cluster-2, matching Fig. 4D
- The extracted Ras-signaling module contains the five intermediate nodes KRAS, NRAS, HRAS, RHOA, and RHOD (Fig. 4E)
- Clustering on cosine similarities of over-represented biological-process profiles recovers four patient groups consistent with Fig. 4A
- Each tested module yields only annotations passing the hypergeometric significance criterion, with no enrichment reported for untested/empty communities

## Limitations

- Inferred networks frequently contain generic biological processes such as transcription and replication, which can dominate enrichment results and hinder interpretation
- Incomplete reference interactomes bias topological modules toward protein complexes over metabolic pathways, affecting which terms become enriched
- pyPARAGON cannot handle alternative copies of hits such as protein isoforms or different post-translational modifications; isoform/PTM information is generalized and potentially lost in the network

## Evidence

- [readme] The communities are tested with hypergeometric tests to identify significantly overrepresented annotations.: "The communities are tested with hypergeometric tests to identify significantly overrepresented annotations."
- [methods] The community analysis module recruits the Louvain community detection method based on network topology (Figure 3).: "The community analysis module recruits the Louvain community detection method based on network topology (Figure 3)."
- [discussion] such as biological processes in GOA, pathways in KEGG, and drug knowledge in TTP: "such as biological processes in GOA, pathways in KEGG, and drug knowledge in TTP"
- [discussion] Patients were clustered based on the network-based similarity between the overrepresented biological processes identified with functional modules.: "Patients were clustered based on the network-based similarity between the overrepresented biological processes identified with functional modules."
- [readme] This integration of annotations and inferred subnetworks provides patient stratification, survival analysis, and personalized medicine.: "This integration of annotations and inferred subnetworks provides patient stratification, survival analysis, and personalized medicine."
