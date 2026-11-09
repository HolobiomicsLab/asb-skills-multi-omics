---
name: cytoscape-network-automation-via-rcy3
description: Use when the goal is to reproduce or redraw a multi-layered trans-omic
  network (e.g., KEGG rno04910 over two rno01100 layers, or the five-layer mmu04910/mmu01100
  glucose-response network) automatically from tabular connectivity data rather than
  manual Cytoscape editing.
when_to_use_negative:
- The pathway source is Reactome, Wikipathways, or SBGN/other non-KEGG formats — transomics2cytoscape
  currently supports only the KEGG ID and pathway file format
- Inter-layer (trans-omic) connectivity has not yet been reconstructed and must be
  fetched from databases such as BRENDA, OmniPath, SIGNOR, or IntAct — retrieval of
  inter-layer edges is not automated
- A single 2D pathway needs only standard import/analysis with no layer stacking or
  cross-layer edges
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: transomics2cytoscape
  role: R Bioconductor package that orchestrates the automated 2.5D network construction
    (create3Dnetwork layer stacking, createTransomicEdges inter-layer edge drawing)
  repo: https://github.com/ecell/transomics2cytoscape
- name: RCy3
  role: R interface (v2.24.0) used to drive Cytoscape automation, including network-editing
    commands that insert inter-layer edges
- name: Cytoscape
  role: Network rendering engine (3.8.0 installed per methods; vignette validated
    on 3.10.3) hosting the 2.5D network session
  repo: https://github.com/cytoscape/cytoscape
- name: KEGGREST
  role: Retrieves KGML (KEGG pathway XML) for the pathway IDs listed in the layer-definition
    table
- name: KEGGscape
  role: Cytoscape app (0.9.2) that imports each KGML while preserving the conventional
    pathway node/edge layout
- name: Cy3D
  role: Cytoscape app (1.1.3) used to visualize/validate the stacked 2.5D layers in
    3D space
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_005
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:98138fe0b64e91e38a7cb876e126d5d6ad621658a82014c97f9545b71c7baf52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ecell/transomics2cytoscape
  tools:
  - Cy3D
  - Cytoscape
  - KEGGREST
  - KEGGscape
  - R
  - RCy3
  - transomics2cytoscape
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# cytoscape-network-automation-via-rcy3

## Summary

Automates construction of 2.5D trans-omic networks by driving Cytoscape 3.10.3 from R (RCy3 2.24.0) through the transomics2cytoscape package, which retrieves KEGG KGML via KEGGREST, imports pathway layers via KEGGscape, stacks them at Z-coordinates rendered by Cy3D, and adds inter-layer edges from tabular connectivity files.

## When to use

Use when the goal is to reproduce or redraw a multi-layered trans-omic network (e.g., KEGG rno04910 over two rno01100 layers, or the five-layer mmu04910/mmu01100 glucose-response network) automatically from tabular connectivity data rather than manual Cytoscape editing.

## When NOT to use

- The pathway source is Reactome, Wikipathways, or SBGN/other non-KEGG formats — transomics2cytoscape currently supports only the KEGG ID and pathway file format
- Inter-layer (trans-omic) connectivity has not yet been reconstructed and must be fetched from databases such as BRENDA, OmniPath, SIGNOR, or IntAct — retrieval of inter-layer edges is not automated
- A single 2D pathway needs only standard import/analysis with no layer stacking or cross-layer edges

## Inputs

- Four-column layer-definition TSV (layer index; KEGG pathway ID or Cytoscape-compatible pathway file; Z-coordinate; edge-midpoint flag)
- Seven-column trans-omic interaction TSV (source layer index, source node-table column, source attribute value, target layer index, target node-table column, target attribute value, interaction type)
- KEGG pathway IDs (e.g., rno04910, rno01100, mmu04910, mmu01100) resolved to KGML via KEGGREST
- Deposited connectivity tables from Zenodo doi:10.5281/zenodo.8201898

## Outputs

- Cytoscape session file (.cys) containing one network per requested layer stacked at the specified Z-coordinates with inter-layer edges
- 2.5D network visualization corresponding to Figs. 4-5 of the paper
- Timing measurements (e.g., runtime.txt, drawing_time_table.csv) when benchmarking layer stacking

## How to apply

Install OpenJDK 11 and Cytoscape 3.8.0+ (vignette validated against 3.10.3), launch it with the KEGGscape 0.9.2 and Cy3D 1.1.3 apps, and connect from R 4.4.1 / Bioconductor 3.19 via RCy3 2.24.0. Provide a four-column layer-definition TSV (layer index, KEGG pathway ID or local pathway file, Z-coordinate, edge-midpoint flag); for KEGG IDs, KEGGREST retrieves the KGML and KEGGscape imports each map preserving its conventional node/edge layout, while the Z-height positions each layer in 2.5D space. Provide a 7-column trans-omic interaction TSV whose first three columns define the source node (layer index, node-table column name, attribute value), columns 4-6 the target node the same way, and the last column the interaction type written into the edge table's 'interaction' column; convert molecule IDs to KEGG-compatible IDs (KEGG gene IDs, EC numbers, e.g., via Ensembl BioMart) so rows resolve against node tables. The decision point of where to set Z-coordinates is not automated — the user supplies them — and where the edge-midpoint flag is set an extra node is inserted at the edge midpoint. Save the result as a .cys session (e.g., usecase1_2.5d_network.cys, usecase2_network.cys) and validate the stacked layout against the paper's figures.

## Related tools

- **transomics2cytoscape** (R Bioconductor package that orchestrates the automated 2.5D network construction (create3Dnetwork layer stacking, createTransomicEdges inter-layer edge drawing)) — https://github.com/ecell/transomics2cytoscape
- **RCy3** (R interface (v2.24.0) used to drive Cytoscape automation, including network-editing commands that insert inter-layer edges)
- **Cytoscape** (Network rendering engine (3.8.0 installed per methods; vignette validated on 3.10.3) hosting the 2.5D network session) — https://github.com/cytoscape/cytoscape
- **KEGGREST** (Retrieves KGML (KEGG pathway XML) for the pathway IDs listed in the layer-definition table)
- **KEGGscape** (Cytoscape app (0.9.2) that imports each KGML while preserving the conventional pathway node/edge layout)
- **Cy3D** (Cytoscape app (1.1.3) used to visualize/validate the stacked 2.5D layers in 3D space)

## Examples

```
wget https://github.com/cytoscape/cytoscape/releases/download/3.8.0/cytoscape-unix-3.8.0.tar.gz
```

## Evaluation signals

- RCy3 connection check: R can drive the running Cytoscape instance (RCy3 2.24.0 against Cytoscape 3.10.3 with KEGGscape 0.9.2 and Cy3D 1.1.3 installed)
- The saved .cys session contains one network per requested layer, each placed at its specified Z-coordinate, with inter-layer edges present
- Every row of the 7-column trans-omic interaction TSV yields exactly one edge with the correct interaction type and endpoints
- Rendered 2.5D network reproduces the paper's layer configuration: Fig. 4 (rno04910 over two rno01100 layers) or Fig. 5 (five-layer WT/ob-ob liver glucose-response network)
- Case Study I completes within a few minutes (paper reports ~3 minutes on a Windows 11 i7-12700/128 GB machine), and drawing time grows approximately linearly for n = 2-6 stacked layers

## Limitations

- Estimation of Z-coordinates in Case Study I and II is not automated; the user supplies layer Z-heights
- Connectivity data of trans-omic networks are provided after reconstruction procedures and are not fully automatically reconstructed
- Only the KEGG ID and pathway file format are supported; Reactome, Wikipathways, and SBGN are not supported yet
- Some nodes and edges are hidden under the layer planes in 2.5D visualization
- Retrieval of inter-layer edges from databases (BRENDA, OmniPath, SIGNOR, IntAct) is not automated

## Evidence

- [discussion] Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML).: "Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML)."
- [discussion] The transomics2cytoscape package enables automatic visualization of 2.5D networks in conventional pathway layouts.: "The transomics2cytoscape package enables automatic visualization of 2.5D networks in conventional pathway layouts."
- [intro] We specified four requirements for the human-interpretable and reproducible visualization of multi-layered biochemical networks: "We specified four requirements for the human-interpretable and reproducible visualization of multi-layered biochemical networks"
- [results] The trans-omic interaction table defines edges connecting layers: its first three columns specify the source node (layer index, node-table column name, attribute value), the fourth to sixth columns specify the target node the same way, and the last column gives the interaction type: [evidence span withheld — anchor status: not_found]
- [readme] an R package that automatically creates 3D network visualization in combination with Cytoscape: "Here we propose **transomics2cytoscape**, an R package that automatically creates
3D network visualization in combination with
Cytoscape"
