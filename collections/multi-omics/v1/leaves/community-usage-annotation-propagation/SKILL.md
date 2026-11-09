---
name: community-usage-annotation-propagation
description: Use when you have a community-by-sample usage matrix from mosaicMPI integration
  plus a sample metadata table (categorical layers like transcriptional subtypes,
  cell-type labels, driver alterations;
when_to_use_negative:
- Input data contains missing (NaN) values that must be fed to cNMF — cNMF supports
  sparse data with zeros but not missing values, so impute or handle before factorization-level
  steps
- You need to discover programs or build the integrated landscape itself — this skill
  operates downstream, after the community-by-sample usage matrix already exists
- The goal is gene-level functional enrichment (e.g. ssGSEA of cNMF programs with
  GO:BP terms) rather than sample-metadata association of communities
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: mosaicMPI
  role: produces the program communities and community-by-sample usage matrix whose
    metadata associations this skill computes; annotations are statistically propagated
    onto the program network
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: loads the usage matrix and metadata table and assembles the community-by-metadata
    enrichment table (enrichment_scores.tsv)
- name: NumPy
  role: computes Pearson correlations between numerical metadata (e.g. tumor purity)
    and community usage across samples
- name: AnnData
  role: input container format for data/metadata matrices that can be imported alongside
    pandas data matrices
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:23810a71cc8c3cc435d69cad0f3b66f43eda15b4eea30bfd7329b7cf6a38580a
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# community-usage-annotation-propagation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Associates mosaicMPI program community usage (community-by-sample matrix) with categorical and numerical sample metadata — driver gene alterations (EGFR/NF1/PDGFRA/BRAF), transcriptional subtypes, cell types, and tumor purity — via over-representation statistics and Pearson correlations to link genotypes to tumor phenotypes.

## When to use

Use when you have a community-by-sample usage matrix from mosaicMPI integration plus a sample metadata table (categorical layers like transcriptional subtypes, cell-type labels, driver alterations; numerical layers like tumor purity) and need to characterize which metadata each community is enriched for or correlated with.

## When NOT to use

- Input data contains missing (NaN) values that must be fed to cNMF — cNMF supports sparse data with zeros but not missing values, so impute or handle before factorization-level steps
- You need to discover programs or build the integrated landscape itself — this skill operates downstream, after the community-by-sample usage matrix already exists
- The goal is gene-level functional enrichment (e.g. ssGSEA of cNMF programs with GO:BP terms) rather than sample-metadata association of communities

## Inputs

- community-by-sample usage matrix from mosaicMPI integration (e.g. TSV/h5ad)
- sample metadata table with categorical layers (transcriptional subtypes, cell-type labels, driver alterations EGFR/NF1/PDGFRA/BRAF) and numerical layers (tumor purity)
- AnnData (.h5ad) objects or pandas data matrices

## Outputs

- community-by-metadata enrichment table (enrichment_scores.tsv) containing over-representation scores for categorical layers and Pearson correlations for numerical layers

## How to apply

Load the community-by-sample usage matrix produced by the mosaicMPI integration step together with the sample metadata table using pandas (or from AnnData objects). For each categorical metadata layer, compute an over-representation statistic per category within each community, comparing observed versus expected category membership across samples assigned to that community. For each numerical metadata layer (e.g. tumor purity), compute the Pearson correlation between metadata values and community usage across samples, excluding missing numerical values. Assemble over-representation and correlation scores into a single community-by-metadata enrichment table written as enrichment_scores.tsv. This operationalizes mosaicMPI's statistical propagation of clinical and molecular annotations onto the program network; a correct result recovers one-to-one community-metadata associations such as community C15 being uniquely linked to PDGFRA alterations and a proneural tumor phenotype.

## Related tools

- **mosaicMPI** (produces the program communities and community-by-sample usage matrix whose metadata associations this skill computes; annotations are statistically propagated onto the program network) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (loads the usage matrix and metadata table and assembles the community-by-metadata enrichment table (enrichment_scores.tsv))
- **NumPy** (computes Pearson correlations between numerical metadata (e.g. tumor purity) and community usage across samples)
- **AnnData** (input container format for data/metadata matrices that can be imported alongside pandas data matrices)

## Examples

```
import pandas as pd; usage = pd.read_csv('community_usage.tsv', sep='\t', index_col=0); meta = pd.read_csv('samples.metadata.txt', sep='\t', index_col=0); # over-representation per categorical layer, Pearson corr for numeric layers (e.g. tumor purity) -> enrichment_scores.tsv
```

## Evaluation signals

- enrichment_scores.tsv exists and contains rows for every community and columns for every categorical and numerical metadata layer
- Categorical over-representation values compare observed versus expected category membership per community (enrichment/depletion sign and magnitude are interpretable)
- Numerical layers yield Pearson correlation values in [-1, 1] with missing values excluded rather than treated as zeros
- Known association is recovered: community C15 shows a one-to-one association with PDGFRA alterations, uniquely linking this driver genotype to a proneural tumor phenotype
- No community-metadata pair is silently dropped: samples with missing numerical metadata are ignored, not erroring the correlation step

## Limitations

- Associations are statistical propagations onto the program network; they do not establish causality between driver alterations and community-defined phenotypes
- Mosaic integration may lose relevant information between modalities because not all features are shared, which can affect downstream metadata associations
- Datasets without the metadata layer of interest contribute no signal for that layer, so enrichment statistics depend on which cohorts carry which annotations

## Evidence

- [abstract] Clinical and molecular annotations from cohorts are statistically propagated onto this network of programs: "Clinical and molecular annotations from cohorts are statistically propagated onto this network of programs"
- [readme] Program communities are then interpreted using sample/cell metadata and gene set analyses.: [evidence span withheld — anchor status: external_doc]
