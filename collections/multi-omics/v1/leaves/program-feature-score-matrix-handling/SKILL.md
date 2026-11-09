---
name: program-feature-score-matrix-handling
description: Use when an integrated mosaicMPI network (e.g. network_integration.pkl.gz)
  exists and community-representative programs must be exported and scored against
  gene sets (GMT) to validate community labels such as T-cell or vasculature programs.
when_to_use_negative:
- When the input data has not been factorized and integrated (no network_integration.pkl.gz,
  no program communities) — representative programs cannot be defined without the
  median-correlation rule applied to an integrated graph
- When the goal is de novo pathway discovery across all cNMF programs of a single
  dataset rather than validating pre-assigned community themes on representative programs
- When the expression matrix contains missing values — cNMF (underlying program discovery)
  supports sparse data with zeros but not NaNs
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: mosaicmpi
  role: CLI 'ssgsea' subcommand computes ssGSEA NES for programs against a '-g' GMT
    gene set file; the network object supplies representative programs per community
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Collect ssGSEA NES values into a program-by-gene-set DataFrame and export
    ssgsea_nes_table.tsv; mosaicMPI objects can be built from pandas data matrices
- name: cNMF
  role: Produces the non-negative programs whose feature scores and community-representative
    selection feed the ssGSEA step
  repo: https://github.com/MorrissyLab/mosaicMPI
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:d9499c3d7811cce482e1bb8d99928a9ff92ee0168660d7bd7c2cd4c3729da0e0
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
  - cNMF
  - mosaicmpi
  - pandas
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# program-feature-score-matrix-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract community-representative cNMF programs (the program nearest the median of its mosaicMPI community) and use their program feature scores as input to ssGSEA with literature-derived gene sets, validating the biological themes assigned to program communities in the integrated network.

## When to use

Use when an integrated mosaicMPI network (e.g. network_integration.pkl.gz) exists and community-representative programs must be exported and scored against gene sets (GMT) to validate community labels such as T-cell or vasculature programs.

## When NOT to use

- When the input data has not been factorized and integrated (no network_integration.pkl.gz, no program communities) — representative programs cannot be defined without the median-correlation rule applied to an integrated graph
- When the goal is de novo pathway discovery across all cNMF programs of a single dataset rather than validating pre-assigned community themes on representative programs
- When the expression matrix contains missing values — cNMF (underlying program discovery) supports sparse data with zeros but not NaNs

## Inputs

- Integrated network object (network_integration.pkl.gz) from upstream mosaicMPI integration, containing programs, communities, and program feature scores
- Literature-derived gene sets formatted as a GMT file (whitespace-delimited, one gene set per line)
- Optional: integrated dataset AnnData/h5ad objects referenced by the ssgsea subcommand

## Outputs

- Representative program per dataset-community pair (nearest the median program of the community)
- ssGSEA Normalized Enrichment Score (NES) matrix indexed by program and gene set, exported as ssgsea_nes_table.tsv

## How to apply

Load the integrated network object (network_integration.pkl.gz) from the upstream mosaicMPI integration and, for each dataset-community pair, extract the representative program defined as the program nearest the median of programs in that community. Assemble the literature-derived gene sets (e.g. GBM signature sets for PAN/hypoxia, vasculature, T-cells) into a GMT-formatted whitespace-delimited gene set file. Run the mosaicMPI 'ssgsea' subcommand with the representative programs as input and '-g' pointed at the custom GMT file to compute ssGSEA Normalized Enrichment Scores, then collect the NES values with pandas into a tabular matrix indexed by program and gene set and export as ssgsea_nes_table.tsv. Rationale: community-representative programs identified via the feature-score median correlation rule are the canonical units for downstream analyses like pathway enrichment in mosaicMPI, and the NES directions should reproduce the metadata-based label enrichment assignments (e.g. neuronal/oligodendrocytic in C4 and C11, T-cell in C9, vasculature/mesenchymal tumor in C8 in CPTAC GBM).

## Related tools

- **mosaicmpi** (CLI 'ssgsea' subcommand computes ssGSEA NES for programs against a '-g' GMT gene set file; the network object supplies representative programs per community) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Collect ssGSEA NES values into a program-by-gene-set DataFrame and export ssgsea_nes_table.tsv; mosaicMPI objects can be built from pandas data matrices)
- **cNMF** (Produces the non-negative programs whose feature scores and community-representative selection feed the ssGSEA step) — https://github.com/MorrissyLab/mosaicMPI

## Examples

```
mosaicmpi ssgsea -o ssgsea_results -n cptac_snrna.h5ad -g GO_Biological_Process_2023
```

## Evaluation signals

- The exported ssgsea_nes_table.tsv contains an ssGSEA score for every representative program per dataset-community pair from the integration, with no missing enrichment values
- NES sign and magnitude reproduce the expected community-theme characterizations: neuronal and oligodendrocytic programs enriched in C4 and C11, T-cell enrichment in C9, vasculature/mesenchymal tumor programs in C8 in CPTAC GBM
- NES assignments match the metadata-based label enrichment assignments made upstream on the program network
- The GMT input file is well-formed (whitespace-delimited gene set lines) and each gene set yields NES values for all queried programs

## Limitations

- Mosaic integration can lose relevant information between modalities because not all features are shared across datasets, which propagates into program feature scores
- No software version information is stated for mosaicMPI/cNMF in the article; record versions via 'mosaicmpi --help' when reproducing
- Gene identifier mismatch between gene sets and programs may require Ensembl-based conversion, since mosaicMPI converts between gene names and Ensembl IDs within or between species

## Evidence

- [methods] To run ssGSEA on all cNMF programs for a dataset, using GO:BP terms (the default): mosaicmpi ssgsea -o ssgsea_results -n cptac_snrna.h5ad -g GO_Biological_Process_2023: [evidence span withheld — anchor status: not_found]
- [readme] Program communities are then interpreted using sample/cell metadata and gene set analyses.: [evidence span withheld — anchor status: external_doc]
