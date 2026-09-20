---
name: transomic-network-2-5d-visualization
description: Use when tabular trans-omic connectivity data (signaling-to-TF, TF-enzyme,
  enzyme-reaction, metabolite-reaction edges) plus KEGG pathway IDs must be turned
  into a stacked multi-layer 2.5D network figure that conserves conventional pathway
  layouts.
when_to_use_negative:
- Your pathway data is not in KEGG ID / KGML format — the tool 'supports only the
  KEGG ID and pathway file format' (Reactome, Wikipathways, SBGN are not supported
  yet)
- You need an automatically chosen Z-coordinate per layer or fully automatic retrieval
  of inter-layer edges from databases (BRENDA, OmniPath, SIGNOR, IntAct) — these steps
  are not automated
- You need an algorithmic network layout (degree-sorted circle, CoSE) rather than
  conventional pathway-map layouts — those layouts obscure recognizable structures
  like the TCA cycle
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: transomics2cytoscape
  role: R Bioconductor package that automates importing multiple KEGG pathway maps,
    stacking them as 2.5D layers, and adding inter-layer edges in Cytoscape
  repo: https://github.com/ecell/transomics2cytoscape
- name: Cytoscape
  role: Network desktop application into which layers are imported and the 2.5D network
    is rendered and saved
  repo: https://github.com/cytoscape/cytoscape
- name: RCy3
  role: R-to-Cytoscape automation bridge (v2.24.0) used to drive network construction
    from the vignette code
- name: KEGGREST
  role: Retrieves KGML (KEGG pathway XML) for the pathway layers
- name: Cy3D
  role: Cytoscape app (1.1.3) providing the 3D rendering of stacked layers
- name: KEGGscape
  role: Cytoscape app (0.9.2) that imports KGML pathway maps conserving their conventional
    layout
provenance:
  source_task_ids:
  - task_001
  - task_008
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:bda70d72c2d11d7caec0c22176b14eb58a89b41c3b35566419da38cf12f603eb
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# transomic-network-2.5d-visualization

## Summary

Automatically builds an interpretable 2.5D trans-omic network by stacking KEGG pathway maps as layers along a Z axis in Cytoscape's Cy3D renderer and connecting them with inter-layer regulatory edges, using the R Bioconductor package transomics2cytoscape driven through RCy3.

## When to use

Use when tabular trans-omic connectivity data (signaling-to-TF, TF-enzyme, enzyme-reaction, metabolite-reaction edges) plus KEGG pathway IDs must be turned into a stacked multi-layer 2.5D network figure that conserves conventional pathway layouts.

## When NOT to use

- Your pathway data is not in KEGG ID / KGML format — the tool 'supports only the KEGG ID and pathway file format' (Reactome, Wikipathways, SBGN are not supported yet)
- You need an automatically chosen Z-coordinate per layer or fully automatic retrieval of inter-layer edges from databases (BRENDA, OmniPath, SIGNOR, IntAct) — these steps are not automated
- You need an algorithmic network layout (degree-sorted circle, CoSE) rather than conventional pathway-map layouts — those layouts obscure recognizable structures like the TCA cycle

## Inputs

- Tabular trans-omic connectivity tables (inter-layer edges: signaling-to-TF, TF-enzyme, enzyme-reaction, metabolite-reaction), e.g. Zenodo deposit 10.5281/zenodo.8201898
- KEGG pathway identifiers to be stacked (e.g. rno04910, rno01100, mmu04910, mmu01100)
- Layer definition with Z-coordinates per pathway layer
- Molecule identifier tables convertible to KEGG-compatible IDs (KEGG gene IDs, EC numbers)

## Outputs

- A 2.5D Cytoscape network session (.cys, e.g. usecase1_2.5d_network.cys, usecase2_network.cys) with stacked pathway layers and inter-layer edges
- An exported 2.5D visualization figure (e.g. corresponding to Fig. 4 / Fig. 5 of the paper)
- Runtime record (e.g. runtime.txt) and layer-count vs drawing-time table (drawing_time_table.csv) for scalability checks

## How to apply

Set up the environment per the methods: install OpenJDK 11 (e.g. `sudo apt-get install -y openjdk-11-jdk wget xvfb` on Linux), install Cytoscape (3.8.0 tarball; validated against 3.10.3) with the KEGGscape 0.9.2 and Cy3D 1.1.3 apps, and connect from R 4.4.1 / Bioconductor 3.19 via RCy3 2.24.0. Use KEGGREST to retrieve KGML for the pathway IDs to stack (e.g. rno04910 insulin signaling over two rno01100 metabolic layers in Case Study I; mmu04910 with three mmu01100 layers in Case Study II). Convert molecule identifiers to KEGG-compatible IDs (KEGG gene IDs, EC numbers) and supply inter-layer edges from tabular connectivity data (e.g. the Zenodo deposit 10.5281/zenodo.8201898). Assign Z-coordinates to each layer — note that Z-coordinate estimation is manual, not automated — and call transomics2cytoscape to stack layers along the Z axis and draw edges between them. Rationale: the tool deliberately conserves the conventional KGML node/edge layout of each layer, which algorithmic layouts (degree-sorted circle, CoSE) destroy; expect runs to complete within a few minutes (Case Study I: 3 min on a Windows 11 i7-12700/128 GB PC) and drawing time scales roughly linearly for 2–6 stacked layers.

## Related tools

- **transomics2cytoscape** (R Bioconductor package that automates importing multiple KEGG pathway maps, stacking them as 2.5D layers, and adding inter-layer edges in Cytoscape) — https://github.com/ecell/transomics2cytoscape
- **Cytoscape** (Network desktop application into which layers are imported and the 2.5D network is rendered and saved) — https://github.com/cytoscape/cytoscape
- **RCy3** (R-to-Cytoscape automation bridge (v2.24.0) used to drive network construction from the vignette code)
- **KEGGREST** (Retrieves KGML (KEGG pathway XML) for the pathway layers)
- **Cy3D** (Cytoscape app (1.1.3) providing the 3D rendering of stacked layers)
- **KEGGscape** (Cytoscape app (0.9.2) that imports KGML pathway maps conserving their conventional layout)

## Examples

```
sudo apt-get install -y openjdk-11-jdk wget xvfb && wget https://github.com/cytoscape/cytoscape/releases/download/3.8.0/cytoscape-unix-3.8.0.tar.gz
```

## Evaluation signals

- The saved .cys session contains the expected number of stacked layers (e.g. 3 layers for Case Study I: rno04910 over two rno01100; 5 layers for Case Study II) matching the corresponding paper figure
- Inter-layer edges of each expected type (signaling-to-TF, TF-enzyme, enzyme-reaction, metabolite-reaction) are present between the stacked layers
- The conventional node/edge layout of each pathway layer is conserved — the TCA cycle or glycolysis remains recognizable in the conventional layout, unlike under degree-sorted circle or CoSE layouts
- The run completes within a few minutes (Case Study I benchmark: 3 minutes); drawing time for n = 2–6 stacked layers increases approximately linearly (drawing_time_table.csv)
- runtime.txt records a successful elapsed time and the exported image matches the reference figure on qualitative/expert review

## Limitations

- Only KEGG ID and pathway file format are supported; Reactome, Wikipathways, and SBGN are not yet supported
- Z-coordinate estimation in Case Study I and II is not automated
- Retrieval of inter-layer edges from databases such as BRENDA, OmniPath, SIGNOR, and IntAct is not automated; connectivity data are provided after reconstruction procedures but not fully automatically reconstructed
- 2.5D visualization hides some nodes and edges under the layer planes
- Network summarization via enrichment analysis (as in Kokaji et al.) is not implemented

## Evidence

- [abstract] we present an R Bioconductor package ‘transomics2cytoscape’ for automated visualization of 2.5D trans-omics networks: "we present an R Bioconductor package ‘transomics2cytoscape’ for automated visualization of 2.5D trans-omics networks"
- [discussion] Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML).: "Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML)."
- [discussion] The transomics2cytoscape package adds edges between layers to understand molecular interactions between pathway layers: "The transomics2cytoscape package adds edges between layers to understand molecular interactions between pathway layers"
- [methods] Run `sudo apt-get install -y openjdk-11-jdk wget xvfb` in the terminal.: "Run `sudo apt-get install -y openjdk-11-jdk wget xvfb` in the terminal."
- [readme] Visualization of trans-omic networks helps biological interpretation by illustrating pathways where the signals are transmitted.: "Visualization of trans-omic networks helps biological interpretation by illustrating pathways where the signals are transmitted."
- [discussion] Currently, transomics2cytoscape supports only the KEGG ID and pathway file format.: "Currently, transomics2cytoscape supports only the KEGG ID and pathway file format."
