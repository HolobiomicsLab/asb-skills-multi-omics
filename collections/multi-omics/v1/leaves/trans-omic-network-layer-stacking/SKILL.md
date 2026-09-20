---
name: trans-omic-network-layer-stacking
description: Use when tabular trans-omic connectivity data must be rendered as a stacked
  multi-layer network (e.g., insulin signaling over metabolic maps) in Cytoscape's
  2.5D layout instead of manual VANTED-style assembly.
when_to_use_negative:
- Your pathway data is not in KEGG ID/KGML format — transomics2cytoscape currently
  supports only KEGG IDs and pathway files (Reactome, Wikipathways, SBGN are not supported).
- You need fully automated retrieval of inter-layer edges from databases (BRENDA,
  OmniPath, SIGNOR, IntAct) — retrieval and reconstruction of connectivity data are
  not automated.
- You need automatic Z-coordinate estimation — it must be supplied manually for the
  layered layout.
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0092
tools:
- name: transomics2cytoscape
  role: R Bioconductor package that automates importing pathway maps, stacking them
    as 2.5D layers, and adding inter-layer edges from tabular connectivity data
  repo: https://github.com/ecell/transomics2cytoscape
- name: RCy3
  role: R interface used to connect to and automate Cytoscape for importing maps and
    building the 2.5D network
- name: KEGGREST
  role: Retrieves KGML (KEGG pathway XML) for the pathway layers (e.g., mmu04910,
    mmu01100)
- name: Cytoscape
  role: Desktop application hosting the stacked 2.5D network via CyREST/Cy3D; run
    headless under xvfb on Linux
  repo: https://github.com/cytoscape/cytoscape
- name: Cy3D
  role: Cytoscape app that renders the stacked network in 3D/2.5D perspective
- name: KEGGscape
  role: Cytoscape app that imports KEGG pathway maps preserving their conventional
    node/edge layout
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:93ad0c6eafe1dcbd25d35c4cd5c8ae39d68621c6d2d091e78327bcd7125a72e5
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
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# trans-omic-network-layer-stacking

## Summary

Automated stacking of multiple pathway maps as 2.5D layers with inter-layer regulatory edges in Cytoscape, using the transomics2cytoscape R/Bioconductor package driven by RCy3, KEGGREST, KEGGscape, and Cy3D.

## When to use

Use when tabular trans-omic connectivity data must be rendered as a stacked multi-layer network (e.g., insulin signaling over metabolic maps) in Cytoscape's 2.5D layout instead of manual VANTED-style assembly.

## When NOT to use

- Your pathway data is not in KEGG ID/KGML format — transomics2cytoscape currently supports only KEGG IDs and pathway files (Reactome, Wikipathways, SBGN are not supported).
- You need fully automated retrieval of inter-layer edges from databases (BRENDA, OmniPath, SIGNOR, IntAct) — retrieval and reconstruction of connectivity data are not automated.
- You need automatic Z-coordinate estimation — it must be supplied manually for the layered layout.

## Inputs

- Tabular trans-omic connectivity tables (inter-layer edges: signaling-to-TF, TF-enzyme, enzyme-reaction, metabolite-reaction), e.g., usecase2 tables from Zenodo doi:10.5281/zenodo.8201898
- KEGG pathway IDs or KGML pathway files (e.g., mmu04910, mmu01100) retrieved via KEGGREST
- Layer definitions with Z-coordinates for each stacked pathway layer
- Molecule identifier mapping tables convertible to KEGG-compatible IDs (KEGG gene IDs, EC numbers)

## Outputs

- A 2.5D stacked trans-omic network in Cytoscape saved as a .cys session (e.g., usecase2_network.cys)
- 2.5D network visualization images (e.g., corresponding to Fig. 5 of the paper)
- Layer-count vs. wall-clock drawing-time measurements (e.g., drawing_time_table.csv) when benchmarking scalability

## How to apply

Launch Cytoscape with the Cy3D and KEGGscape apps installed and connect from R via RCy3 (e.g., Cytoscape 3.10.3, RCy3 2.24.0). Retrieve pathway KGML with KEGGREST (e.g., mmu04910 and mmu01100) and configure the layers with Z-coordinates so transomics2cytoscape stacks them as 2.5D layers. Convert molecule identifiers into KEGG-compatible IDs (KEGG gene IDs, EC numbers, e.g., via Ensembl BioMart) and supply inter-layer connectivity (signaling-to-TF, TF-enzyme, enzyme-reaction, metabolite-reaction) as tabular inputs from deposits such as Zenodo doi:10.5281/zenodo.8201898. Run the transomics2cytoscape workflow to import the maps, stack them along the Z axis, and add the inter-layer edges, then save the .cys session and export the 2.5D figure. On Linux, run Cytoscape headless under xvfb with OpenJDK 11 for reproducible, unattended rendering.

## Related tools

- **transomics2cytoscape** (R Bioconductor package that automates importing pathway maps, stacking them as 2.5D layers, and adding inter-layer edges from tabular connectivity data) — https://github.com/ecell/transomics2cytoscape
- **RCy3** (R interface used to connect to and automate Cytoscape for importing maps and building the 2.5D network)
- **KEGGREST** (Retrieves KGML (KEGG pathway XML) for the pathway layers (e.g., mmu04910, mmu01100))
- **Cytoscape** (Desktop application hosting the stacked 2.5D network via CyREST/Cy3D; run headless under xvfb on Linux) — https://github.com/cytoscape/cytoscape
- **Cy3D** (Cytoscape app that renders the stacked network in 3D/2.5D perspective)
- **KEGGscape** (Cytoscape app that imports KEGG pathway maps preserving their conventional node/edge layout)

## Examples

```
R: library(transomics2cytoscape); library(KEGGREST); kgml <- KEGGREST::keggGet("mmu04910", "kgml"); # then run transomics2cytoscape usecase2 workflow with RCy3 connected to Cytoscape 3.10.3
```

## Evaluation signals

- The saved .cys session contains the expected number of stacked pathway layers (e.g., five layers for usecase2: insulin signaling map, TF layer, and three metabolic map layers) with correct Z-coordinate ordering.
- Inter-layer edges of each expected type (signaling-to-TF, TF-enzyme, enzyme-reaction, metabolite-reaction) are present and their layout matches the corresponding paper figure (e.g., Fig. 5).
- Node and edge layout within each pathway layer is conserved relative to the conventional KEGG pathway map layout.
- For scalability benchmarking, drawing time increases approximately linearly with layer count (n = 2–6) and stays within the paper's reported few-minutes regime.
- drawing_time_table.csv exists and contains rows for all five stack counts (n = 2–6).

## Limitations

- Z-coordinate estimation in Case Study I and II is not automated; the user must supply layer Z-coordinates.
- Connectivity data of trans-omic networks are not fully automatically reconstructed; inter-layer edge retrieval from databases such as BRENDA, OmniPath, SIGNOR, and IntAct is not automated.
- Only KEGG ID and pathway file format are supported; Reactome, Wikipathways, and SBGN are not yet supported.
- Network summarization via enrichment analysis (as in Kokaji et al.) is not implemented yet.
- Some nodes and edges may be hidden under the layer planes in the 2.5D view.

## Evidence

- [discussion] KGML retrieval via KEGGREST: "Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML)."
- [discussion] 2.5D integration in Cytoscape: [evidence span withheld — anchor status: not_found]
- [discussion] ID conversion to KEGG-compatible IDs: "The IDs of molecules were converted into KEGG–compatible IDs such as KEGG gene ID and EC numbers by using Ensembl BioMart42"
- [discussion] inter-layer edge addition: "The transomics2cytoscape package adds edges between layers to understand molecular interactions between pathway layers"
- [methods] headless Linux setup: "Run `sudo apt-get install -y openjdk-11-jdk wget xvfb` in the terminal."
- [discussion] limited pathway format support: "Currently, transomics2cytoscape supports only the KEGG ID and pathway file format."
- [abstract] rapid rendering performance: "We confirmed that transomics2cytoscape could be used for rapid visualization of trans-omics networks presented in published papers within a few minutes"
- [discussion] layout conservation rationale: "The concept behind the workflow of transomics2cytoscape is to conserve the conventional layout of nodes and edges in the pathway layers."
