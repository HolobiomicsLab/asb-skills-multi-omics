---
name: conventional-pathway-layout-preservation
description: Use when visualizing KEGG pathway networks (KGML) where biological interpretability
  of the conventional map layout matters, e.g. when comparing how automated layout
  algorithms obscure cyclic structures like the TCA cycle.
when_to_use_negative:
- The pathway data is not in KEGG ID/KGML format — transomics2cytoscape currently
  supports only KEGG, not Reactome, Wikipathways, or SBGN
- The goal is a novel exploratory network topology rather than recognition of the
  conventional published pathway map arrangement
- Fully automated retrieval of inter-layer edges from BRENDA, OmniPath, SIGNOR, or
  IntAct is required — this step is not automated by the package
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0602
tools:
- name: transomics2cytoscape
  role: R Bioconductor package that builds the network while conserving the conventional
    KGML node/edge layout of pathway layers
  repo: https://github.com/ecell/transomics2cytoscape
- name: Cytoscape
  role: Network visualization platform into which the 2.5D network is imported; hosts
    the degree-sorted circle and CoSE layouts
  repo: https://github.com/cytoscape/cytoscape
- name: RCy3
  role: Cytoscape automation from R used to apply the degree-sorted circle and CoSE
    layouts and export the renderings
- name: KEGGREST
  role: Retrieves KGML (KEGG pathway XML) for the pathway maps
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:79f2abf577f2e66d7cddd9ba7859a78001ee4af16ab9003a5764fa155aa43602
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
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# conventional-pathway-layout-preservation

## Summary

Generates pathway network figures that conserve the conventional KGML node/edge layout biologists rely on to recognize structures like glycolysis and the TCA cycle, contrasting them against algorithmically generated layouts (degree-sorted circle, CoSE) that distort recognition.

## When to use

Use when visualizing KEGG pathway networks (KGML) where biological interpretability of the conventional map layout matters, e.g. when comparing how automated layout algorithms obscure cyclic structures like the TCA cycle.

## When NOT to use

- The pathway data is not in KEGG ID/KGML format — transomics2cytoscape currently supports only KEGG, not Reactome, Wikipathways, or SBGN
- The goal is a novel exploratory network topology rather than recognition of the conventional published pathway map arrangement
- Fully automated retrieval of inter-layer edges from BRENDA, OmniPath, SIGNOR, or IntAct is required — this step is not automated by the package

## Inputs

- KEGG KGML pathway XML files (e.g. glycolysis and TCA-cycle maps) retrieved via KEGGREST
- Tabular data describing trans-omic network connectivity (e.g. Yugi et al. Tables S3/S4, Kokaji et al. Data file S11)
- Molecule identifier tables to be converted into KEGG-compatible IDs (KEGG gene IDs, EC numbers) via Ensembl BioMart

## Outputs

- Exported PNG renderings of the pathway network in the conventional layout, degree-sorted circle layout, and CoSE layout
- Multi-panel layout comparison figure (e.g. figure6_tca_layout_comparison.png)

## How to apply

Install Cytoscape 3.10.3 with RCy3 2.24.0 and the KEGGscape 0.9.2 app (on Linux: 'sudo apt-get install -y openjdk-11-jdk wget xvfb', then wget and extract the cytoscape-unix-3.8.0 tarball and add it to $PATH). Retrieve the KEGG glycolysis and TCA-cycle pathway maps as KGML via KEGGREST through transomics2cytoscape, converting molecule IDs into KEGG-compatible IDs (KEGG gene IDs and EC numbers). Run transomics2cytoscape to build the network while conserving the conventional KGML node/edge layout, and export this rendering as the conventional-layout panel. Apply Cytoscape's degree-sorted circle layout, then the CoSE layout, to the same network via RCy3, exporting each rendering, and assemble the three panels into a comparison figure (e.g. figure6_tca_layout_comparison.png). The rationale is that transomics2cytoscape's workflow concept is to conserve the conventional layout of nodes and edges in pathway layers, whereas generic layout algorithms destroy recognizable structure.

## Related tools

- **transomics2cytoscape** (R Bioconductor package that builds the network while conserving the conventional KGML node/edge layout of pathway layers) — https://github.com/ecell/transomics2cytoscape
- **Cytoscape** (Network visualization platform into which the 2.5D network is imported; hosts the degree-sorted circle and CoSE layouts) — https://github.com/cytoscape/cytoscape
- **RCy3** (Cytoscape automation from R used to apply the degree-sorted circle and CoSE layouts and export the renderings)
- **KEGGREST** (Retrieves KGML (KEGG pathway XML) for the pathway maps)

## Examples

```
library(RCy3); layoutNetwork(network, layout.name="degree-sorted-circle")
```

## Evaluation signals

- The conventional-layout panel reproduces the recognizable cyclic arrangement of the TCA cycle, with metabolites and enzymes locatable relative to it
- The degree-sorted circle and CoSE panels visibly obscure the TCA cycle structure (qualitative check via expert review or LLM judge on the exported image)
- Node/edge positions in the conventional panel match the coordinates inherited from the source KGML rather than being recomputed by a layout algorithm
- The three renderings in the comparison figure derive from the same network, differing only in the applied layout
- The full workflow (KGML retrieval, network construction, layout application, export) completes within a few minutes using Cytoscape Automation

## Limitations

- Algorithmically generated layouts (degree-sorted circle and CoSE) distort the TCA cycle so that it is difficult to recognize
- Only the KEGG ID and pathway file format are currently supported; Reactome, Wikipathways, and SBGN are not
- Estimation of Z-coordinates in the 2.5D case studies and retrieval of inter-layer edges are not automated

## Evidence

- [discussion] conservation of conventional layout: "The concept behind the workflow of transomics2cytoscape is to conserve the conventional layout of nodes and edges in the pathway layers."
- [intro] reproducible conventional layout images: "The transomics2cytoscape package satisfies these four requirements, and reproducibly generates images of the trans-omic networks in the context of conventional pathway map layouts"
- [readme] automated 3D visualization via Cytoscape: "Here we propose **transomics2cytoscape**, an R package that automatically creates 3D network visualization in combination with Cytoscape"
- [methods] Cytoscape environment setup: "Run `wget https://github.com/cytoscape/cytoscape/releases/download/3.8.0/cytoscape-unix-3.8.0.tar.gz` in the terminal."
