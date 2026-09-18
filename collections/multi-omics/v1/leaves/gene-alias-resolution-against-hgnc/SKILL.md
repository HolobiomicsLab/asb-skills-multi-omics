---
name: gene-alias-resolution-against-hgnc
description: Use when RNA-seq data are keyed by Ensembl gene IDs but a downstream
  biomolecular network model expects gene symbols, and direct symbol matches fail
  for some genes requiring GeneCards/HGNC alias lookup.
when_to_use_negative:
- The RNA-seq matrix is already keyed by official HGNC gene symbols that directly
  match the network node list (no ID mapping or aliasing is needed).
- The downstream model uses a non-gene-symbol node namespace (e.g. UniProt or protein
  identifiers), which HGNC alias records cannot resolve.
- The goal is de novo gene discovery or differential expression independent of a predefined
  network — mapping to a fixed node list would discard genes not present in the model.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0621
tools:
- name: CanSeer
  role: Integrated identifier-mapping (BioTools.fr-style) service that converts Ensembl
    IDs to gene symbols, filters them against network nodes, and drives the GeneCards/HGNC
    alias-resolution step; writes gene_node_mapping.csv
  repo: github.com/BIRL/CanSeer
- name: GeneCards/HGNC
  role: External alias records queried to resolve gene symbols that do not directly
    match an accepted network node name
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:88f8e5bf8097cdcc1ad72d65be277a867365db18ca348649079c9614f78e612f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BIRL/CanSeer
  repo_url: https://github.com/BIRL/CanSeer
  tools:
  - CanSeer
  - GeneCards/HGNC
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# gene-alias-resolution-against-hgnc

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

This skill reconciles patient RNA-seq Ensembl gene identifiers with the gene-symbol namespace of a literature-derived biomolecular regulatory network, so that every quantified gene can be attached to a network node (CanSeer Step 2B).

## When to use

Use when RNA-seq data are keyed by Ensembl gene IDs but a downstream biomolecular network model expects gene symbols, and direct symbol matches fail for some genes requiring GeneCards/HGNC alias lookup.

## When NOT to use

- The RNA-seq matrix is already keyed by official HGNC gene symbols that directly match the network node list (no ID mapping or aliasing is needed).
- The downstream model uses a non-gene-symbol node namespace (e.g. UniProt or protein identifiers), which HGNC alias records cannot resolve.
- The goal is de novo gene discovery or differential expression independent of a predefined network — mapping to a fixed node list would discard genes not present in the model.

## Inputs

- Patient RNA-seq gene list/matrix keyed by Ensembl gene IDs (CanSeer Step 2B input)
- Literature-derived biomolecular regulatory network node symbol list
- GeneCards/HGNC alias records (queried online or cached) for unresolved symbols

## Outputs

- gene_node_mapping.csv with one row per gene: Ensembl ID, official gene symbol, resolved network node name
- Filtered set of RNA-seq genes aligned to network nodes, usable for downstream model annotation

## How to apply

Load the patient RNA-seq Ensembl gene ID list and the literature-derived network node symbol list from the CanSeer package (github.com/BIRL/CanSeer). Map each Ensembl gene ID to its official gene symbol via the BioTools.fr-style identifier mapping service integrated in CanSeer. Filter the mapped gene symbols against the network node list, retaining only symbols that correspond to nodes in the biomolecular regulatory network model. For symbols that do not directly match a node, query GeneCards or HGNC alias records to resolve the symbol to an accepted network node name. Assemble the finalized mapping as gene_node_mapping.csv with Ensembl ID, gene symbol, and resolved node per row.

## Related tools

- **CanSeer** (Integrated identifier-mapping (BioTools.fr-style) service that converts Ensembl IDs to gene symbols, filters them against network nodes, and drives the GeneCards/HGNC alias-resolution step; writes gene_node_mapping.csv) — github.com/BIRL/CanSeer
- **GeneCards/HGNC** (External alias records queried to resolve gene symbols that do not directly match an accepted network node name)

## Evaluation signals

- Every Ensembl ID in the input list either maps to a network node in gene_node_mapping.csv or is explicitly recorded as unmatched — no silent drops.
- Every retained gene symbol in the final mapping corresponds to a node in the literature-derived network model.
- Rows whose symbol was not a direct match must trace back to a GeneCards or HGNC alias record (auditability of the alias source).
- Output file is written in the expected schema and filename (gene_node_mapping.csv with Ensembl ID, gene symbol, resolved node columns).
- Downstream CanSeer dynamical analysis accepts the mapping without namespace errors, since model annotation expects node-symbol keys.

## Limitations

- Symbols with no record in GeneCards/HGNC alias sources, or genes absent from the network node list, cannot be resolved and are excluded from the personalized model.
- The published network model used for demonstration lacked coverage for certain patient features, yielding inconclusive outcomes for some patients; coverage gaps in the node list propagate through this mapping step.
- The framework may need expansion for panomics data to avoid under- or over-fitting the patient profile; this skill handles transcriptomic (RNA-seq) identifiers only.

## Evidence

- [methods] Map each Ensembl gene ID to its official gene symbol using the BioTools.fr-style identifier mapping service integrated in CanSeer.: [evidence span withheld — anchor status: not_found]
- [methods] Filter the mapped gene symbols against the network node list, retaining only symbols that correspond to nodes in the biomolecular regulatory network model.: [evidence span withheld — anchor status: not_found]
- [methods] For genes whose symbols do not directly match a network node, query GeneCards/HGNC alias records to resolve the symbol to an accepted network node name.: [evidence span withheld — anchor status: not_found]
