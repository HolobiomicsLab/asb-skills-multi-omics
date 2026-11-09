---
name: multi-layer-2-5d-network-stacking
description: Use when multi-omics connectivity tables must be visualized as stacked
  pathway layers in 3D (2.5D) space with regulatory edges between omic layers, e.g.,
  reproducing Yugi/Kokaji-style trans-omic network figures.
when_to_use_negative:
- Pathway layers come from non-KEGG sources such as Reactome, Wikipathways, or SBGN
  — only KEGG ID and pathway file format are currently supported
- You expect automated retrieval of inter-layer edges from databases (BRENDA, OmniPath,
  SIGNOR, IntAct) or automated Z-coordinate estimation — these steps are not automated
  and must be supplied manually
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0092
tools:
- name: transomics2cytoscape
  role: R Bioconductor package that orchestrates layer import, stacking at Z-coordinates,
    and inter-layer edge insertion
  repo: https://github.com/ecell/transomics2cytoscape
- name: KEGGREST
  role: Retrieves KGML (KEGG pathway XML) for each KEGG pathway ID in the layer definition
    table
- name: RCy3
  role: Automates importing multiple pathway maps and integrating them into a 2.5D
    network in Cytoscape
- name: KEGGscape
  role: Cytoscape app used to import KGML maps preserving the conventional node/edge
    layout of each layer
- name: Cy3D
  role: Cytoscape app used to render/validate the stacked 2.5D network in 3D space
- name: Cytoscape
  role: Desktop platform in which the layered network is assembled and saved as a
    .cys session
  repo: https://github.com/cytoscape/cytoscape
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:164afff801351923237f41136eb7027b41e61f70577fce58f07c18d1c43a7a9c
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# multi-layer-2.5d-network-stacking

## Summary

Automated construction of a 2.5D trans-omic network in Cytoscape by stacking conventional 2D pathway maps (from KEGG KGML or local pathway files) as layers at specified Z-coordinates and drawing inter-layer edges between them, preserving each layer's conventional node/edge layout.

## When to use

Use when multi-omics connectivity tables must be visualized as stacked pathway layers in 3D (2.5D) space with regulatory edges between omic layers, e.g., reproducing Yugi/Kokaji-style trans-omic network figures.

## When NOT to use

- Pathway layers come from non-KEGG sources such as Reactome, Wikipathways, or SBGN — only KEGG ID and pathway file format are currently supported
- You expect automated retrieval of inter-layer edges from databases (BRENDA, OmniPath, SIGNOR, IntAct) or automated Z-coordinate estimation — these steps are not automated and must be supplied manually

## Inputs

- Four-column layer-definition TSV (layer index, KEGG pathway ID or local pathway file, Z-coordinate, edge-midpoint flag)
- Trans-omic connectivity tables (e.g., Yugi et al. Table S3/Table S4, Kokaji et al. Data file S11) describing inter-layer edges
- KEGG pathway IDs (KGML retrieval) or Cytoscape-compatible pathway files

## Outputs

- Cytoscape session file transomics2cytoscape.cys containing one network per requested layer stacked at specified Z-coordinates with inter-layer edges
- 2.5D network rendering viewable in the Cy3D app

## How to apply

Read a four-column layer-definition table (layer index, KEGG pathway ID or Cytoscape-compatible pathway file, Z-height, edge-midpoint flag) into R. For each row, retrieve KGML via KEGGREST (for KEGG IDs) or load the supplied pathway file, then use RCy3 with the KEGGscape app to import each map into Cytoscape, conserving the conventional node/edge layout. Apply each row's Z-height as the layer's coordinate in the 2.5D space, visualized with the Cy3D app. Add inter-layer edges connecting molecules across layers; where the edge-midpoint flag is set, insert an extra node at the midpoint of those edges, then save the result as a Cytoscape session file (transomics2cytoscape.cys).

## Related tools

- **transomics2cytoscape** (R Bioconductor package that orchestrates layer import, stacking at Z-coordinates, and inter-layer edge insertion) — https://github.com/ecell/transomics2cytoscape
- **KEGGREST** (Retrieves KGML (KEGG pathway XML) for each KEGG pathway ID in the layer definition table)
- **RCy3** (Automates importing multiple pathway maps and integrating them into a 2.5D network in Cytoscape)
- **KEGGscape** (Cytoscape app used to import KGML maps preserving the conventional node/edge layout of each layer)
- **Cy3D** (Cytoscape app used to render/validate the stacked 2.5D network in 3D space)
- **Cytoscape** (Desktop platform in which the layered network is assembled and saved as a .cys session) — https://github.com/cytoscape/cytoscape

## Evaluation signals

- The output session file transomics2cytoscape.cys exists and loads in Cytoscape without errors
- The session contains one network per row of the layer-definition table, each positioned at its specified Z-coordinate
- Inter-layer edges are present connecting molecules across stacked layers, with midpoint nodes inserted where the edge-midpoint flag is set
- Each layer retains its conventional KEGG node/edge layout (KEGGscape import preserved), verifiable by expert review of the rendered 2.5D network in Cytoscape/Cy3D
- The visualization reproduces published trans-omic networks (e.g., Yugi et al. Tables S3/S4, Kokaji et al. Data file S11) within a few minutes

## Limitations

- Z-coordinate estimation in Case Study I and II is not automated and must be supplied by the user
- Retrieval of inter-layer edges from databases such as BRENDA, OmniPath, SIGNOR, and IntAct is not automated
- Only KEGG ID and pathway file formats are supported; Reactome, Wikipathways, and SBGN are not yet supported
- Some nodes and edges can be hidden under the layer planes in the 2.5D perspective

## Evidence

- [discussion] Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML).: "Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML)."
- [intro] The prior knowledge used to visualize interpretable networks has two components: "The prior knowledge used to visualize interpretable networks has two components: (1) conventional pathway map layout and (2) stacked omic layers with a 2.5D perspective"
- [discussion] Currently, transomics2cytoscape supports only the KEGG ID and pathway file format.: "Currently, transomics2cytoscape supports only the KEGG ID and pathway file format."
- [discussion] We have not yet automated the estimation of Z-coordinates in Case Study I and II.: "We have not yet automated the estimation of Z-coordinates in Case Study I and II."
- [readme] an R package that automatically creates 3D network visualization in combination with Cytoscape: "Here we propose **transomics2cytoscape**, an R package that automatically creates 3D network visualization in combination with Cytoscape"
