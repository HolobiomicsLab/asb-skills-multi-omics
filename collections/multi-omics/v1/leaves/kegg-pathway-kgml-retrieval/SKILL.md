---
name: kegg-pathway-kgml-retrieval
description: Use when building a stacked 2.5D trans-omic network in Cytoscape whose
  layers are KEGG pathways (e.g. insulin signaling rno04910/mmu04910 over metabolic
  maps rno01100/mmu01100), requiring KGML retrieval and ID conversion to KEGG-compatible
  IDs.
when_to_use_negative:
- Your pathway data comes from Reactome, Wikipathways, or SBGN-ML — transomics2cytoscape
  currently supports only the KEGG ID and pathway file format.
- The inter-layer connectivity data must be fetched automatically from databases such
  as BRENDA, OmniPath, SIGNOR, or IntAct — that retrieval is not automated and must
  be supplied as tables.
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: KEGGREST
  role: R/Bioconductor client used by transomics2cytoscape to retrieve KGML for each
    KEGG pathway layer
- name: transomics2cytoscape
  role: Orchestrates KGML retrieval, layer stacking at Z-coordinates, and inter-layer
    edge insertion into Cytoscape
  repo: https://github.com/ecell/transomics2cytoscape
- name: RCy3
  role: R connection to Cytoscape used to automate importing the KGML-derived pathway
    maps into Cytoscape
- name: Cytoscape
  role: Desktop application (3.10.3 in the vignettes) hosting the stacked 2.5D network
    via Cytoscape Automation
  repo: https://github.com/cytoscape/cytoscape
merged_aliases:
- kegg-kgml-pathway-retrieval
merged_alias_records:
- alias: kegg-kgml-pathway-retrieval
  slug: kegg-kgml-pathway-retrieval
  jaccard_score: 1.0
  method: token-set-jaccard
  decision: auto
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_008
  - task_001
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:2611fbd9eb79afaef0c8fdb91e5f5c0192e42983703b0f3806482a2c54c62c38
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
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# kegg-pathway-kgml-retrieval

## Summary

Retrieves KEGG pathway maps as KGML (KEGG pathway XML) via KEGGREST so they can serve as conventionally-laid-out layers in a 2.5D trans-omic network built by transomics2cytoscape in Cytoscape.

## When to use

Use when building a stacked 2.5D trans-omic network in Cytoscape whose layers are KEGG pathways (e.g. insulin signaling rno04910/mmu04910 over metabolic maps rno01100/mmu01100), requiring KGML retrieval and ID conversion to KEGG-compatible IDs.

## When NOT to use

- Your pathway data comes from Reactome, Wikipathways, or SBGN-ML — transomics2cytoscape currently supports only the KEGG ID and pathway file format.
- The inter-layer connectivity data must be fetched automatically from databases such as BRENDA, OmniPath, SIGNOR, or IntAct — that retrieval is not automated and must be supplied as tables.

## Inputs

- KEGG pathway IDs for each layer (e.g. rno04910, rno01100, mmu04910, mmu01100)
- Four-column layer definition TSV (layer index, KEGG pathway ID or Cytoscape-compatible pathway file, Z-coordinate, edge-midpoint flag)
- Tabular trans-omic connectivity tables (Zenodo deposit 10.5281/zenodo.8201898)
- Molecule identifier lists to convert into KEGG-compatible IDs (KEGG gene IDs, EC numbers)

## Outputs

- KGML (KEGG pathway XML) documents retrieved via KEGGREST for each layer pathway
- KEGGscape-imported Cytoscape networks preserving the conventional pathway layout, stacked at specified Z-coordinates in a saved .cys session

## How to apply

Identify the KEGG pathway IDs for each omic layer (e.g. rno04910 insulin signaling, two rno01100 metabolic-map layers for Fao cells; mmu04910 plus three mmu01100 layers for WT/ob-ob liver), then list them in the four-column layer-definition TSV (layer index, KEGG pathway ID or pathway file, Z-coordinate, edge-midpoint flag). Use KEGGREST from R to retrieve the KGML for each KEGG ID, and import each KGML into Cytoscape 3.10.3 through RCy3 and the KEGGscape 0.9.2 app so the conventional node/edge layout of each pathway map is conserved. Convert molecule identifiers into KEGG-compatible IDs (KEGG gene IDs, EC numbers, e.g. via Ensembl BioMart) so inter-layer edges from the tabular connectivity data (Zenodo doi:10.5281/zenodo.8201898) can be attached across the stacked layers. Apply each layer's Z-coordinate for the 2.5D stacking visualized in Cy3D 1.1.3.

## Related tools

- **KEGGREST** (R/Bioconductor client used by transomics2cytoscape to retrieve KGML for each KEGG pathway layer)
- **transomics2cytoscape** (Orchestrates KGML retrieval, layer stacking at Z-coordinates, and inter-layer edge insertion into Cytoscape) — https://github.com/ecell/transomics2cytoscape
- **RCy3** (R connection to Cytoscape used to automate importing the KGML-derived pathway maps into Cytoscape)
- **Cytoscape** (Desktop application (3.10.3 in the vignettes) hosting the stacked 2.5D network via Cytoscape Automation) — https://github.com/cytoscape/cytoscape

## Examples

```
library(KEGGREST); kgml <- keggGet("rno04910", option = "kgml")
```

## Evaluation signals

- KGML is successfully retrieved via KEGGREST for every pathway ID listed in the layer-definition TSV (e.g. rno04910, rno01100, mmu04910, mmu01100).
- The saved Cytoscape session contains one KEGGscape network per requested layer, stacked at the specified Z-coordinates and checkable in Cy3D.
- The conventional node/edge layout of each pathway map is conserved (TCA cycle and glycolysis remain recognizable, unlike degree-sorted circle or CoSE layouts).
- Molecule nodes carry KEGG-compatible IDs (KEGG gene IDs, EC numbers) so inter-layer edges from the connectivity tables attach correctly.
- The assembled network matches the published figures (Fig. 4 three-layer Fao network; Fig. 5 five-layer WT/ob-ob liver network).

## Limitations

- Currently transomics2cytoscape supports only the KEGG ID and pathway file format; Reactome, Wikipathways, and SBGN are not supported.
- Retrieval of inter-layer edges from databases (BRENDA, OmniPath, SIGNOR, IntAct) and full reconstruction of connectivity data are not automated; tables must be provided.
- Z-coordinate estimation for Case Study I and II is not automated and must be chosen manually in the layer definition file.

## Evidence

- [discussion] KEGGREST KGML retrieval: "Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML)."
- [discussion] ID conversion to KEGG-compatible IDs: "The IDs of molecules were converted into KEGG–compatible IDs such as KEGG gene ID and EC numbers by using Ensembl BioMart42"
- [discussion] Format scope limitation: "Currently, transomics2cytoscape supports only the KEGG ID and pathway file format."
- [abstract] Package purpose: [evidence span withheld — anchor status: not_found]
