---
name: rnaseq-gene-annotation
description: Use when patient RNA-seq data is keyed by Ensembl gene IDs and must be
  converted to gene symbols that match nodes of a fixed network model, with unmatched
  symbols resolved via GeneCards/HGNC aliases.
when_to_use_negative:
- The RNA-seq data is already keyed by official gene symbols that match the network
  node list — mapping would be redundant and alias resolution could introduce errors.
- The goal is transcript/gene quantification or differential expression analysis of
  RNA-seq reads; this skill only performs ID-to-symbol mapping and node filtering,
  not expression estimation or statistics.
- No fixed network node list is available to filter against, so the alias-resolution
  step has no target vocabulary to resolve symbols into.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CanSeer
  role: 'Provides the Step 2B pipeline: Ensembl-to-symbol mapping (BioTools.fr-style),
    filtering against the network node list, GeneCards/HGNC alias resolution, and
    writing gene_node_mapping.csv.'
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:5ea7574c4a2027e60399f5ffca8e1b1dc906890c89957e4390407fb1699fa3db
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# rnaseq-gene-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

CanSeer's Step 2B identifier-resolution routine: map patient RNA-seq Ensembl gene IDs to official gene symbols (BioTools.fr-style mapping), then filter and alias-resolve them against the literature-derived network node list so every retained gene maps onto a biomolecular regulatory network node.

## When to use

Use when patient RNA-seq data is keyed by Ensembl gene IDs and must be converted to gene symbols that match nodes of a fixed network model, with unmatched symbols resolved via GeneCards/HGNC aliases.

## When NOT to use

- The RNA-seq data is already keyed by official gene symbols that match the network node list — mapping would be redundant and alias resolution could introduce errors.
- The goal is transcript/gene quantification or differential expression analysis of RNA-seq reads; this skill only performs ID-to-symbol mapping and node filtering, not expression estimation or statistics.
- No fixed network node list is available to filter against, so the alias-resolution step has no target vocabulary to resolve symbols into.

## Inputs

- Patient RNA-seq gene list keyed by Ensembl gene IDs
- Literature-derived biomolecular regulatory network node symbol list (from the CanSeer package, github.com/BIRL/CanSeer)
- GeneCards / HGNC alias records for resolving non-matching gene symbols

## Outputs

- gene_node_mapping.csv (Ensembl ID, gene symbol, resolved network node)
- Filtered set of gene symbols restricted to network model nodes, used to annotate the personalized in silico model

## How to apply

Load the patient RNA-seq Ensembl gene ID list together with the literature-derived network node symbol list from the CanSeer package (github.com/BIRL/CanSeer). Map each Ensembl ID to its official gene symbol using the BioTools.fr-style identifier mapping service integrated in CanSeer. Filter the mapped symbols against the network node list, retaining only symbols corresponding to nodes in the biomolecular regulatory network model. For symbols with no direct node match, query GeneCards or HGNC alias records to resolve the gene to an accepted network node name. The decision point is direct-match vs. alias-resolution vs. drop: only genes resolvable to a network node enter the downstream annotation; the rest are discarded so the finalized gene set is fully node-compatible for downstream dynamical analysis.

## Related tools

- **CanSeer** (Provides the Step 2B pipeline: Ensembl-to-symbol mapping (BioTools.fr-style), filtering against the network node list, GeneCards/HGNC alias resolution, and writing gene_node_mapping.csv.) — github.com/BIRL/CanSeer

## Evaluation signals

- Every retained gene symbol in the output corresponds to a node in the biomolecular regulatory network model — no orphan symbols survive the filter.
- gene_node_mapping.csv is produced with the expected columns (Ensembl ID, gene symbol, resolved node) and a one-to-one Ensembl-ID-to-node resolution.
- Symbols that failed direct matching are traceable to a GeneCards or HGNC alias record rather than silently dropped.
- Downstream personalized model annotation succeeds, i.e., the finalized gene set integrates with patient-specific genomic/transcriptomic data for dynamical analysis without ID mismatches.

## Limitations

- Symbols that neither directly match a network node nor resolve to a node via GeneCards/HGNC aliases are dropped, so genes absent from the network vocabulary are lost from the patient profile.
- Correctness depends on the currency of external alias sources (BioTools.fr mapping service, GeneCards, HGNC); stale alias records can mis-resolve or fail to resolve gene symbols.
- The published model used for demonstration lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes — node coverage gaps propagate to incomplete gene-to-node mappings.

## Evidence

- [other] Step 2B mapping and alias resolution: [evidence span withheld — anchor status: not_found]
