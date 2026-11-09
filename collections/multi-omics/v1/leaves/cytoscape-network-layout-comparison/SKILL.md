---
name: cytoscape-network-layout-comparison
description: Use when you need to verify that an algorithmically generated network
  layout (e.g. degree-sorted circle or CoSE) retains the conventional pathway-map
  layout biologists rely on to locate metabolites and enzymes.
when_to_use_negative:
- The network is not pathway-map based and has no conventional layout to compare against
  (e.g. co-expression or PPI networks where no reference arrangement exists).
- The pathway data are not in KEGG KGML format — transomics2cytoscape currently supports
  only KEGG ID and pathway file format, so Reactome, Wikipathways, or SBGN inputs
  cannot follow this workflow.
- The goal is to produce a novel 2.5D trans-omic visualization rather than evaluate
  layout fidelity against a conventional layout.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3314
tools:
- name: transomics2cytoscape
  role: Builds the network from KGML while conserving the conventional KGML node/edge
    layout, providing the reference panel for the layout comparison
  repo: https://github.com/ecell/transomics2cytoscape
- name: Cytoscape
  role: Platform that renders the network and supplies the degree-sorted circle and
    CoSE layout algorithms and export functions
  repo: https://github.com/cytoscape/cytoscape
- name: RCy3
  role: R/Bioconductor interface used to script layout application and rendering export
    from Cytoscape
- name: KEGGREST
  role: Retrieves the KGML pathway files for glycolysis and the TCA cycle that seed
    the comparison
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:03a0d2a0f4cc1ebd1359df8ca225b456de254f8927e134938a75cfa3822a037e
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# cytoscape-network-layout-comparison

## Summary

Reproduce the same pathway network in Cytoscape under its conventional KGML-derived layout and under algorithmic layouts (degree-sorted circle, CoSE), exporting side-by-side renderings to test whether the automated layouts preserve the biologically recognizable structure of maps like the TCA cycle.

## When to use

Use when you need to verify that an algorithmically generated network layout (e.g. degree-sorted circle or CoSE) retains the conventional pathway-map layout biologists rely on to locate metabolites and enzymes.

## When NOT to use

- The network is not pathway-map based and has no conventional layout to compare against (e.g. co-expression or PPI networks where no reference arrangement exists).
- The pathway data are not in KEGG KGML format — transomics2cytoscape currently supports only KEGG ID and pathway file format, so Reactome, Wikipathways, or SBGN inputs cannot follow this workflow.
- The goal is to produce a novel 2.5D trans-omic visualization rather than evaluate layout fidelity against a conventional layout.

## Inputs

- KEGG pathway maps as KGML (KEGG pathway XML), e.g. glycolysis and TCA cycle (hsa00010, hsa00020)
- Tabular connectivity data describing the trans-omic network
- Molecule IDs convertible to KEGG-compatible IDs (KEGG gene IDs, EC numbers)

## Outputs

- Exported conventional-layout network rendering (reference panel)
- Exported degree-sorted-circle-layout rendering of the same network
- Exported CoSE-layout rendering of the same network
- Assembled layout-comparison figure (e.g. figure6_tca_layout_comparison.png)

## How to apply

Set up Cytoscape (3.8.0/3.10.3) with RCy3 and the KEGGscape app, installing openjdk-11-jdk, wget, and xvfb on Linux, and add the extracted cytoscape-unix-3.8.0 directory to $PATH. Retrieve the KEGG pathway maps (e.g. glycolysis, TCA cycle) as KGML via KEGGREST through transomics2cytoscape, converting molecule IDs into KEGG-compatible IDs (KEGG gene IDs and EC numbers via Ensembl BioMart); transomics2cytoscape conserves the conventional node/edge layout of the pathway layers, which becomes the reference panel. Apply Cytoscape's degree-sorted circle layout to the identical network via RCy3 and export it, then apply the CoSE layout and export it. Assemble the conventional, degree-sorted-circle, and CoSE renderings into a comparison figure (e.g. figure6_tca_layout_comparison.png) and judge whether the algorithmic layouts obscure the recognizable cyclic TCA structure.

## Related tools

- **transomics2cytoscape** (Builds the network from KGML while conserving the conventional KGML node/edge layout, providing the reference panel for the layout comparison) — https://github.com/ecell/transomics2cytoscape
- **Cytoscape** (Platform that renders the network and supplies the degree-sorted circle and CoSE layout algorithms and export functions) — https://github.com/cytoscape/cytoscape
- **RCy3** (R/Bioconductor interface used to script layout application and rendering export from Cytoscape)
- **KEGGREST** (Retrieves the KGML pathway files for glycolysis and the TCA cycle that seed the comparison)

## Examples

```
sudo apt-get install -y openjdk-11-jdk wget xvfb && wget https://github.com/cytoscape/cytoscape/releases/download/3.8.0/cytoscape-unix-3.8.0.tar.gz && tar xf cytoscape-unix-3.8.0.tar.gz
```

## Evaluation signals

- The conventional-layout panel reproduces the recognizable cyclic arrangement of the TCA cycle with metabolites and enzymes locatable relative to it.
- The degree-sorted-circle and CoSE panels of the same network are judged (expert review or LLM judge on the exported image) to obscure the TCA-cycle structure.
- All three renderings derive from the identical network (same nodes/edges, converted to KEGG gene IDs and EC numbers), so differences are attributable to layout only.
- Cytoscape environment is reproducible: cytoscape-unix-3.8.0 tarball extracted, in $PATH, with RCy3 and KEGGscape installed and reachable.

## Limitations

- transomics2cytoscape currently supports only the KEGG ID and pathway file format; Reactome, Wikipathways, and SBGN are not supported.
- The comparison finding is qualitative (expert review or LLM judge on the exported image), not a quantitative layout metric.
- Cytoscape core apps can fail to build from source for reasons unrelated to the JDK (maven-bundle-plugin/bndlib 4.1.0 bug, or unresolvable app dependencies), which can complicate environment setup.
- The transomics2cytoscape 3D/2.5D workflow requires a running Cytoscape instance with the Cy3D app via Cytoscape Automation.

## Evidence

- [discussion] conventional layout conservation: "The concept behind the workflow of transomics2cytoscape is to conserve the conventional layout of nodes and edges in the pathway layers."
- [discussion] KGML retrieval: "Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML)."
- [discussion] 2.5D visualization in conventional layouts: "The transomics2cytoscape package enables automatic visualization of 2.5D networks in conventional pathway layouts."
- [methods] Cytoscape install tarball: "wget https://github.com/cytoscape/cytoscape/releases/download/3.8.0/cytoscape-unix-3.8.0.tar.gz"
- [methods] Linux dependencies: "Run `sudo apt-get install -y openjdk-11-jdk wget xvfb` in the terminal."
- [methods] PATH configuration: "Add the decompressed cytoscape-unix-3.8.0 directory path to the $PATH environmental variable."
- [discussion] ID conversion to KEGG-compatible IDs: "The IDs of molecules were converted into KEGG–compatible IDs such as KEGG gene ID and EC numbers by using Ensembl BioMart42"
