---
name: trans-omic-inter-layer-edge-construction
description: Use when a stacked 2.5D network of KEGG pathway layers has been imported
  into Cytoscape and connectivity data describing molecular interactions between omic
  layers must be added as inter-layer edges.
when_to_use_negative:
- Your pathway layers are not KEGG pathway IDs or supported pathway files — transomics2cytoscape
  currently supports only the KEGG ID and pathway file format (no Reactome, Wikipathways,
  or SBGN)
- You expect the package to fetch inter-layer edges automatically from BRENDA, OmniPath,
  SIGNOR, or IntAct — retrieval processes for inter-layer edges are not automated
- You need automated Z-coordinate estimation — it is not automated in the Case Study
  I/II workflows
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0092
tools:
- name: transomics2cytoscape
  role: R/Bioconductor package that adds edges between stacked pathway layers and
    integrates them into the 2.5D network in Cytoscape
  repo: https://github.com/ecell/transomics2cytoscape
- name: RCy3
  role: Cytoscape Automation interface used by transomics2cytoscape to import pathway
    maps and wire inter-layer edges in the Cytoscape network
- name: Cytoscape
  role: Hosts the resulting 2.5D trans-omic network in which inter-layer edges are
    rendered
  repo: https://github.com/cytoscape/cytoscape
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:7467a74f407588bd1b953bfab989eeccbb847cca050dec5ea442c321206ffeef
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
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# trans-omic-inter-layer-edge-construction

## Summary

Construction of inter-layer (trans-omic) edges that connect molecules across stacked 2.5D pathway layers in transomics2cytoscape, including optional midpoint nodes for cross-layer edges, so that signal transmission between pathway layers is visible in Cytoscape/Cy3D.

## When to use

Use when a stacked 2.5D network of KEGG pathway layers has been imported into Cytoscape and connectivity data describing molecular interactions between omic layers must be added as inter-layer edges.

## When NOT to use

- Your pathway layers are not KEGG pathway IDs or supported pathway files — transomics2cytoscape currently supports only the KEGG ID and pathway file format (no Reactome, Wikipathways, or SBGN)
- You expect the package to fetch inter-layer edges automatically from BRENDA, OmniPath, SIGNOR, or IntAct — retrieval processes for inter-layer edges are not automated
- You need automated Z-coordinate estimation — it is not automated in the Case Study I/II workflows

## Inputs

- Trans-omic connectivity/edge-list tables describing inter-layer molecular interactions (e.g., Yugi et al. Table S3 and Table S4; Kokaji et al. Data file S11)
- Molecule identifier tables (e.g., Ensembl IDs) to be converted into KEGG-compatible IDs such as KEGG gene ID and EC numbers
- Four-column layer definition table (layer index, KEGG pathway ID or pathway file, Z-coordinate, edge-midpoint flag)
- Cytoscape session with stacked pathway layers created from KGML (via KEGGREST) or local pathway files

## Outputs

- Inter-layer edges connecting molecule nodes across stacked 2.5D pathway layers in Cytoscape
- Midpoint nodes inserted on cross-layer edges where the edge-midpoint flag is set
- Cytoscape session file transomics2cytoscape.cys containing the assembled 2.5D network

## How to apply

After each pathway layer (KGML retrieved from KEGG via KEGGREST, or a local pathway file) is imported into Cytoscape via RCy3 and positioned at its Z-coordinate from the four-column layer definition table (layer index, KEGG ID/file, Z-height, edge-midpoint flag), read the trans-omic connectivity tables (e.g., Yugi et al. Table S3/S4 or Kokaji et al. Data file S11 style edge lists) describing interactions between molecules in different layers. Convert molecule identifiers into KEGG-compatible IDs (KEGG gene IDs, EC numbers) so source/target nodes match nodes in the stacked layers, then use RCy3 automation to add edges between layers in Cytoscape. Where a layer row's edge-midpoint flag is set, insert an extra node at the midpoint of those cross-layer edges so the edge bends visibly between the layer planes. Save the assembled network as the Cytoscape session file transomics2cytoscape.cys. Note the connectivity/edge-list preparation is a manual, non-automated prerequisite: the package itself does not retrieve inter-layer edges from databases such as BRENDA, OmniPath, SIGNOR, or IntAct.

## Related tools

- **transomics2cytoscape** (R/Bioconductor package that adds edges between stacked pathway layers and integrates them into the 2.5D network in Cytoscape) — https://github.com/ecell/transomics2cytoscape
- **RCy3** (Cytoscape Automation interface used by transomics2cytoscape to import pathway maps and wire inter-layer edges in the Cytoscape network)
- **Cytoscape** (Hosts the resulting 2.5D trans-omic network in which inter-layer edges are rendered) — https://github.com/cytoscape/cytoscape

## Evaluation signals

- The saved session file transomics2cytoscape.cys exists and contains edges whose source and target nodes lie in different pathway layers
- Every molecule endpoint in the inter-layer edge list resolves to a node present in the stacked layers, i.e., IDs were converted to KEGG-compatible IDs (KEGG gene ID / EC numbers)
- For layer rows with the edge-midpoint flag set, cross-layer edges contain an inserted midpoint node
- Visual inspection (expert review) in Cytoscape/Cy3D shows inter-layer edges connecting the stacked layers without orphan endpoints
- Adding inter-layer edges does not disturb the conventional node/edge layout of each pathway layer

## Limitations

- Retrieval of inter-layer edges from databases such as BRENDA, OmniPath, SIGNOR, and IntAct is not automated; connectivity tables must be prepared manually
- The connectivity data of trans-omic networks are not fully automatically reconstructed
- Only KEGG ID and pathway file formats are supported for layers; Reactome, Wikipathways, and SBGN are not supported yet

## Evidence

- [discussion] adds edges between layers: "The transomics2cytoscape package adds edges between layers to understand molecular interactions between pathway layers"
- [discussion] KEGG-compatible ID conversion: "The IDs of molecules were converted into KEGG–compatible IDs such as KEGG gene ID and EC numbers by using Ensembl BioMart42"
- [discussion] inter-layer edge retrieval not automated: "transomics2cytoscape has not yet automated the retrieval processes for inter-layer edges."
- [readme] package creates 3D visualization: "Here we propose **transomics2cytoscape**, an R package that automatically creates 3D network visualization in combination with"
- [discussion] inter-layer edge sources: "Interactions between omic layers can also be obtained from databases such as BRENDA as well as pathway data from KEGG."
