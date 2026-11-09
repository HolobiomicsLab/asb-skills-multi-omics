---
name: integrated-program-community-interpretation
description: Use when a community-by-sample usage matrix from mosaicMPI integration
  exists and you need to associate communities with categorical metadata (e.g. EGFR/NF1/PDGFRA/BRAF
  alterations, subtypes, cell types) or numerical metadata (e.g. tumor purity).
when_to_use_negative:
- Input is raw expression data before mosaicMPI program discovery and integration
  — communities do not yet exist, so there is nothing to associate metadata with
- You need gene-level or program-level (not community-level) interpretation, e.g.
  ssGSEA on individual cNMF programs or marker gene identification — use the dedicated
  downstream analyses instead
- cNMF input data contains missing values rather than zeros — cNMF 'supports input
  data that is sparse (i.e. with zeros), but not with missing values'
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: mosaicMPI
  role: Framework whose integrated program communities and annotation propagation
    provide the usage matrix and community structure that this skill interprets
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Loads the community-by-sample usage matrix and sample metadata table for computing
    over-representation statistics and Pearson correlations
- name: NumPy
  role: Numerical computation backend for Pearson correlation of numerical metadata
    with community usage
- name: AnnData
  role: File/object format from which data matrices can be imported ('These can be
    made from from pandas data matrices or imported from AnnData files/objects')
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:7ff9ada2d9d1193a3c9c66382de839510b486488b5b75cfee4241b403c5ecd6f
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
  - NumPy
  - mosaicMPI
  - pandas
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# integrated-program-community-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistically characterizes mosaicMPI program communities by testing over-representation of categorical sample metadata (driver alterations, transcriptional subtypes, cell types) and Pearson-correlating numerical metadata (e.g. tumor purity) with community usage, producing a community-by-metadata enrichment table that links genotypes/phenotypes to specific communities.

## When to use

Use when a community-by-sample usage matrix from mosaicMPI integration exists and you need to associate communities with categorical metadata (e.g. EGFR/NF1/PDGFRA/BRAF alterations, subtypes, cell types) or numerical metadata (e.g. tumor purity).

## When NOT to use

- Input is raw expression data before mosaicMPI program discovery and integration — communities do not yet exist, so there is nothing to associate metadata with
- You need gene-level or program-level (not community-level) interpretation, e.g. ssGSEA on individual cNMF programs or marker gene identification — use the dedicated downstream analyses instead
- cNMF input data contains missing values rather than zeros — cNMF 'supports input data that is sparse (i.e. with zeros), but not with missing values'

## Inputs

- community-by-sample usage matrix from the mosaicMPI integration step
- sample metadata table (categorical layers: transcriptional subtypes, cell-type labels, driver alterations EGFR/NF1/PDGFRA/BRAF; numerical layers: tumor purity)

## Outputs

- community-by-metadata enrichment table (enrichment_scores.tsv) containing over-representation statistics for categorical layers and Pearson correlations for numerical layers

## How to apply

Load the community-by-sample usage matrix produced by the mosaicMPI integration step together with the sample metadata table (categorical layers such as transcriptional subtypes, cell-type labels, and driver alterations EGFR/NF1/PDGFRA/BRAF; numerical layers such as tumor purity) using pandas. For each categorical layer, compute an over-representation statistic for each category within each community, comparing observed versus expected category membership across samples assigned to that community. For each numerical layer, compute the Pearson correlation between metadata values and community usage across samples, ignoring missing numerical values. Assemble over-representation and correlation scores into a single community-by-metadata enrichment table written as enrichment_scores.tsv. This follows the paper's rationale that 'Clinical and molecular annotations from cohorts are statistically propagated onto this network of programs, yielding a richly characterized landscape of biological themes', enabling coordinated multi-dataset annotation using community-specific metadata from every input modality.

## Related tools

- **mosaicMPI** (Framework whose integrated program communities and annotation propagation provide the usage matrix and community structure that this skill interprets) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Loads the community-by-sample usage matrix and sample metadata table for computing over-representation statistics and Pearson correlations)
- **NumPy** (Numerical computation backend for Pearson correlation of numerical metadata with community usage)
- **AnnData** (File/object format from which data matrices can be imported ('These can be made from from pandas data matrices or imported from AnnData files/objects'))

## Examples

```
import pandas as pd; usage = pd.read_csv('community_usage.tsv', sep='\t', index_col=0); meta = pd.read_csv('sample_metadata.txt', sep='\t', index_col=0); scores = compute_enrichment(usage, meta); scores.to_csv('enrichment_scores.tsv', sep='\t')
```

## Evaluation signals

- enrichment_scores.tsv contains scores for every community-by-metadata-layer combination, covering both categorical over-representation and numerical Pearson correlations
- Missing numerical metadata values are excluded from Pearson correlations rather than treated as zeros or causing errors
- Known one-to-one associations are recovered, e.g. community C15 uniquely associated with PDGFRA alterations (proneural phenotype) in glioblastoma
- Over-representation scores distinguish observed versus expected category membership within each community rather than raw category counts
- Usage matrix and metadata table are joined on shared sample identifiers across all categorical and numerical layers

## Limitations

- A potential concern with mosaic integration is loss of relevant information between modalities given that not all features are shared, which can affect community composition and hence metadata associations
- The workflow assumes samples in the usage matrix can be matched to metadata rows; cohorts that 'do not all share the same samples' require mosaic handling before this step
- Categorical over-representation depends on category membership assignment of samples to communities, so ambiguous or low-usage assignments weaken the statistics

## Evidence

- [readme] Program communities are then interpreted using sample/cell metadata and gene set analyses.: [evidence span withheld — anchor status: external_doc]
