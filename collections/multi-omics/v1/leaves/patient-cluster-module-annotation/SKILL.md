---
name: patient-cluster-module-annotation
description: 'Use when patient-cluster-specific inferred subnetworks need functional
  interpretation: Louvain modules must be tested for KEGG over-representation, pathway
  frequencies tabulated per cluster, or a Ras-signaling module extracted.'
when_to_use_negative:
- When only raw omic data are available and context-specific subnetworks have not
  yet been inferred — this skill consumes pyPARAGON network-inference output, not
  raw phosphoproteomics.
- When the prior knowledge set is single-cell or isoform/PTM-resolved — the article
  notes pyPARAGON cannot handle alternative copies of hits such as protein isoforms
  and post-translational modifications.
- When the goal is differential expression or clustering of patients rather than biological
  interpretation of already-inferred context-specific networks.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0602
tools:
- name: pyPARAGON
  role: 'Community analysis module: runs Louvain community detection on cluster-specific
    subnetworks and applies hypergeometric tests against prior knowledge (KEGG) to
    identify overrepresented pathways'
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Runtime environment for pyPARAGON and for aggregating pathway frequencies
    and exporting pathway_overrepresentation_table.csv
- name: KEGG
  role: Pathway prior-knowledge source used as the annotation set for hypergeometric
    over-representation testing
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:a8927d930365577a55fe32e56325dfd9e893369dec31fbcda17d895c41c0608d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - KEGG
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# patient-cluster-module-annotation

## Summary

Annotates Louvain-derived functional modules of patient-cluster-specific pyPARAGON subnetworks with prior knowledge (KEGG pathways) via hypergeometric tests, then tabulates pathway over-representation frequencies per patient cluster and extracts specific signaling modules (e.g., the Ras module). This reproduces the breast cancer (105 CPTAC tumors) pathway-frequency analysis (Fig. 4D) of the pyPARAGON paper.

## When to use

Use when patient-cluster-specific inferred subnetworks need functional interpretation: Louvain modules must be tested for KEGG over-representation, pathway frequencies tabulated per cluster, or a Ras-signaling module extracted.

## When NOT to use

- When only raw omic data are available and context-specific subnetworks have not yet been inferred — this skill consumes pyPARAGON network-inference output, not raw phosphoproteomics.
- When the prior knowledge set is single-cell or isoform/PTM-resolved — the article notes pyPARAGON cannot handle alternative copies of hits such as protein isoforms and post-translational modifications.
- When the goal is differential expression or clustering of patients rather than biological interpretation of already-inferred context-specific networks.

## Inputs

- Patient-cluster-specific subnetworks from pyPARAGON network inference on the 105-tumor breast cancer phosphoproteomic dataset
- KEGG pathway annotations (prior knowledge set for hypergeometric testing)
- Node membership of the five Ras intermediate nodes: KRAS, NRAS, HRAS, RHOA, RHOD

## Outputs

- pathway_overrepresentation_table.csv — aggregated KEGG pathway over-representation frequencies per patient cluster (Fig. 4D analogue)
- Per-cluster list of Louvain functional modules with significant KEGG pathway assignments
- Extracted Ras-signaling module subgraph linking cluster phosphoproteins to KRAS, NRAS, HRAS, RHOA, and RHOD (Fig. 4E analogue)

## How to apply

Load each patient-cluster-specific subnetwork (from the pyPARAGON network-inference output on the 105-tumor breast cancer phosphoproteomic stratification) into the pyPARAGON community analysis module, which partitions the network with Louvain community detection based on network topology. For each community, apply a hypergeometric test against KEGG pathway annotations (the paper uses prior knowledge such as 'pathways in KEGG') to find significantly overrepresented pathways. Aggregate significant pathways per cluster and count their over-representation frequency across clusters, reproducing the Fig. 4D table (cell cycle and PI3K/Akt signaling are the most frequent except in cluster-2). Separately extract and visualize the Ras-signaling module connecting cluster phosphoproteins to intermediate nodes KRAS, NRAS, HRAS, RHOA, and RHOD (Fig. 4E), and save the aggregated table as pathway_overrepresentation_table.csv.

## Related tools

- **pyPARAGON** (Community analysis module: runs Louvain community detection on cluster-specific subnetworks and applies hypergeometric tests against prior knowledge (KEGG) to identify overrepresented pathways) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Runtime environment for pyPARAGON and for aggregating pathway frequencies and exporting pathway_overrepresentation_table.csv)
- **KEGG** (Pathway prior-knowledge source used as the annotation set for hypergeometric over-representation testing)

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git; python3 -m venv pyPARAGONenv && source venv/bin/activate
```

## Evaluation signals

- pathway_overrepresentation_table.csv exists as a structured CSV with per-cluster pathway over-representation frequencies
- Frequency distribution of over-represented pathways across clusters is consistent with the Fig. 4D pattern (cell cycle and PI3K/Akt signaling most frequent, less frequent in cluster-2)
- The extracted Ras-signaling module contains all five intermediate nodes: KRAS, NRAS, HRAS, RHOA, and RHOD
- Each module's pathway calls derive from a hypergeometric test of community members against KEGG annotations, not raw node overlap
- Louvain communities are reproducible per cluster network based on network topology (community detection runs on every cluster-specific subnetwork)

## Limitations

- Reference interactomes are incomplete, so protein complexes tend to form more topological modules than metabolic pathways, biasing module-level interpretation
- Generic biological processes such as transcription and replication can be found more frequently in inferred networks, complicating pathway interpretation
- pyPARAGON cannot handle alternative copies of individual hits (e.g., protein isoforms and different PTMs), so isoform/PTM information is generalized and potentially lost in the network

## Evidence

- [methods] The community analysis module recruits the Louvain community detection method based on network topology (Figure 3).: "The community analysis module recruits the Louvain community detection method based on network topology (Figure 3)."
- [readme] The communities are tested with hypergeometric tests to identify significantly overrepresented annotations.: "The communities are tested with hypergeometric tests to identify significantly overrepresented annotations."
- [discussion] such as biological processes in GOA, pathways in KEGG, and drug knowledge in TTP: "such as biological processes in GOA, pathways in KEGG, and drug knowledge in TTP"
- [discussion] Patients were clustered based on the network-based similarity between the overrepresented biological processes identified with functional modules.: "Patients were clustered based on the network-based similarity between the overrepresented biological processes identified with functional modules."
- [discussion] protein complexes tend to form more topological modules than metabolic pathways: "As a result of incomplete knowledge in large reference interactomes, protein complexes tend to form more topological modules than metabolic pathways"
