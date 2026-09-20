---
name: kegg-pathway-overrepresentation-analysis
description: Use when patient- or condition-specific inferred networks must be partitioned
  into functional modules and tested for KEGG pathway over-representation to reproduce
  cluster-level pathway frequency analyses (e.g., Fig. 4D Ras-module style extraction).
when_to_use_negative:
- Input is a raw omics data matrix rather than an inferred context-specific network
  — network inference must run first (GGN construction, PPR flux scoring, subnetwork
  reconstruction).
- The reference annotation space is not KEGG/GO/TP-style prior knowledge sets — hypergeometric
  testing requires a defined annotation universe.
- Single-cell omics data — the current pyPARAGON version integrates bulk omic data
  for static tumor-specific network models only.
edam_operation: http://edamontology.org/operation_3636
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0602
tools:
- name: pyPARAGON
  role: 'Runs the community analysis module: Louvain community detection on cluster-specific
    subnetworks and hypergeometric testing of modules against KEGG pathway annotations'
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Execution environment; pyPARAGON is installed as a Python package via pip
  repo: https://github.com/metunetlab/pyPARAGON
- name: KEGG
  role: Pathway annotation prior knowledge against which module over-representation
    is tested
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:51b4dfe6d351bcfaaf1d812176e949af14567803bccdc8e52442c5435c7cfe4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - KEGG
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_3636
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# kegg-pathway-overrepresentation-analysis

## Summary

Annotates functional modules from context-specific networks (e.g., pyPARAGON patient-cluster-specific subnetworks for 105 breast cancer tumors) with KEGG pathways via hypergeometric over-representation testing, then aggregates per-cluster pathway frequencies to identify recurrently over-represented pathways such as cell cycle and PI3K/Akt signaling.

## When to use

Use when patient- or condition-specific inferred networks must be partitioned into functional modules and tested for KEGG pathway over-representation to reproduce cluster-level pathway frequency analyses (e.g., Fig. 4D Ras-module style extraction).

## When NOT to use

- Input is a raw omics data matrix rather than an inferred context-specific network — network inference must run first (GGN construction, PPR flux scoring, subnetwork reconstruction).
- The reference annotation space is not KEGG/GO/TP-style prior knowledge sets — hypergeometric testing requires a defined annotation universe.
- Single-cell omics data — the current pyPARAGON version integrates bulk omic data for static tumor-specific network models only.

## Inputs

- Patient-cluster-specific subnetworks from pyPARAGON network inference (105 breast cancer tumors with phosphoproteomic data)
- KEGG pathway annotation sets (prior knowledge), optionally GO biological process and TTP annotations
- List of intermediate nodes for targeted module extraction (KRAS, NRAS, HRAS, RHOA, RHOD)

## Outputs

- pathway_overrepresentation_table.csv — aggregated KEGG pathway over-representation frequencies per patient cluster
- Per-module significant KEGG pathway assignments from hypergeometric tests
- Extracted Ras-signaling module (subnetwork containing KRAS, NRAS, HRAS, RHOA, RHOD)

## How to apply

Load the patient-cluster-specific subnetworks (upstream pyPARAGON network-inference output for the 105-tumor breast cancer phosphoproteomic stratification) into the pyPARAGON community analysis module, which recruits Louvain community detection based on network topology to divide each network into functional units. For each community, apply a hypergeometric test against KEGG pathway annotations to identify significantly over-represented pathways; the same module supports prior knowledge sets such as GO biological processes, KEGG pathways, and TTP drug knowledge. Aggregate significant pathways per cluster and tabulate over-representation frequencies across patient clusters — in the breast cancer case, cell cycle and PI3K/Akt signaling emerged as the most frequent pathways, except in cluster-2 where they are less frequent. Separately, extract the Ras-signaling module connecting cluster phosphoproteins to intermediate nodes KRAS, NRAS, HRAS, RHOA, and RHOD. Save the aggregated results as a structured CSV (pathway_overrepresentation_table.csv) for downstream comparison against the published frequency distribution.

## Related tools

- **pyPARAGON** (Runs the community analysis module: Louvain community detection on cluster-specific subnetworks and hypergeometric testing of modules against KEGG pathway annotations) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Execution environment; pyPARAGON is installed as a Python package via pip) — https://github.com/metunetlab/pyPARAGON
- **KEGG** (Pathway annotation prior knowledge against which module over-representation is tested)

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- pathway_overrepresentation_table.csv exists and is a well-structured CSV with pathway over-representation frequencies per cluster.
- Per-cluster pathway frequency distribution is consistent with the published Fig. 4D distribution (cell cycle and PI3K/Akt most frequent; less frequent in cluster-2).
- Extracted Ras-signaling module contains all five intermediate nodes: KRAS, NRAS, HRAS, RHOA, and RHOD.
- Each significant module pathway is traceable to a hypergeometric test on a Louvain community of the cluster-specific network.

## Limitations

- Incomplete reference interactomes bias inferred networks toward protein complexes, so topological modules over-represent generic processes such as transcription and replication rather than metabolic pathways.
- Information about protein isoforms and post-translational modifications is generalized and potentially lost in the network.
- Hub proteins may dominate inferred networks with unrelated interactions, affecting module composition.

## Evidence

- [methods] Louvain community detection with hypergeometric KEGG testing: "The community analysis module recruits the Louvain community detection method based on network topology (Figure 3)."
- [readme] Hypergeometric test against prior knowledge: "Then, pyPARAGON can identify significant modules by applying a hypergeometric test to a given prior knowledge, such as biological processes, pathways."
- [discussion] Prior knowledge annotation sources: "such as biological processes in GOA, pathways in KEGG, and drug knowledge in TTP"
- [intro] Breast cancer application dataset: "We demonstrated the utility of pyPARAGON in patient stratification using a breast cancer dataset comprising 105 tumors and associated phosphoproteomic data"
- [results] Over-representation finding: [evidence span withheld — anchor status: not_found]
