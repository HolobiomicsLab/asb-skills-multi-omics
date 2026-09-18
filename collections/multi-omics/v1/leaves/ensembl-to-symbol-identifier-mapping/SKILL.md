---
name: ensembl-to-symbol-identifier-mapping
description: Use when a patient RNA-seq expression table is keyed by Ensembl gene
  IDs and must be reconciled with HGNC-style gene-symbol network node names before
  model annotation.
when_to_use_negative:
- The expression matrix is already keyed by official HGNC gene symbols and matches
  the network node list directly — mapping and alias resolution are unnecessary.
- The target model nodes are identified by Ensembl IDs themselves, so symbol conversion
  would introduce a gratuitous intermediate representation.
- You need whole-transcriptome annotation with no fixed reference node set — filtering
  against the CanSeer network list would discard most genes.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CanSeer
  role: 'Hosts the Step 2B pipeline: Ensembl-to-symbol mapping (BioTools.fr-style
    service), network-node filtering, and GeneCards/HGNC alias resolution'
  repo: github.com/BIRL/CanSeer
- name: GeneCards
  role: External alias database queried to resolve gene symbols that do not directly
    match a network node
- name: HGNC
  role: External nomenclature authority queried alongside GeneCards for gene alias
    resolution
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:8352e069fc32c342c2415c359de8fb6ca98c72282951556d58123fcc41831b43
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
  - GeneCards
  - HGNC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ensembl-to-symbol-identifier-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

CanSeer's Step 2B identifier-mapping procedure: converts patient RNA-seq Ensembl gene IDs into official gene symbols, filters them against a literature-derived biomolecular network node list, and resolves alias mismatches using GeneCards/HGNC records so expression data can annotate the personalized network model.

## When to use

Use when a patient RNA-seq expression table is keyed by Ensembl gene IDs and must be reconciled with HGNC-style gene-symbol network node names before model annotation.

## When NOT to use

- The expression matrix is already keyed by official HGNC gene symbols and matches the network node list directly — mapping and alias resolution are unnecessary.
- The target model nodes are identified by Ensembl IDs themselves, so symbol conversion would introduce a gratuitous intermediate representation.
- You need whole-transcriptome annotation with no fixed reference node set — filtering against the CanSeer network list would discard most genes.

## Inputs

- Patient RNA-seq gene expression table keyed by Ensembl gene IDs
- Literature-derived biomolecular regulatory network node symbol list (from the CanSeer package)

## Outputs

- gene_node_mapping.csv with columns (Ensembl ID, gene symbol, resolved node)
- Filtered gene-symbol set restricted to network nodes

## How to apply

Load two lists from the CanSeer package (github.com/BIRL/CanSeer): the patient RNA-seq Ensembl gene ID list and the literature-derived network node symbol list. Map each Ensembl gene ID to its official gene symbol using the BioTools.fr-style identifier mapping service integrated in CanSeer. Filter the mapped symbols against the network node list, retaining only symbols that correspond to nodes in the biomolecular regulatory network model; this filtering is required because the validated model is integrated with patient transcriptomic data for dynamical analysis. For genes whose symbols do not directly match a network node, query GeneCards/HGNC alias records to resolve the symbol to an accepted network node name rather than dropping the gene outright. Assemble the finalized Ensembl ID / gene symbol / resolved node triples and write them as gene_node_mapping.csv, which is the artifact used downstream to annotate the personalized model with patient-specific genetic alterations and RNA-seq data.

## Related tools

- **CanSeer** (Hosts the Step 2B pipeline: Ensembl-to-symbol mapping (BioTools.fr-style service), network-node filtering, and GeneCards/HGNC alias resolution) — github.com/BIRL/CanSeer
- **GeneCards** (External alias database queried to resolve gene symbols that do not directly match a network node)
- **HGNC** (External nomenclature authority queried alongside GeneCards for gene alias resolution)

## Evaluation signals

- gene_node_mapping.csv exists and every row has non-empty Ensembl ID, gene symbol, and resolved node fields.
- Every gene symbol in the mapping corresponds to a node in the literature-derived network model — no unmapped symbols leak through.
- Genes that failed direct symbol matching were checked against GeneCards/HGNC aliases before being dropped; only unresolvable aliases are absent.
- Spot-check a sample of Ensembl ID → symbol pairs against BioTools.fr or an independent reference to confirm correct mapping.
- Downstream model annotation succeeds, i.e. the resolved nodes match the network node identifiers CanSeer expects when integrating patient-specific genomic and transcriptomic data.

## Limitations

- The published model used for demonstration lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes — network node coverage limits how much mapped data can be used.
- Genes whose aliases cannot be resolved via GeneCards/HGNC are lost from the mapping, potentially underfitting the patient's profile (the article notes a broader need to cater for panomics data to avoid under- or overfitting).
- Mapping correctness depends on the external BioTools.fr-style service and alias databases being current; stale alias records would silently misresolve symbols.

## Evidence
