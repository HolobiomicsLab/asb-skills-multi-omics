---
name: network-node-symbol-filtering
description: Use when patient RNA-seq data keyed by Ensembl IDs must be aligned to
  a fixed literature-derived biomolecular network node list before model annotation,
  with alias resolution for non-matching symbols.
when_to_use_negative:
- Input RNA-seq data is already keyed by official gene symbols that match network
  nodes, so no Ensembl-to-symbol mapping or alias resolution is needed
- The target model is not a literature-derived biomolecular regulatory network, so
  the network node list used as the filter reference does not exist
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: CanSeer
  role: 'Hosts the Step 2B pipeline: performs BioTools.fr-style Ensembl-to-gene-symbol
    mapping, filters symbols against the network node list, and resolves aliases via
    GeneCards/HGNC'
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:e42c037611f0818377a7abb6ef76f31bc3228a26329fcfb1b402e334922863fa
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BIRL/CanSeer
  tools:
  - CanSeer
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# network-node-symbol-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

CanSeer's Step 2B identifier-resolution routine: map patient RNA-seq Ensembl gene IDs to gene symbols via BioTools.fr, filter the mapped symbols against a literature-derived network node list, and resolve unmatched symbols through GeneCards/HGNC aliases to produce a finalized gene-to-node mapping.

## When to use

Use when patient RNA-seq data keyed by Ensembl IDs must be aligned to a fixed literature-derived biomolecular network node list before model annotation, with alias resolution for non-matching symbols.

## When NOT to use

- Input RNA-seq data is already keyed by official gene symbols that match network nodes, so no Ensembl-to-symbol mapping or alias resolution is needed
- The target model is not a literature-derived biomolecular regulatory network, so the network node list used as the filter reference does not exist

## Inputs

- Patient RNA-seq gene list keyed by Ensembl gene IDs
- Literature-derived biomolecular regulatory network node symbol list

## Outputs

- gene_node_mapping.csv containing Ensembl ID, gene symbol, and resolved network node name

## How to apply

Load the patient RNA-seq Ensembl gene ID list and the literature-derived biomolecular regulatory network node symbol list within the CanSeer package (github.com/BIRL/CanSeer). Map each Ensembl gene ID to its official gene symbol using the BioTools.fr-style identifier mapping service integrated in CanSeer. Filter the mapped gene symbols against the network node list, retaining only symbols that correspond to nodes in the biomolecular regulatory network model. For genes whose symbols do not directly match a network node, query GeneCards/HGNC alias records to resolve the symbol to an accepted network node name. Assemble the finalized gene-to-node mapping (Ensembl ID, gene symbol, resolved node) and write it as gene_node_mapping.csv for downstream patient-model annotation.

## Related tools

- **CanSeer** (Hosts the Step 2B pipeline: performs BioTools.fr-style Ensembl-to-gene-symbol mapping, filters symbols against the network node list, and resolves aliases via GeneCards/HGNC) — github.com/BIRL/CanSeer

## Evaluation signals

- Every retained gene symbol corresponds to a node present in the biomolecular regulatory network model (the filter is a strict intersection against the node list)
- gene_node_mapping.csv exists and carries the three expected columns: Ensembl ID, gene symbol, resolved node
- Each input Ensembl gene ID is either mapped to a network node or explicitly resolved via a GeneCards/HGNC alias; no unmapped Ensembl IDs remain unresolved in the output
- Resolved node names match accepted (HGNC) network node names rather than deprecated aliases

## Limitations

- The published model used for demonstration lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes, so symbols mapping to uncovered nodes may yield uninformative annotations
- Alias resolution depends on the completeness of GeneCards/HGNC alias records; symbols absent from those sources cannot be resolved to network nodes
- The framework requires expansion to cater for panomics data to avoid under- or overfitting of a patient's profile, so Ensembl-ID/RNA-seq-only filtering may under-represent other omics modalities

## Evidence

- [other] CanSeer's Step 2B maps RNA-seq data identified by Ensembl IDs to gene symbols using BioTools.fr, filters the resulting gene symbols against the network node list, and resolves unmatched gene names by sourcing aliases from GeneCards or HGNC.: [evidence span withheld — anchor status: not_found]
- [other] Workflow step 2 and 4: [evidence span withheld — anchor status: not_found]
- [other] Alias resolution step: [evidence span withheld — anchor status: not_found]
