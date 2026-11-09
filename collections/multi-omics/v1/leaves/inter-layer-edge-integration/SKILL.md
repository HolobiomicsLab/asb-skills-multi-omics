---
name: inter-layer-edge-integration
description: Use when you have a layered KEGG-based pathway visualization (e.g., rno04910
  over rno01100 layers, or the five-layer mouse liver glucose-response network) and
  tabular trans-omic connectivity data that must be joined across layers by KEGG-compatible
  molecule IDs.
when_to_use_negative:
- Your pathway data is not in KEGG ID / KGML format — the package currently supports
  only the KEGG ID and pathway file format, not Reactome, Wikipathways, or SBGN
- You expect inter-layer edges to be retrieved automatically from databases such as
  BRENDA, OmniPath, SIGNOR, or IntAct — that retrieval is not automated and edges
  must be supplied as tabular input
- You need automated estimation of layer Z-coordinates — Z-coordinate estimation in
  Case Study I and II is manual
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0092
tools:
- name: transomics2cytoscape
  role: R Bioconductor package that stacks pathway layers by Z-coordinate and adds
    the inter-layer regulatory edges into the 2.5D network
  repo: https://github.com/ecell/transomics2cytoscape
- name: KEGGREST
  role: Retrieves KGML (KEGG pathway XML) for the pathway maps used as layers
- name: RCy3
  role: R interface used by transomics2cytoscape to automate importing pathway maps
    and integrating them into Cytoscape
- name: Cytoscape
  role: Network visualization platform in which the layered network with inter-layer
    edges is rendered; used with KEGGscape 0.9.2 and Cy3D 1.1.3 apps
  repo: https://github.com/cytoscape/cytoscape
- name: Cy3D
  role: Cytoscape app enabling the 3D (2.5D) rendering of stacked layers
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:fdee786660947d4b8763a75bc1bf56fed2adef923d3b9ea17bb5481f6cc7d638
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
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# inter-layer-edge-integration

## Summary

Adds regulatory edges that connect molecules across stacked pathway layers (signaling-to-TF, TF-enzyme, enzyme-reaction, metabolite-reaction) so a 2.5D trans-omic network in Cytoscape shows how signals are transmitted between omic layers, not just within each KEGG pathway map.

## When to use

Use when you have a layered KEGG-based pathway visualization (e.g., rno04910 over rno01100 layers, or the five-layer mouse liver glucose-response network) and tabular trans-omic connectivity data that must be joined across layers by KEGG-compatible molecule IDs.

## When NOT to use

- Your pathway data is not in KEGG ID / KGML format — the package currently supports only the KEGG ID and pathway file format, not Reactome, Wikipathways, or SBGN
- You expect inter-layer edges to be retrieved automatically from databases such as BRENDA, OmniPath, SIGNOR, or IntAct — that retrieval is not automated and edges must be supplied as tabular input
- You need automated estimation of layer Z-coordinates — Z-coordinate estimation in Case Study I and II is manual

## Inputs

- Tabular trans-omic connectivity tables (e.g., Yugi et al. Table S3/S4, Kokaji et al. Data file S11 from Zenodo doi:10.5281/zenodo.8201898)
- KGML pathway files retrieved via KEGGREST for KEGG pathway IDs (e.g., rno04910, rno01100, mmu04910, mmu01100)
- Molecule identifier tables requiring conversion to KEGG-compatible IDs (KEGG gene IDs, EC numbers)
- Layer configuration with Z-coordinates for stacking pathway maps

## Outputs

- A 2.5D trans-omic network in Cytoscape with stacked KEGG pathway layers and inter-layer regulatory edges (saved as a .cys session, e.g., usecase1_2.5d_network.cys or usecase2_network.cys)
- A 2.5D visualization export corresponding to Fig. 4 / Fig. 5 of the paper

## How to apply

First convert molecule identifiers in the connectivity tables into KEGG-compatible IDs (KEGG gene IDs, EC numbers) so nodes in different layers can be matched, e.g., via Ensembl BioMart as described in the paper. Retrieve the KGML for each KEGG pathway (e.g., rno04910, rno01100, mmu04910, mmu01100) with KEGGREST, and supply the inter-layer edge list from the deposited connectivity tables (Zenodo deposit 10.5281/zenodo.8201898, e.g., Yugi et al. Tables S3/S4 and Kokaji et al. Data file S11). Configure layer Z-coordinates so transomics2cytoscape stacks the pathway maps (e.g., two rno01100 metabolic layers under the rno04910 signaling layer, or five layers for the mmu use case) and let the package draw the inter-layer regulatory edges via RCy3 into Cytoscape with the Cy3D and KEGGscape apps. The rationale is to conserve the conventional node/edge layout of each pathway layer while connecting layers with regulatory edges, satisfying the four requirements of importing maps, stacking layers, drawing inter-layer edges, and automating the workflow. Save the result as a .cys session and validate against the expected 2.5D configuration (Fig. 4 or Fig. 5).

## Related tools

- **transomics2cytoscape** (R Bioconductor package that stacks pathway layers by Z-coordinate and adds the inter-layer regulatory edges into the 2.5D network) — https://github.com/ecell/transomics2cytoscape
- **KEGGREST** (Retrieves KGML (KEGG pathway XML) for the pathway maps used as layers)
- **RCy3** (R interface used by transomics2cytoscape to automate importing pathway maps and integrating them into Cytoscape)
- **Cytoscape** (Network visualization platform in which the layered network with inter-layer edges is rendered; used with KEGGscape 0.9.2 and Cy3D 1.1.3 apps) — https://github.com/cytoscape/cytoscape
- **Cy3D** (Cytoscape app enabling the 3D (2.5D) rendering of stacked layers)

## Examples

```
library(transomics2cytoscape); library(KEGGREST); kgml <- KEGGREST::keggGet("rno04910", "kgml"); transomics2cytoscape::create3Dnetwork(RCy3::cytoscapePing(), layerDefs, edgeTable)
```

## Evaluation signals

- The saved Cytoscape session reproduces the Fig. 4 three-layer configuration (rno04910 over two rno01100 layers) for Case Study I
- For Case Study II, the session contains five stacked layers (mmu04910 signaling map, TF layer, three mmu01100 metabolic layers) whose layout matches Fig. 5
- Inter-layer edges exist for all four interaction types: signaling-to-TF, TF-enzyme, enzyme-reaction, and metabolite-reaction
- Node IDs in the inter-layer edge list resolve to valid KEGG-compatible IDs (KEGG gene IDs, EC numbers)
- The complete run finishes within a few minutes (the paper reports ~3 minutes on a Windows 11 Desktop with an Intel Core i7-12700 and 128 GB RAM)

## Limitations

- Only the KEGG ID and pathway file format are supported; Reactome, Wikipathways, and SBGN are not supported yet
- Retrieval of inter-layer edges from databases such as BRENDA, OmniPath, SIGNOR, and IntAct is not automated; connectivity tables must be provided
- Z-coordinate estimation for layers is not automated in Case Study I and II
- Some nodes and edges are hidden under the layer planes in the 2.5D view

## Evidence

- [discussion] The transomics2cytoscape package adds edges between layers to understand molecular interactions between pathway layers: "The transomics2cytoscape package adds edges between layers to understand molecular interactions between pathway layers"
- [discussion] IDs converted to KEGG-compatible: "The IDs of molecules were converted into KEGG–compatible IDs such as KEGG gene ID and EC numbers by using Ensembl BioMart42"
- [discussion] Transomics2cytoscape uses KEGGREST: "Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML)."
- [readme] README: 3D visualization via Cytoscape and Cy3D: "an R package that automatically creates
3D network visualization in combination with
Cytoscape, [Cy3D App](http://apps.cytoscape.org/apps/cy3d), and
[Cytoscape Automation]"
- [discussion] Only KEGG format supported: "Currently, transomics2cytoscape supports only the KEGG ID and pathway file format."
