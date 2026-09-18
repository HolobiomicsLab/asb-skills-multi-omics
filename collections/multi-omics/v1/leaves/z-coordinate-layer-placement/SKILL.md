---
name: z-coordinate-layer-placement
description: Use when building a multi-layer trans-omic network where pathway maps
  (e.g., KEGG pathways given as KEGG IDs or local pathway files) must be stacked at
  specified Z-heights to form an interpretable 2.5D visualization.
when_to_use_negative:
- Pathway layers come from databases other than KEGG (e.g., Reactome, Wikipathways)
  or formats beyond the pathway file format — the package currently supports only
  KEGG IDs and pathway files
- You need automated estimation of layer Z-coordinates — this is explicitly not automated
  in the package and must be supplied manually
- You only have a single 2D pathway/network with no multiple omic layers to stack
  — there is no Z-stacking to perform
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0602
tools:
- name: transomics2cytoscape
  role: R Bioconductor package whose create3Dnetwork function consumes the layer-definition
    table and stacks pathway layers at their Z-coordinates
  repo: https://github.com/ecell/transomics2cytoscape
- name: KEGGREST
  role: Retrieves KGML (KEGG pathway XML) for each layer's KEGG pathway ID
- name: RCy3
  role: Automates importing multiple pathway maps and integrating them into a 2.5D
    network in Cytoscape
- name: Cytoscape
  role: Hosts the stacked 2.5D network; layer Z-coordinates are applied and visualized
    within Cytoscape
  repo: https://github.com/cytoscape/cytoscape
- name: Cy3D
  role: Cytoscape app used to render and validate the 3D placement of layers at their
    Z-coordinates
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:6e582226a16f7622e014046b48eea4d812dcb3fb827428b787297b8ccef43ff9
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# z-coordinate-layer-placement

## Summary

Assigns each omic layer in a trans-omic network an explicit Z-coordinate via the four-column layer-definition table (layer index, KEGG pathway ID or pathway file, Z-height, edge-midpoint flag) so that transomics2cytoscape's create3Dnetwork can stack the pathway layers in 2.5D space within Cytoscape.

## When to use

Use when building a multi-layer trans-omic network where pathway maps (e.g., KEGG pathways given as KEGG IDs or local pathway files) must be stacked at specified Z-heights to form an interpretable 2.5D visualization.

## When NOT to use

- Pathway layers come from databases other than KEGG (e.g., Reactome, Wikipathways) or formats beyond the pathway file format — the package currently supports only KEGG IDs and pathway files
- You need automated estimation of layer Z-coordinates — this is explicitly not automated in the package and must be supplied manually
- You only have a single 2D pathway/network with no multiple omic layers to stack — there is no Z-stacking to perform

## Inputs

- Four-column layer-definition table (TSV/data.frame): layer index, KEGG pathway ID or Cytoscape-compatible pathway file, Z-coordinate, edge-midpoint flag
- KEGG pathway IDs referencing pathways retrievable as KGML (KEGG pathway XML) via KEGGREST
- Cytoscape-compatible pathway files (for layers not specified by KEGG ID)

## Outputs

- 2.5D network in Cytoscape with one pathway layer per table row, each placed at its specified Z-coordinate
- Inter-layer edges connecting molecules across stacked layers (with midpoint nodes where the flag is set)
- Cytoscape session file transomics2cytoscape.cys

## How to apply

First, prepare the layer-definition table as a four-column tabular file: (1) layer index, (2) KEGG pathway ID or a Cytoscape-compatible pathway file, (3) the Z-coordinate (height) at which that layer will sit in the 2.5D space, and (4) an edge-midpoint flag controlling whether an extra node is inserted at the midpoint of inter-layer edges. Feed this table to transomics2cytoscape's create3Dnetwork, which reads each row, retrieves KGML via KEGGREST for KEGG IDs or loads the supplied pathway file, imports the pathway maps into Cytoscape (via RCy3 and the KEGGscape app) preserving the conventional node/edge layout of each map, and applies the row's Z-height as the layer's coordinate in the 3D space rendered by the Cy3D app. Z-coordinates are NOT estimated automatically — the article states this estimation has not been automated, so heights must be chosen manually by the user (e.g., to separate transcriptome, proteome, and metabolome layers visibly). Finally, inter-layer edges connect molecules across the stacked layers, and the assembled network is saved as the Cytoscape session file transomics2cytoscape.cys.

## Related tools

- **transomics2cytoscape** (R Bioconductor package whose create3Dnetwork function consumes the layer-definition table and stacks pathway layers at their Z-coordinates) — https://github.com/ecell/transomics2cytoscape
- **KEGGREST** (Retrieves KGML (KEGG pathway XML) for each layer's KEGG pathway ID)
- **RCy3** (Automates importing multiple pathway maps and integrating them into a 2.5D network in Cytoscape)
- **Cytoscape** (Hosts the stacked 2.5D network; layer Z-coordinates are applied and visualized within Cytoscape) — https://github.com/cytoscape/cytoscape
- **Cy3D** (Cytoscape app used to render and validate the 3D placement of layers at their Z-coordinates)

## Examples

```
Rscript -e "library(transomics2cytoscape); create3Dnetwork('layerdefinition.tsv')"  # table rows: layer index, KEGG ID/pathway file, Z-height, edge-midpoint flag
```

## Evaluation signals

- The Cytoscape session file transomics2cytoscape.cys is produced and exists after the run
- The session contains one network per requested layer, each stacked at the Z-coordinate given in the corresponding row of the layer-definition table
- Each layer retains the conventional node/edge layout of its source pathway map (imported via KEGGscape)
- Inter-layer edges are present connecting molecules across the stacked layers, with midpoint nodes inserted only where the edge-midpoint flag is set
- The rendered 2.5D network in Cy3D shows layers at visually distinct heights with no unintended overlap of layer planes

## Limitations

- Z-coordinate estimation is not automated — the user must specify each layer's Z-height manually in the layer-definition table ('We have not yet automated the estimation of Z-coordinates in Case Study I and II.')
- Only KEGG IDs and pathway file formats are supported as layer sources; Reactome, Wikipathways, and SBGN are not supported
- In 2.5D visualization some nodes and edges can be hidden under the layer planes
- Retrieval of inter-layer edges from databases (BRENDA, OmniPath, SIGNOR, IntAct) is not automated, so the layers' connectivity data must be provided externally

## Evidence

- [intro] Prior knowledge used for stacking has two components: "The prior knowledge used to visualize interpretable networks has two components: (1) conventional pathway map layout and (2) stacked omic layers with a 2.5D perspective"
- [discussion] Z-coordinate estimation not automated: "We have not yet automated the estimation of Z-coordinates in Case Study I and II."
- [discussion] Integrating stacked layers in Cytoscape: "automate40 importing multiple pathway maps and integrating them into a 2.5D network in Cytoscape"
- [discussion] Only KEGG ID and pathway file format supported: "Currently, transomics2cytoscape supports only the KEGG ID and pathway file format."
- [readme] README: 3D stacking via R package with Cytoscape and Cy3D: "Here we propose **transomics2cytoscape**, an R package that automatically creates 3D network visualization in combination with Cytoscape, [Cy3D App]"
