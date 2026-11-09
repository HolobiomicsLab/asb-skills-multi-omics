---
name: multi-layer-node-matching-by-node-table-attributes
description: Use when a 7-column trans-omic interaction table (source/target layer
  index, node-table column, attribute value, interaction type) must be connected onto
  an already-stacked multi-layer Cytoscape network.
when_to_use_negative:
- The inter-layer interactions have not been reconstructed yet — the article states
  retrieval of inter-layer edges from databases such as BRENDA, OmniPath, SIGNOR,
  and IntAct is not automated, so those must be prepared first
- Node identifiers in the layers are not KEGG-compatible IDs — transomics2cytoscape
  currently supports only the KEGG ID and pathway file format
- You need a fully 3D-rendered VR export rather than a 2.5D Cytoscape network — Cy2VR
  handles that separate conversion
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3391
tools:
- name: transomics2cytoscape
  role: Provides the trans-omic interaction file contract and the inter-layer edge-drawing
    logic (createTransomicEdges module) executed from R
  repo: https://github.com/ecell/transomics2cytoscape
- name: RCy3
  role: R interface used to drive Cytoscape and perform the network-editing commands
    that add inter-layer edges and annotate the interaction column
- name: Cytoscape
  role: Hosts the stacked 2.5D network whose node tables are matched against the trans-omic
    interaction table's attribute values
  repo: https://github.com/cytoscape/cytoscape
- name: KEGGREST
  role: Retrieves KGML pathway XML that defines the stacked pathway layers whose node
    tables supply the attribute columns used for matching
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:3847281ad79984aa1be4e7f115e60575e0da9c78944879d5813604aad9da477d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ecell/transomics2cytoscape
  tools:
  - Cytoscape
  - KEGGREST
  - R
  - RCy3
  - transomics2cytoscape
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# Multi-layer node matching by node-table attributes

## Summary

Resolves source and target nodes of trans-omic (inter-layer) interactions inside a stacked 2.5D Cytoscape network by matching attribute values in per-layer node tables, then draws the inter-layer edges with annotated interaction types via transomics2cytoscape/RCy3.

## When to use

Use when a 7-column trans-omic interaction table (source/target layer index, node-table column, attribute value, interaction type) must be connected onto an already-stacked multi-layer Cytoscape network.

## When NOT to use

- The inter-layer interactions have not been reconstructed yet — the article states retrieval of inter-layer edges from databases such as BRENDA, OmniPath, SIGNOR, and IntAct is not automated, so those must be prepared first
- Node identifiers in the layers are not KEGG-compatible IDs — transomics2cytoscape currently supports only the KEGG ID and pathway file format
- You need a fully 3D-rendered VR export rather than a 2.5D Cytoscape network — Cy2VR handles that separate conversion

## Inputs

- 7-column trans-omic interaction TSV (source layer index, source node-table column, source attribute value, target layer index, target node-table column, target attribute value, interaction type)
- Stacked 2.5D multi-layer Cytoscape network with per-layer node tables (KEGG-compatible IDs such as KEGG gene IDs and EC numbers)
- Running Cytoscape (e.g., 3.10.3) instance controlled via RCy3 (e.g., 2.24.0)

## Outputs

- Inter-layer edges in the Cytoscape network connecting matched nodes across stacked pathway layers, one per trans-omic interaction row
- Edge table populated with the interaction type in the 'interaction' column
- Saved Cytoscape session file (e.g., transomic_edges.cys)

## How to apply

Load the 7-column trans-omic interaction TSV whose first three columns identify the source node (layer index, node-table column name, attribute value) and whose fourth-to-sixth columns identify the target node the same way, with the last column giving the interaction type. Load the layered 2.5D network into a running Cytoscape instance driven by RCy3. For each row, resolve the source and target node by matching the specified node-table column against the attribute value within the specified layer, then add the inter-layer edge — inserting edge-midpoint nodes where the interaction requires them — with the interaction type written into the edge table's 'interaction' column. Node identifiers should be KEGG-compatible (e.g., KEGG gene ID, EC numbers) since transomics2cytoscape currently supports only the KEGG ID and pathway file format. Validate that each TSV row yields exactly one correctly typed edge with the expected endpoints before saving the session.

## Related tools

- **transomics2cytoscape** (Provides the trans-omic interaction file contract and the inter-layer edge-drawing logic (createTransomicEdges module) executed from R) — https://github.com/ecell/transomics2cytoscape
- **RCy3** (R interface used to drive Cytoscape and perform the network-editing commands that add inter-layer edges and annotate the interaction column)
- **Cytoscape** (Hosts the stacked 2.5D network whose node tables are matched against the trans-omic interaction table's attribute values) — https://github.com/cytoscape/cytoscape
- **KEGGREST** (Retrieves KGML pathway XML that defines the stacked pathway layers whose node tables supply the attribute columns used for matching)

## Evaluation signals

- Every row of the trans-omic interaction TSV yields exactly one edge in the resulting Cytoscape network
- Each added edge has the correct interaction type recorded in the edge table's 'interaction' column
- Each added edge connects the endpoints resolved by matching the specified node-table column/attribute value within the specified layer
- The stacked-layer topology (2.5D arrangement) is preserved after edge insertion, consistent with conserving the conventional layout of nodes and edges in the pathway layers
- Session saves successfully as transomic_edges.cys and reopens with all inter-layer edges present

## Limitations

- Retrieval of inter-layer edges from databases (BRENDA, OmniPath, SIGNOR, IntAct) is not automated; the connectivity data of trans-omic networks are not fully automatically reconstructed
- Only the KEGG ID and pathway file format are currently supported; Reactome, Wikipathways, and SBGN are not yet supported
- Some nodes and edges are hidden under the layer planes in the 2.5D visualization

## Evidence

- [discussion] The transomics2cytoscape package adds edges between layers to understand molecular interactions between pathway layers: "The transomics2cytoscape package adds edges between layers to understand molecular interactions between pathway layers"
- [intro] tabular input contract: "Here, we present an R package, transomics2cytoscape, that automates the visualization of trans-omic networks from tabular data describing network connectivity."
- [discussion] KEGG-compatible ID requirement: "The IDs of molecules were converted into KEGG–compatible IDs such as KEGG gene ID and EC numbers by using Ensembl BioMart42"
- [readme] automation stack in README: [evidence span withheld — anchor status: external_doc]
- [discussion] ID format scope limitation: "Currently, transomics2cytoscape supports only the KEGG ID and pathway file format."
