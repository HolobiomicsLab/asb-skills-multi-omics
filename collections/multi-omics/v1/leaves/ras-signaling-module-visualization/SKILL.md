---
name: ras-signaling-module-visualization
description: Use when a pyPARAGON community analysis over patient-cluster-specific
  breast cancer networks (105 tumors) has identified modules, and you must extract
  the Ras-signaling module whose intermediate nodes are KRAS, NRAS, HRAS, RHOA, and
  RHOD for Figure 4E-style visualization.
when_to_use_negative:
- Input is single-cell omic data — pyPARAGON currently integrates only bulk omic data;
  single-cell support is planned for a future version
- The task is to compute KEGG pathway over-representation frequencies across clusters
  (Fig. 4D table) rather than to extract/visualize the specific Ras-signaling module
- No patient-cluster-specific pyPARAGON subnetworks exist yet — network inference
  (GGN construction, PageRank flux, subnetwork reconstruction) must be run first
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: pyPARAGON
  role: Infers the cluster-specific subnetworks and runs the community analysis (Louvain
    + hypergeometric KEGG testing) from which the Ras-signaling module is extracted
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Runtime environment and scripting for installing pyPARAGON, extracting the
    module, and saving the CSV/figure outputs
- name: KEGG
  role: Prior-knowledge pathway annotation source used in the hypergeometric tests
    that flag the Ras-signaling module
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:7de214b7453db61eb6dbbd43783ea8a168fe43e74797e5240aaca9c609693f3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - KEGG
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ras-signaling-module-visualization

## Summary

Extracts and visualizes the Ras-signaling module from pyPARAGON patient-cluster-specific subnetworks of the 105-tumor breast cancer phosphoproteomic stratification, connecting cluster phosphoproteins to the intermediate nodes KRAS, NRAS, HRAS, RHOA, and RHOD to reproduce Figure 4E.

## When to use

Use when a pyPARAGON community analysis over patient-cluster-specific breast cancer networks (105 tumors) has identified modules, and you must extract the Ras-signaling module whose intermediate nodes are KRAS, NRAS, HRAS, RHOA, and RHOD for Figure 4E-style visualization.

## When NOT to use

- Input is single-cell omic data — pyPARAGON currently integrates only bulk omic data; single-cell support is planned for a future version
- The task is to compute KEGG pathway over-representation frequencies across clusters (Fig. 4D table) rather than to extract/visualize the specific Ras-signaling module
- No patient-cluster-specific pyPARAGON subnetworks exist yet — network inference (GGN construction, PageRank flux, subnetwork reconstruction) must be run first

## Inputs

- Patient-cluster-specific pyPARAGON subnetworks (node/edge lists) for the 105-tumor breast cancer stratification
- Phosphoprotein seed/hit lists per patient cluster
- KEGG pathway annotations used for module over-representation testing

## Outputs

- Extracted Ras-signaling module subnetwork containing KRAS, NRAS, HRAS, RHOA, and RHOD as intermediate nodes
- Fig. 4E-style network visualization of the Ras-signaling module per patient cluster
- Companion aggregated pathway over-representation table (pathway_overrepresentation_table.csv)

## How to apply

Load the patient-cluster-specific subnetworks inferred by pyPARAGON (built from 105 breast cancer tumor phosphoproteomic seeds on a reference interactome via GGN construction, Personalized PageRank flux edge scoring, and high-score edge selection). Run the pyPARAGON community analysis module, which partitions each cluster network into functional units using Louvain community detection based on network topology, then annotate communities with prior knowledge such as KEGG pathways via hypergeometric tests to confirm the Ras-signaling module. Within the cluster-specific network, trace and extract the subnetwork connecting the cluster's phosphoprotein seed/hit nodes to the five intermediate nodes KRAS, NRAS, HRAS, RHOA, and RHOD; pyPARAGON's graphlet-guided step already keeps such intermediate (highest-interacting) nodes that bridge hidden connections. Export the extracted module as a node/edge table (e.g., pathway_overrepresentation_table.csv companion output) and render it as a network figure analogous to Fig. 4E. Validation: confirm the extracted module contains all five intermediate nodes and is consistent with the cluster assignment used in the Fig. 4D pathway frequency table.

## Related tools

- **pyPARAGON** (Infers the cluster-specific subnetworks and runs the community analysis (Louvain + hypergeometric KEGG testing) from which the Ras-signaling module is extracted) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Runtime environment and scripting for installing pyPARAGON, extracting the module, and saving the CSV/figure outputs)
- **KEGG** (Prior-knowledge pathway annotation source used in the hypergeometric tests that flag the Ras-signaling module)

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- The extracted module contains all five intermediate nodes: KRAS, NRAS, HRAS, RHOA, and RHOD
- Node/edge membership of the module is a subset of the corresponding patient-cluster-specific subnetwork from the Louvain community analysis
- The module's annotated pathways are consistent with the Fig. 4D over-representation frequency table for that cluster
- Output is a structured CSV (pathway_overrepresentation_table.csv) and a rendered network figure analogous to Fig. 4E
- Hypergeometric-test significance of the module's pathway annotations is reproducible with the same KEGG prior knowledge and module membership

## Limitations

- Reference interactomes are incomplete, so inferred networks (and hence modules) tend to be biased toward protein complexes over metabolic pathways
- Generic biological processes such as transcription and replication can appear more frequently in inferred networks, complicating interpretation
- Hub proteins may dominate the inferred network with unrelated interactions despite pyPARAGON's graphlet/flux countermeasures
- Information about isoforms and post-translational modifications of hits is generalized and potentially lost in the network

## Evidence

- [readme] Louvain module per cluster: "The community analysis module recruits the Louvain community detection method based on network topology (Figure 3)."
- [readme] hypergeometric module testing: "The communities are tested with hypergeometric tests to identify significantly overrepresented annotations."
- [readme] intermediate-node bridging in graphlets: "Each graphlet includes an intermediate node that is the most interacting nodes in the graphlet."
- [abstract] 105-tumor breast cancer stratification context: "integrating phosphoproteomic data from 105 breast cancer tumors with the interactome and demonstrating tumor-specific signaling pathways"
