---
name: community-representative-program-selection
description: Use when an integrated mosaicMPI network (network_integration.pkl.gz)
  has program communities and you need a single program per dataset-community pair
  to summarize that community for downstream gene set or marker analyses.
when_to_use_negative:
- You need per-program (not per-community) analyses — mosaicMPI notes program feature
  scores support many downstream analyses directly, so summarizing to one representative
  per community would discard resolution
- The integrated network object has not yet been produced by the upstream mosaicMPI
  integration step (no communities to summarize)
- Community interpretation relies on metadata label enrichment alone and no feature-level
  summarization is required
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: mosaicMPI
  role: Provides the integrated program network and communities from which the representative
    program per dataset-community is extracted; downstream 'ssgsea' subcommand consumes
    the representative programs
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Used to assemble/tabulate feature scores and export downstream result tables
    (e.g. the ssGSEA NES matrix as ssgsea_nes_table.tsv)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:f5d517446c7e9d4640cd50c41825956c55158a90503612b3f33457f52fdc7555
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
  - mosaicMPI
  - pandas
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# community-representative-program-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects, for each community in the mosaicMPI integrated program network, the representative program nearest to the median of the community's programs (feature-score median correlation rule), so downstream analyses like ssGSEA can be run on one program per dataset-community instead of every program.

## When to use

Use when an integrated mosaicMPI network (network_integration.pkl.gz) has program communities and you need a single program per dataset-community pair to summarize that community for downstream gene set or marker analyses.

## When NOT to use

- You need per-program (not per-community) analyses — mosaicMPI notes program feature scores support many downstream analyses directly, so summarizing to one representative per community would discard resolution
- The integrated network object has not yet been produced by the upstream mosaicMPI integration step (no communities to summarize)
- Community interpretation relies on metadata label enrichment alone and no feature-level summarization is required

## Inputs

- Integrated mosaicMPI network object (network_integration.pkl.gz) containing programs, communities, and program feature scores
- Community assignments per program from the mosaicMPI integration

## Outputs

- One representative program per dataset-community pair (the program nearest to the median of community program feature scores)
- Representative-program set used as input to downstream analyses (e.g. mosaicMPI ssgsea, producing ssgsea_nes_table.tsv)

## How to apply

Load the integrated network object (network_integration.pkl.gz) produced by the upstream mosaicMPI integration step. For each dataset-community pair, extract the programs assigned to that community and compute the median of their feature scores; select as the community representative the program nearest to this median (the feature-score median correlation rule). The result is one representative program per dataset and community, which mosaicMPI uses for downstream analyses such as ssGSEA with custom GMT gene sets (e.g. literature-derived GBM gene sets for PAN/hypoxia, vasculature, T-cell themes). Validate that every dataset-community pair from the integration yields exactly one representative program; in the CPTAC GBM landscape this selection fed ssGSEA that confirmed neuronal/oligodendrocytic programs in C4/C11, T-cell enrichment in C9, and vasculature/mesenchymal tumor programs in C8.

## Related tools

- **mosaicMPI** (Provides the integrated program network and communities from which the representative program per dataset-community is extracted; downstream 'ssgsea' subcommand consumes the representative programs) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Used to assemble/tabulate feature scores and export downstream result tables (e.g. the ssGSEA NES matrix as ssgsea_nes_table.tsv))

## Examples

```
mosaicmpi ssgsea -o ssgsea_results -n network_integration.pkl.gz -g gbm_gene_sets.gmt
```

## Evaluation signals

- Every dataset-community pair present in network_integration.pkl.gz has exactly one selected representative program
- Each representative is the program nearest to the median of feature scores of programs in its community (median correlation rule satisfied)
- Downstream ssGSEA on the representative set yields no missing NES values: the exported ssgsea_nes_table.tsv has an enrichment score for every representative program × gene set combination
- NES-based characterizations of community themes match metadata-based label enrichment assignments (e.g. neuronal/oligodendrocytic in C4/C11, T-cells in C9, vasculature/mesenchymal in C8 in CPTAC GBM)

## Limitations

- Selection summarizes a community to a single program, so community-internal program heterogeneity is not captured in downstream analyses
- Representative programs inherit the feature overlap constraints of mosaic integration — potential loss of relevant information between modalities given that not all features are shared
- Interpretation of representatives depends on prior community assignment quality from the integration step

## Evidence

- [discussion] Identification of community-representative programs: "once community-representative programs are identified, program feature scores can be used for numerous downstream analyses"
- [methods] ssGSEA subcommand usage: [evidence span withheld — anchor status: not_found]
- [abstract] Integration into biological themes: "integration within and across datasets into a network representing biological themes"
- [discussion] Community-specific metadata annotation: "mosaicMPI can utilize community-specific metadata from every input modality, thereby enabling coordinated multi-dataset annotation of an integrated biological landscape"
