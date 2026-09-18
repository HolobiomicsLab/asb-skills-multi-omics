---
name: ssgsea-gene-set-enrichment-scoring
description: Use when program feature scores or program loadings need functional interpretation
  against literature-derived or ontology gene sets (e.g. GBM PAN/hypoxia, vasculature,
  T-cell signatures) to confirm community-level biological themes.
when_to_use_negative:
- When gene sets are not defined in advance — use community metadata propagation or
  marker-gene analysis instead of ssGSEA to discover themes
- When the goal is dataset integration or program discovery itself (cNMF factorization
  / network integration), not scoring existing programs against known signatures
- When the expression matrix contains missing (non-zero-sparse NaN) values, which
  cNMF/mosaicMPI inputs do not support
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3173
tools:
- name: mosaicmpi
  role: Provides the 'ssgsea' subcommand that computes ssGSEA Normalized Enrichment
    Scores for gene sets against cNMF programs
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Used to assemble and export the NES matrix as a tabular file (ssgsea_nes_table.tsv)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:30c93dcfd9fa985ecfe5931e81f403a01239187e4e633bee74ae38fa8c94941c
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
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.2
n_claims: 5
---

# ssGSEA-gene-set-enrichment-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores gene set enrichment of molecular programs (e.g. community-representative cNMF programs) using mosaicMPI's 'ssgsea' subcommand, producing a matrix of ssGSEA Normalized Enrichment Scores (NES) per program per gene set to validate biological themes assigned to program communities.

## When to use

Use when program feature scores or program loadings need functional interpretation against literature-derived or ontology gene sets (e.g. GBM PAN/hypoxia, vasculature, T-cell signatures) to confirm community-level biological themes.

## When NOT to use

- When gene sets are not defined in advance — use community metadata propagation or marker-gene analysis instead of ssGSEA to discover themes
- When the goal is dataset integration or program discovery itself (cNMF factorization / network integration), not scoring existing programs against known signatures
- When the expression matrix contains missing (non-zero-sparse NaN) values, which cNMF/mosaicMPI inputs do not support

## Inputs

- Integrated network object from upstream mosaicMPI integration (e.g. network_integration.pkl.gz) containing programs and community assignments
- Representative programs per dataset-community (program nearest to the median of the community's programs)
- GMT-formatted gene set file of literature-derived gene sets (or a built-in named collection such as GO_Biological_Process_2023)
- Integrated dataset AnnData file (e.g. cptac_snrna.h5ad) when running ssGSEA over all cNMF programs

## Outputs

- ssGSEA NES table (e.g. ssgsea_nes_table.tsv): NES matrix indexed by representative program and gene set
- ssGSEA results directory (e.g. ssgsea_results) with per-run enrichment outputs

## How to apply

Extract the representative program per dataset-community from the integrated network object (e.g. network_integration.pkl.gz) — the program nearest to the median of programs in that community, selected via the feature-score median correlation rule. Assemble the literature-derived gene sets into a GMT-formatted file (or supply a named set such as GO_Biological_Process_2023). Run the mosaicMPI 'ssgsea' subcommand with the representative programs as input and '-g' pointed at the custom GMT, collecting the resulting ssGSEA Normalized Enrichment Scores into a program-by-gene-set matrix (e.g. ssgsea_nes_table.tsv). The NES pattern is then compared against metadata-based community label enrichment: e.g. neuronal/oligodendrocytic programs in C4 and C11, T-cell enrichment in C9, and vasculature/mesenchymal tumor programs in C8 in the CPTAC GBM landscape.

## Related tools

- **mosaicmpi** (Provides the 'ssgsea' subcommand that computes ssGSEA Normalized Enrichment Scores for gene sets against cNMF programs) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Used to assemble and export the NES matrix as a tabular file (ssgsea_nes_table.tsv))

## Examples

```
mosaicmpi ssgsea -o ssgsea_results -n cptac_snrna.h5ad -g my_gbm_gene_sets.gmt
```

## Evaluation signals

- The exported NES table contains an ssGSEA score for every representative program per dataset-community pair from the integration, with no missing enrichment values
- NES-based characterizations reproduce the article's community themes: neuronal/oligodendrocytic enrichment in C4 and C11, T-cell enrichment in C9, vasculature/mesenchymal tumor programs in C8
- The GMT file parses without errors and each gene set is evaluated against every input program
- The NES table is a program × gene set matrix matching the number of representative programs extracted from network_integration.pkl.gz

## Limitations

- Some analyses require extra dependencies (e.g. GSEApy) installed via other channels beyond the base conda-forge mosaicmpi environment
- ssGSEA scores existing gene sets — it validates pre-assigned themes rather than discovering them de novo
- Mosaic integration can lose relevant information between modalities since not all features are shared, which can affect program gene sets passed to ssGSEA

## Evidence

- [results] Run the mosaicMPI 'ssgsea' subcommand with the representative programs as input and '-g' pointed at the custom GMT file: [evidence span withheld — anchor status: not_found]
- [methods] To run ssGSEA on all cNMF programs for a dataset, using GO:BP terms (the default):: [evidence span withheld — anchor status: not_found]
- [readme] Program communities are then interpreted using sample/cell metadata and gene set analyses.: [evidence span withheld — anchor status: external_doc]
