---
name: edge-midpoint-node-insertion
description: Use when you have a trans-omic interaction TSV whose rows connect nodes
  in different stacked pathway layers of a Cytoscape 2.5D network and need those inter-layer
  edges drawn programmatically.
when_to_use_negative:
- Your network is a single flat pathway layer with no stacked omic layers — transomics2cytoscape
  targets 2.5D stacked-layer networks
- You need to retrieve inter-layer edges directly from databases (BRENDA, OmniPath,
  SIGNOR, IntAct) — the package has not automated the retrieval processes for inter-layer
  edges; provide a pre-built interaction TSV
- Your pathway data is not in KEGG ID / KGML format — 'Currently, transomics2cytoscape
  supports only the KEGG ID and pathway file format'
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: transomics2cytoscape
  role: R Bioconductor package implementing the trans-omic interaction file contract
    and edge drawing between stacked layers
  repo: https://github.com/ecell/transomics2cytoscape
- name: RCy3
  role: R interface used to automate importing pathway maps and adding inter-layer
    edges via Cytoscape Automation network-editing commands
- name: Cytoscape
  role: Host application in which the 2.5D network is visualized and inter-layer edges
    are added
  repo: https://github.com/cytoscape/cytoscape
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:0f3161cbb01e8a4a816f9f12cf3c089abd5b1506eeae1b8fa9806892b679594d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ecell/transomics2cytoscape
  tools:
  - Cytoscape
  - R
  - RCy3
  - transomics2cytoscape
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# edge-midpoint-node-insertion

## Summary

A transomics2cytoscape method for realizing requirement (3) 'drawing edges between stacked layers': it parses the 7-column trans-omic interaction table and inserts inter-layer edges — with edge-midpoint nodes where required — into a 2.5D Cytoscape network via RCy3.

## When to use

Use when you have a trans-omic interaction TSV whose rows connect nodes in different stacked pathway layers of a Cytoscape 2.5D network and need those inter-layer edges drawn programmatically.

## When NOT to use

- Your network is a single flat pathway layer with no stacked omic layers — transomics2cytoscape targets 2.5D stacked-layer networks
- You need to retrieve inter-layer edges directly from databases (BRENDA, OmniPath, SIGNOR, IntAct) — the package has not automated the retrieval processes for inter-layer edges; provide a pre-built interaction TSV
- Your pathway data is not in KEGG ID / KGML format — 'Currently, transomics2cytoscape supports only the KEGG ID and pathway file format'

## Inputs

- 7-column trans-omic interaction TSV (source layer index, source node-table column, source attribute value, target layer index, target node-table column, target attribute value, interaction type)
- Layered 2.5D network (stacked pathway layers) in a running Cytoscape instance controlled via RCy3

## Outputs

- Inter-layer edges between stacked layers, with edge-midpoint nodes where required
- Edge table annotated with interaction types in the 'interaction' column
- Saved Cytoscape session file (e.g., transomic_edges.cys)

## How to apply

Load the 7-column trans-omic interaction TSV, where the first three columns specify the source node (layer index, node-table column name, attribute value), the fourth to sixth specify the target node the same way, and the last column gives the interaction type. Load the layered 2.5D network into a running Cytoscape instance (e.g., 3.10.3) driven by RCy3, then for each row resolve the source and target nodes by matching the specified node-table column against the attribute value within the specified layer. Add each inter-layer edge using RCy3 network-editing commands, inserting edge-midpoint nodes where the interaction requires them, and write the interaction type into the edge table's 'interaction' column. Validate that every TSV row yields exactly one edge with the correct endpoints and interaction type, then save the session (e.g., transomic_edges.cys).

## Related tools

- **transomics2cytoscape** (R Bioconductor package implementing the trans-omic interaction file contract and edge drawing between stacked layers) — https://github.com/ecell/transomics2cytoscape
- **RCy3** (R interface used to automate importing pathway maps and adding inter-layer edges via Cytoscape Automation network-editing commands)
- **Cytoscape** (Host application in which the 2.5D network is visualized and inter-layer edges are added) — https://github.com/cytoscape/cytoscape

## Evaluation signals

- Every row of the trans-omic interaction TSV yields exactly one edge in the Cytoscape network
- Each added edge has the correct source and target endpoints resolved from the specified node-table column and attribute value in the specified layer
- Each added edge carries the correct interaction type in the edge table's 'interaction' column
- The final session saves cleanly as transomic_edges.cys and reopens showing the expected 2.5D layered layout

## Limitations

- The connectivity data of trans-omic networks are provided after reconstruction procedures, but these are not fully automated yet
- Retrieval of inter-layer edges from databases such as BRENDA, OmniPath, SIGNOR, and IntAct is not automated; the interaction table must be prepared beforehand
- Currently only the KEGG ID and pathway file format are supported; Reactome, Wikipathways, and SBGN are not supported yet

## Evidence

- [discussion] edges added between layers: "The transomics2cytoscape package adds edges between layers to understand molecular interactions between pathway layers"
- [discussion] 2.5D network integration: [evidence span withheld — anchor status: not_found]
- [readme] package purpose: "Visualization of trans-omic networks helps biological interpretation by illustrating pathways where the signals are transmitted."
