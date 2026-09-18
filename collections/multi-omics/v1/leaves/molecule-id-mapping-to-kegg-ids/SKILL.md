---
name: molecule-id-mapping-to-kegg-ids
description: Use when input connectivity tables or pathway layers contain non-KEGG
  molecule identifiers (e.g., Ensembl, gene symbols) that must be harmonized to KEGG
  gene IDs and EC numbers before building a multi-layered trans-omic network.
when_to_use_negative:
- The pathway database is not KEGG — transomics2cytoscape supports only the KEGG ID
  and pathway file format, so Reactome, Wikipathways, or SBGN identifiers cannot be
  mapped this way.
- Input tables already use KEGG gene IDs and EC numbers as their molecule identifiers,
  in which case remapping is unnecessary.
- Inter-layer connectivity data must be retrieved automatically from databases such
  as BRENDA, OmniPath, SIGNOR, or IntAct — this retrieval is not automated and the
  mapping step assumes pre-reconstructed connectivity tables.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: transomics2cytoscape
  role: Consumes the KEGG-compatible ID tables to build the 2.5D stacked network;
    only accepts KEGG IDs and pathway file format
  repo: https://github.com/ecell/transomics2cytoscape
- name: KEGGREST
  role: Retrieves KGML (KEGG pathway XML) so that KEGG pathway nodes are already in
    KEGG-compatible ID space
- name: RCy3
  role: Automates importing the mapped, layered network into Cytoscape and rendering
    it as a 2.5D network
merged_aliases:
- molecule-id-kegg-compatible-mapping
merged_alias_records:
- alias: molecule-id-kegg-compatible-mapping
  slug: molecule-id-kegg-compatible-mapping
  jaccard_score: 0.6667
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:faa0ab01a087ba404314625bbba0d9adefdc8a49a27a285b20653cd46b5b0077
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ecell/transomics2cytoscape
  tools:
  - KEGGREST
  - R
  - RCy3
  - transomics2cytoscape
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# Molecule ID mapping to KEGG-compatible IDs

## Summary

Converts molecule identifiers from heterogeneous omic tables and pathway sources into KEGG-compatible IDs (KEGG gene IDs and EC numbers) so that nodes from different omic layers can be joined into a trans-omic network and visualized by transomics2cytoscape in Cytoscape.

## When to use

Use when input connectivity tables or pathway layers contain non-KEGG molecule identifiers (e.g., Ensembl, gene symbols) that must be harmonized to KEGG gene IDs and EC numbers before building a multi-layered trans-omic network.

## When NOT to use

- The pathway database is not KEGG — transomics2cytoscape supports only the KEGG ID and pathway file format, so Reactome, Wikipathways, or SBGN identifiers cannot be mapped this way.
- Input tables already use KEGG gene IDs and EC numbers as their molecule identifiers, in which case remapping is unnecessary.
- Inter-layer connectivity data must be retrieved automatically from databases such as BRENDA, OmniPath, SIGNOR, or IntAct — this retrieval is not automated and the mapping step assumes pre-reconstructed connectivity tables.

## Inputs

- Tabular trans-omic connectivity tables (e.g., Zenodo usecase2 deposit 10.5281/zenodo.8201898) with molecule identifiers in native formats (Ensembl, gene symbols, EC numbers)
- KGML pathway files retrieved via KEGGREST (e.g., mmu04910, mmu01100)
- Molecule identifier lists from omic layers (signaling proteins, transcription factors, enzymes, metabolites)

## Outputs

- Node tables with KEGG gene IDs and/or EC numbers as primary keys
- Inter-layer edge tables keyed by KEGG-compatible IDs, ready for transomics2cytoscape import

## How to apply

Before feeding connectivity tables into transomics2cytoscape, map each molecule identifier to a KEGG-compatible identifier: genes to KEGG gene IDs and enzymes to EC numbers, using a mapping service such as Ensembl BioMart as described in the article. This step is mandatory because transomics2cytoscape currently supports only KEGG IDs and the KEGG pathway file format (KGML), so unmapped IDs cannot be placed onto KEGG pathway layers. Apply the mapping to both the intra-layer pathway nodes (retrieved as KGML via KEGGREST) and the inter-layer edge tables (e.g., signaling-to-TF, TF-enzyme, enzyme-reaction, metabolite-reaction edges from the deposited Zenodo usecase2 tables, doi:10.5281/zenodo.8201898), so that inter-layer edges can be attached to the correct nodes in the stacked 2.5D layers. Verify that every node participating in an inter-layer edge resolves to a valid KEGG ID; otherwise the edge will silently fail to attach in the Cytoscape network.

## Related tools

- **transomics2cytoscape** (Consumes the KEGG-compatible ID tables to build the 2.5D stacked network; only accepts KEGG IDs and pathway file format) — https://github.com/ecell/transomics2cytoscape
- **KEGGREST** (Retrieves KGML (KEGG pathway XML) so that KEGG pathway nodes are already in KEGG-compatible ID space)
- **RCy3** (Automates importing the mapped, layered network into Cytoscape and rendering it as a 2.5D network)

## Examples

```
BiocManager::install("transomics2cytoscape"); library(transomics2cytoscape); library(KEGGREST); kgml <- KEGGREST::keggGet("mmu04910", "kgml")
```

## Evaluation signals

- Every node in the inter-layer edge tables resolves to a valid KEGG gene ID or EC number; no NA/unmapped identifiers remain.
- In the saved Cytoscape session (e.g., usecase2_network.cys), inter-layer edges for signaling-to-TF, TF-enzyme, enzyme-reaction, and metabolite-reaction interactions are attached between the correct stacked layers.
- The five-layered network reproduces Fig. 5 of the paper when built from the usecase2 data, with edges landing on nodes in the correct KEGG pathway layers (mmu04910, mmu01100).
- Nodes retain their conventional KGML positions within each pathway layer after ID mapping, confirming the layout was conserved.

## Limitations

- transomics2cytoscape supports only the KEGG ID and pathway file format; other pathway databases (Reactome, Wikipathways) are not yet supported.
- Retrieval of inter-layer edges from databases (BRENDA, OmniPath, SIGNOR, IntAct) and the reconstruction of connectivity data are not fully automated, so ID mapping must be applied to manually prepared tables.
- Estimation of Z-coordinates for layer stacking in Case Study I and II is not automated and remains a manual configuration step.

## Evidence

- [discussion] ID conversion via Ensembl BioMart: "The IDs of molecules were converted into KEGG–compatible IDs such as KEGG gene ID and EC numbers by using Ensembl BioMart42"
- [discussion] KEGG-only format support: "Currently, transomics2cytoscape supports only the KEGG ID and pathway file format."
- [discussion] KGML retrieval via KEGGREST: "Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML)."
- [readme] Network visualization purpose: "Visualization of trans-omic networks helps biological interpretation by illustrating pathways where the signals are transmitted."
- [intro] Automated trans-omic visualization from tabular data: "Here, we present an R package, transomics2cytoscape, that automates the visualization of trans-omic networks from tabular data describing network connectivity."
