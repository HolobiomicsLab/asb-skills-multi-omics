---
name: nes-table-export-and-interpretation
description: Use when representative programs (e.g. community-representative programs
  from an integrated mosaicMPI network) need functional validation against curated
  gene sets, and NES values must be tabulated per program-by-gene-set pair.
when_to_use_negative:
- When the goal is discovery of new programs rather than validation of already-defined
  representative programs — use cNMF factorization and integration first
- When gene sets are not available in GMT format and no equivalent curated set exists;
  the ssgsea subcommand requires '-g' gene sets to score against
- When input expression data contain missing (non-zero NaN) values — cNMF/ssGSEA input
  supports sparsity (zeros) but not missing values
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: mosaicmpi
  role: Provides the 'ssgsea' subcommand that computes ssGSEA Normalized Enrichment
    Scores of gene sets against programs
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Used to assemble the computed NES values into a tabular matrix indexed by
    program and gene set and export as TSV
- name: AnnData
  role: h5ad format holding the dataset expression matrix that ssGSEA is run on
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:24a25afc7ddb82f053fa34277e8f5f56fd0c8134e534e8c9c61529540ad0407c
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
  - AnnData
  - mosaicmpi
  - pandas
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.25
n_claims: 4
---

# NES-table-export-and-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run ssGSEA via the mosaicMPI 'ssgsea' subcommand against literature-derived gene sets (GMT format) to score representative programs, then export and interpret the resulting Normalized Enrichment Score (NES) matrix as a TSV table to validate community biological themes.

## When to use

Use when representative programs (e.g. community-representative programs from an integrated mosaicMPI network) need functional validation against curated gene sets, and NES values must be tabulated per program-by-gene-set pair.

## When NOT to use

- When the goal is discovery of new programs rather than validation of already-defined representative programs — use cNMF factorization and integration first
- When gene sets are not available in GMT format and no equivalent curated set exists; the ssgsea subcommand requires '-g' gene sets to score against
- When input expression data contain missing (non-zero NaN) values — cNMF/ssGSEA input supports sparsity (zeros) but not missing values

## Inputs

- Integrated network object (network_integration.pkl.gz) from upstream mosaicMPI integration
- Representative programs per dataset-community pair (program nearest to community median)
- Gene set file in GMT format (literature-derived, e.g. GBM gene sets) or a built-in set name such as GO_Biological_Process_2023
- Dataset expression matrix in h5ad format (AnnData)

## Outputs

- ssGSEA NES matrix indexed by program and gene set
- Exported NES table (ssgsea_nes_table.tsv)
- ssGSEA results directory (e.g. ssgsea_results) written by the ssgsea subcommand

## How to apply

Load the integrated network object (network_integration.pkl.gz) and extract the representative program per dataset and community, i.e. the program nearest to the median of the programs in that community. Assemble the literature-derived gene sets (e.g. GBM sets such as PAN/hypoxia, vasculature, T-cells) into a GMT-formatted file. Run 'mosaicmpi ssgsea' with the programs as input ('-n' for the dataset h5ad, or the representative programs) and '-g' pointing at the custom GMT file (or a default set like GO_Biological_Process_2023). Collect the resulting NES values into a pandas DataFrame indexed by program and gene set, and export as ssgsea_nes_table.tsv. Interpretation: positive/high NES for a gene set against a representative program confirms the theme assigned to its community via metadata label enrichment (e.g. neuronal and oligodendrocytic programs in C4 and C11, T-cell enrichment in C9, vasculature/mesenchymal tumor programs in C8).

## Related tools

- **mosaicmpi** (Provides the 'ssgsea' subcommand that computes ssGSEA Normalized Enrichment Scores of gene sets against programs) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Used to assemble the computed NES values into a tabular matrix indexed by program and gene set and export as TSV)
- **AnnData** (h5ad format holding the dataset expression matrix that ssGSEA is run on)

## Examples

```
mosaicmpi ssgsea -o ssgsea_results -n cptac_snrna.h5ad -g custom_gbm_gene_sets.gmt
```

## Evaluation signals

- The exported NES table contains an ssGSEA score for every representative program per dataset-community pair from the integration, with no missing enrichment values
- The table is indexed by program and gene set (rows/cols cover all GMT gene sets x all representative programs)
- NES-based characterizations match the metadata-based community label enrichment assignments (e.g. T-cell NES positive in C9, vasculature in C8, neuronal/oligodendrocytic in C4 and C11)
- The GMT file parses correctly and each gene set appears as a column/index entry in the NES matrix

## Limitations

- Mosaic integration may lose relevant information between modalities because not all features are shared, which can affect program-level enrichment signal
- ssGSEA against default gene sets (e.g. GO:BP) differs from custom literature-derived GMTs; choose '-g' appropriately for the biological question
- No software version information is stated in the article; verify the installed mosaicmpi version with 'mosaicmpi --help'

## Evidence

- [methods] To run ssGSEA on all cNMF programs for a dataset, using GO:BP terms (the default): mosaicmpi ssgsea -o ssgsea_results -n cptac_snrna.h5ad -g GO_Biological_Process_2023: [evidence span withheld — anchor status: not_found]
- [readme] Program communities are then interpreted using sample/cell metadata and gene set analyses.: [evidence span withheld — anchor status: external_doc]
