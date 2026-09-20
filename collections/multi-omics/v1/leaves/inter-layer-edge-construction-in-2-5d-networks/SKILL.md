---
name: inter-layer-edge-construction-in-2-5d-networks
description: Use when a trans-omic interaction table (source/target layer index, node-table
  column, attribute value, interaction type) must be realized as edges connecting
  stacked omic layers in an existing Cytoscape 2.5D network.
when_to_use_negative:
- The pathway layers themselves (intra-layer edges from KGML/pathway maps) still need
  to be imported and stacked — this skill only adds edges between already-stacked
  layers
- You expect automated retrieval of inter-layer edges from interaction databases (BRENDA,
  OmniPath, SIGNOR, IntAct) — transomics2cytoscape has not automated this step, the
  table must be supplied
- Nodes use non-KEGG identifiers — only the KEGG ID and pathway file format is currently
  supported, so IDs must first be converted to KEGG-compatible IDs (e.g. KEGG gene
  ID, EC numbers via Ensembl BioMart)
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: transomics2cytoscape
  role: R/Bioconductor package that parses the trans-omic interaction table and creates
    the inter-layer edges (createTransomicEdges) in the 2.5D network
  repo: https://github.com/ecell/transomics2cytoscape
- name: RCy3
  role: R interface used to drive the Cytoscape instance and execute the network-editing
    commands that add the inter-layer edges and annotate interaction types
- name: Cytoscape
  role: Host application holding the stacked 2.5D network (with Cy3D App) into which
    the trans-omic edges are inserted
  repo: https://github.com/cytoscape/cytoscape
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:97f2e1995c10c45e447a4f6f9260c3fa6fb10e9b4644c0b017c7bac0c8cbabf8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ecell/transomics2cytoscape
  tools:
  - Cytoscape
  - R
  - RCy3
  - transomics2cytoscape
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# inter-layer-edge-construction-in-2.5d-networks

## Summary

Reconstructs and draws trans-omic (inter-layer) edges between stacked pathway layers in a 2.5D Cytoscape network, using a 7-column trans-omic interaction TSV and the transomics2cytoscape/RCy3 toolchain to satisfy the article's requirement (3) 'drawing edges between stacked layers'.

## When to use

Use when a trans-omic interaction table (source/target layer index, node-table column, attribute value, interaction type) must be realized as edges connecting stacked omic layers in an existing Cytoscape 2.5D network.

## When NOT to use

- The pathway layers themselves (intra-layer edges from KGML/pathway maps) still need to be imported and stacked — this skill only adds edges between already-stacked layers
- You expect automated retrieval of inter-layer edges from interaction databases (BRENDA, OmniPath, SIGNOR, IntAct) — transomics2cytoscape has not automated this step, the table must be supplied
- Nodes use non-KEGG identifiers — only the KEGG ID and pathway file format is currently supported, so IDs must first be converted to KEGG-compatible IDs (e.g. KEGG gene ID, EC numbers via Ensembl BioMart)

## Inputs

- 7-column trans-omic interaction TSV (source layer index, source node-table column, source attribute value, target layer index, target node-table column, target attribute value, interaction type)
- Existing layered 2.5D network in a running Cytoscape instance (built from stacked pathway layers, KEGG ID-compatible nodes)

## Outputs

- Cytoscape network with one inter-layer edge per interaction row, interaction type annotated in the edge table's 'interaction' column
- Saved Cytoscape session file (e.g. transomic_edges.cys)

## How to apply

Load the 7-column trans-omic interaction TSV — columns 1–3 specify the source node (layer index, node-table column name, attribute value), columns 4–6 specify the target node the same way, and column 7 gives the interaction type. Load the previously built layered 2.5D network (stacked pathway layers integrated in Cytoscape, e.g. 3.10.3) via RCy3 (2.24.0). For each interaction row, resolve the source and target nodes by matching the specified node-table column against the attribute value within the specified layer, then add the inter-layer edge — inserting edge-midpoint nodes where required — with the interaction type written into the edge table's 'interaction' column using RCy3 network-editing commands. The rationale is that the trans-omic interaction table is the tabular contract from which connectivity between pathway layers is reproducibly reconstructed, since transomics2cytoscape does not yet automate retrieval of inter-layer edges from databases such as BRENDA, OmniPath, SIGNOR, or IntAct.

## Related tools

- **transomics2cytoscape** (R/Bioconductor package that parses the trans-omic interaction table and creates the inter-layer edges (createTransomicEdges) in the 2.5D network) — https://github.com/ecell/transomics2cytoscape
- **RCy3** (R interface used to drive the Cytoscape instance and execute the network-editing commands that add the inter-layer edges and annotate interaction types)
- **Cytoscape** (Host application holding the stacked 2.5D network (with Cy3D App) into which the trans-omic edges are inserted) — https://github.com/cytoscape/cytoscape

## Examples

```
transomics2cytoscape::createTransomicEdges(transomic_edges, layer_list)
```

## Evaluation signals

- Every row of the trans-omic interaction TSV yields exactly one edge in the Cytoscape network with the correct interaction type and endpoints
- Each created edge's 'interaction' column in the edge table matches the interaction type in column 7 of the TSV
- Source and target endpoints resolve to nodes found by matching the specified node-table column against the attribute value within the specified layer (no unresolved/NULL SUIDs)
- Resulting session saves cleanly as transomic_edges.cys and reopens with the inter-layer edges intact

## Limitations

- Retrieval of inter-layer edges from databases such as BRENDA, OmniPath, SIGNOR, and IntAct is not automated; the trans-omic interaction table must be supplied manually
- Currently only the KEGG ID and pathway file format is supported; Reactome, Wikipathways, and SBGN are not supported
- Connectivity data of trans-omic networks are provided after reconstruction procedures that are not fully automated

## Evidence

- [discussion] The transomics2cytoscape package adds edges between layers to understand molecular interactions between pathway layers: "The transomics2cytoscape package adds edges between layers to understand molecular interactions between pathway layers"
- [discussion] transomics2cytoscape has not yet automated the retrieval processes for inter-layer edges.: "transomics2cytoscape has not yet automated the retrieval processes for inter-layer edges."
- [discussion] Currently, transomics2cytoscape supports only the KEGG ID and pathway file format.: "Currently, transomics2cytoscape supports only the KEGG ID and pathway file format."
- [discussion] The IDs of molecules were converted into KEGG–compatible IDs such as KEGG gene ID and EC numbers by using Ensembl BioMart: "The IDs of molecules were converted into KEGG–compatible IDs such as KEGG gene ID and EC numbers by using Ensembl BioMart42"
- [readme] an R package that automatically creates 3D network visualization in combination with Cytoscape: "Here we propose **transomics2cytoscape**, an R package that automatically creates 3D network visualization in combination with
Cytoscape"
