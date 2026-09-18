---
name: gmt-gene-set-file-preparation
description: Use when a set of literature-derived gene signatures must be formatted
  as a GMT file to run ssGSEA with mosaicMPI's ssgsea subcommand, e.g. to validate
  biological themes assigned to cNMF program communities in the CPTAC GBM landscape.
when_to_use_negative:
- When the default gene sets suffice — mosaicmpi ssgsea already defaults to GO_Biological_Process_2023
  terms, so a custom GMT is only needed for literature-derived signatures
- When the goal is program discovery or community detection itself (cNMF factorization
  / integration) rather than downstream enrichment-based annotation of representative
  programs
- When gene set members cannot be mapped to the dataset's feature identifiers — use
  mosaicMPI's Ensembl-based gene identifier conversion first
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: mosaicmpi
  role: Provides the 'ssgsea' subcommand that computes ssGSEA NES values for each
    gene set in the custom GMT file against each representative program
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Used to collect computed NES values into a tabular NES matrix indexed by program
    and gene set and export ssgsea_nes_table.tsv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:9d487fc730607bb88b9a69ed5dfbd86e54f78164216d17ff3032128bea9838bf
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/MorrissyLab/mosaicMPI
  repo_url: https://github.com/MorrissyLab/mosaicMPI
  tools:
  - mosaicmpi
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.1111111111111111
n_claims: 9
---

# GMT-gene-set-file-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepares a custom GMT-formatted gene set file from literature-derived signatures (e.g. GBM gene sets for PAN/hypoxia, vasculature, T-cells, neuronal, oligodendrocytic programs) so that mosaicMPI's 'ssgsea' subcommand can score representative programs via ssGSEA Normalized Enrichment Scores.

## When to use

Use when a set of literature-derived gene signatures must be formatted as a GMT file to run ssGSEA with mosaicMPI's ssgsea subcommand, e.g. to validate biological themes assigned to cNMF program communities in the CPTAC GBM landscape.

## When NOT to use

- When the default gene sets suffice — mosaicmpi ssgsea already defaults to GO_Biological_Process_2023 terms, so a custom GMT is only needed for literature-derived signatures
- When the goal is program discovery or community detection itself (cNMF factorization / integration) rather than downstream enrichment-based annotation of representative programs
- When gene set members cannot be mapped to the dataset's feature identifiers — use mosaicMPI's Ensembl-based gene identifier conversion first

## Inputs

- Literature-derived gene sets (e.g. GBM signature gene lists) as name-to-gene-list mappings
- Integrated network object network_integration.pkl.gz from the upstream mosaicMPI integration step
- Representative programs per dataset-community (program nearest to the median of programs in that community)

## Outputs

- GMT-formatted gene set file consumable by mosaicmpi ssgsea via the -g flag
- ssGSEA Normalized Enrichment Score (NES) matrix indexed by representative program and gene set (ssgsea_nes_table.tsv)

## How to apply

Collect the literature-derived gene sets relevant to the expected community themes (e.g. GBM signatures for PAN/hypoxia, vasculature, T-cells, neuronal, oligodendrocytic programs) and write them as a GMT-formatted file where each line contains a gene set name followed by its member genes. Load the integrated network object (network_integration.pkl.gz) and extract the representative program per dataset-community (the program nearest to the median of programs in that community). Run mosaicmpi 'ssgsea' with the representative programs as input and '-g' pointed at the custom GMT file instead of the default GO_Biological_Process_2023 term set. Collect the resulting NES values into a program-by-gene-set matrix (e.g. via pandas) and export as ssgsea_nes_table.tsv, then compare NES-based characterizations against the metadata-based community label enrichments (neuronal/oligodendrocytic in C4 and C11, T-cell in C9, vasculature/mesenchymal tumor in C8).

## Related tools

- **mosaicmpi** (Provides the 'ssgsea' subcommand that computes ssGSEA NES values for each gene set in the custom GMT file against each representative program) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Used to collect computed NES values into a tabular NES matrix indexed by program and gene set and export ssgsea_nes_table.tsv)

## Examples

```
mosaicmpi ssgsea -o ssgsea_results -n cptac_snrna.h5ad -g gbm_literature_gene_sets.gmt
```

## Evaluation signals

- The exported NES table contains an ssGSEA score for every representative program per dataset-community pair from the integration, with no missing enrichment values
- The GMT file parses with one gene set per line (name followed by member genes) and is accepted by 'mosaicmpi ssgsea' via the -g argument
- NES-based characterizations reproduce the metadata-based community themes: neuronal and oligodendrocytic programs confirmed in C4 and C11, T-cell enrichment in C9, vasculature/mesenchymal tumor programs in C8
- Each gene set in the GMT has genes that overlap the representative programs' features; use mosaicMPI's Ensembl-based identifier conversion if overlap is empty

## Limitations

- Mosaic integration can lose relevant information between modalities because not all features are shared, so some GMT gene set members may be absent from representative programs
- cNMF-derived inputs do not support missing values, so gene sets must be matched against complete feature vectors before scoring
- No software version information is stated for the tools in the article; check 'mosaicmpi --help' for the available subcommands in your installation

## Evidence

- [other] Assemble the literature-derived GBM gene sets into a GMT-formatted gene set file.: [evidence span withheld — anchor status: not_found]
- [methods] ssgsea -g flag usage: [evidence span withheld — anchor status: not_found]
- [readme] README on community interpretation via gene set analyses: [evidence span withheld — anchor status: external_doc]
