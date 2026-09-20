---
name: protein-network-community-annotation
description: Use when you have a context-specific subnetwork (e.g., pyPARAGON patient-cluster-specific
  networks from 105 breast cancer tumors) and need functional modules with over-represented
  pathways or biological processes for stratification-level interpretation.
when_to_use_negative:
- Input is a flat omics feature table (e.g., raw phosphoproteomic intensities) rather
  than a network — network inference (GGN construction, PageRank flux) must happen
  first
- The goal is causal/edge-level interpretation of relations within modules rather
  than enrichment of module annotations — the article notes biological interpretation
  through causal relations is challenging and generic processes like transcription/replication
  can dominate inferred networks
- Single-cell omics data are the input — the current pyPARAGON version integrates
  bulk omic data for static tumor-specific network models only
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: pyPARAGON
  role: 'Provides the community analysis module: Louvain community detection plus
    hypergeometric over-representation testing of modules against KEGG/GOA annotations'
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Runtime environment for pyPARAGON (python3 virtualenv-based installation and
    package usage)
- name: KEGG
  role: Prior-knowledge pathway annotation source used for hypergeometric over-representation
    tests of inferred modules
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:9aae4430963c15bcb6a43a85fe794e3eee7c2a938551a919fb617015c846c8ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - KEGG
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# protein-network-community-annotation

## Summary

Partitions an inferred context-specific protein network into functional modules using Louvain community detection and annotates each module with over-represented prior-knowledge terms (e.g., KEGG pathways) via hypergeometric testing, enabling biological interpretation such as pathway over-representation frequency across patient clusters.

## When to use

Use when you have a context-specific subnetwork (e.g., pyPARAGON patient-cluster-specific networks from 105 breast cancer tumors) and need functional modules with over-represented pathways or biological processes for stratification-level interpretation.

## When NOT to use

- Input is a flat omics feature table (e.g., raw phosphoproteomic intensities) rather than a network — network inference (GGN construction, PageRank flux) must happen first
- The goal is causal/edge-level interpretation of relations within modules rather than enrichment of module annotations — the article notes biological interpretation through causal relations is challenging and generic processes like transcription/replication can dominate inferred networks
- Single-cell omics data are the input — the current pyPARAGON version integrates bulk omic data for static tumor-specific network models only

## Inputs

- Context-specific inferred subnetwork(s) (e.g., pyPARAGON patient-cluster-specific networks for 105 breast cancer tumors)
- Prior-knowledge annotation sets (KEGG pathways, GOA biological processes, TTP drug knowledge)
- Any protein interaction network with communities to be annotated (community analysis works independently of the inference module)

## Outputs

- Functional modules (Louvain communities) per network
- Significantly over-represented pathway/process annotations per module (hypergeometric test results)
- Aggregated pathway over-representation frequency table per patient cluster (e.g., pathway_overrepresentation_table.csv reproducing Fig. 4D)
- Extracted modules of interest (e.g., Ras-signaling module containing KRAS, NRAS, HRAS, RHOA, RHOD)

## How to apply

Load the inferred subnetwork(s) into the pyPARAGON community analysis module, which is independent of the network-inference module and can run on any network. Run Louvain community detection based on network topology to divide the network into functional units (modules). For each community, apply a hypergeometric test against a prior-knowledge annotation set — such as KEGG pathways or GOA biological processes — to identify significantly over-represented terms; the choice of annotation source (GOA, KEGG, TTP) is a decision point driven by the interpretation goal. For multi-cluster studies (e.g., pyPARAGON patient clusters of breast cancer phosphoproteomic data from 105 CPTAC tumors), aggregate significant pathways per cluster and tabulate over-representation frequencies across clusters (as in Fig. 4D), and optionally extract specific modules (e.g., the Ras-signaling module linking cluster phosphoproteins to KRAS, NRAS, HRAS, RHOA, and RHOD). The same annotations-integrated-with-modules approach supports patient stratification, survival analysis, and personalized medicine.

## Related tools

- **pyPARAGON** (Provides the community analysis module: Louvain community detection plus hypergeometric over-representation testing of modules against KEGG/GOA annotations) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Runtime environment for pyPARAGON (python3 virtualenv-based installation and package usage))
- **KEGG** (Prior-knowledge pathway annotation source used for hypergeometric over-representation tests of inferred modules)

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- Output pathway over-representation table exists as a structured CSV (e.g., pathway_overrepresentation_table.csv) with per-cluster frequency entries
- Per-cluster pathway over-representation frequencies are consistent with the distribution reported in Fig. 4D of the pyPARAGON paper (cell cycle and PI3K/Akt signaling most frequent, less frequent in cluster-2)
- Louvain communities are non-trivial partitions of the inferred network based on topology (multiple functional units, not one giant community)
- Extracted module of interest (e.g., Ras-signaling module) contains the expected intermediate nodes KRAS, NRAS, HRAS, RHOA, and RHOD
- Hypergeometric-test annotations are drawn from the specified prior-knowledge source (KEGG pathways / GOA biological processes), matching the chosen reference set

## Limitations

- Inferred networks frequently contain generic biological processes such as transcription and replication, which can hinder interpretation of annotated modules
- Incomplete reference interactomes bias inferred networks toward protein complexes, so topological modules may over-represent complexes rather than metabolic pathways
- Hub proteins may dominate inferred networks with unrelated interactions (pyPARAGON counters this upstream via flux calculation rather than hub penalization)
- Information about protein isoforms and post-translational modifications is generalized and potentially lost in the network

## Evidence

- [methods] The community analysis module recruits the Louvain community detection method based on network topology (Figure 3).: "The community analysis module recruits the Louvain community detection method based on network topology (Figure 3)."
- [readme] pyPARAGON hypergeometric module testing: "Then, pyPARAGON can identify significant modules by applying a hypergeometric test to a given prior knowledge, such as biological processes, pathways."
- [discussion] patient clustering from module annotations: "Patients were clustered based on the network-based similarity between the overrepresented biological processes identified with functional modules."
- [discussion] annotation sources GOA KEGG TTP: "such as biological processes in GOA, pathways in KEGG, and drug knowledge in TTP"
