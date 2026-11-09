---
name: categorical-metadata-overrepresentation-statistics
description: Use when a community-by-sample usage matrix from mosaicMPI integration
  is available and you need to associate program communities with sample/cell metadata
  (categorical layers via over-representation, numerical layers via correlation).
when_to_use_negative:
- Input is raw expression data that has not yet been factorized with cNMF and integrated
  into mosaicMPI program communities — program discovery and integration must precede
  metadata association.
- Metadata layers contain only missing values for the samples of a community, so observed-versus-expected
  category comparison is not computable.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0091
tools:
- name: mosaicMPI
  role: Provides the program communities and the community-by-sample usage matrix
    that the metadata statistics are computed on; downstream analyses use community-specific
    metadata from every input modality.
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Loads the community-by-sample usage matrix and the sample metadata table (categorical
    and numerical layers) for the over-representation and correlation computations.
- name: NumPy
  role: Supports numerical computation of the over-representation statistics and Pearson
    correlations across samples.
- name: AnnData
  role: Container format for the mosaicMPI datasets that carry sample/cell metadata
    used in the association analysis.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:944e45ebc31d1d621281356481d176b9fd927c59e1c1f515d9181b10da607070
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
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# categorical-metadata-overrepresentation-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes over-representation of categorical metadata categories (e.g. transcriptional subtypes, cell-type labels, driver alterations EGFR/NF1/PDGFRA/BRAF) within mosaicMPI program communities, and Pearson correlations of numerical metadata (e.g. tumor purity) with community usage, to characterize landscape communities.

## When to use

Use when a community-by-sample usage matrix from mosaicMPI integration is available and you need to associate program communities with sample/cell metadata (categorical layers via over-representation, numerical layers via correlation).

## When NOT to use

- Input is raw expression data that has not yet been factorized with cNMF and integrated into mosaicMPI program communities — program discovery and integration must precede metadata association.
- Metadata layers contain only missing values for the samples of a community, so observed-versus-expected category comparison is not computable.

## Inputs

- community-by-sample usage matrix produced by the mosaicMPI integration step
- sample metadata table (TSV) with categorical layers (transcriptional subtypes, cell-type labels, driver alterations EGFR/NF1/PDGFRA/BRAF) and numerical layers (tumor purity)
- AnnData (.h5ad) dataset objects produced via mosaicmpi txt-to-h5ad

## Outputs

- enrichment_scores.tsv: community-by-metadata enrichment table combining over-representation statistics and Pearson correlation scores

## How to apply

Load the community-by-sample usage matrix produced by the mosaicMPI integration step together with the sample metadata table (categorical layers such as transcriptional subtypes, cell-type labels, and driver alterations EGFR/NF1/PDGFRA/BRAF; numerical layers such as tumor purity) using pandas. For each categorical metadata layer, compute an over-representation statistic for each category within each community, comparing observed versus expected category membership across samples assigned to that community. For each numerical metadata layer, compute the Pearson correlation between the metadata values and community usage across samples, ignoring missing numerical values. Assemble over-representation and correlation scores into a single community-by-metadata enrichment table and write it as enrichment_scores.tsv. The rationale is mosaicMPI's design of statistically propagating clinical and molecular annotations onto the program network to yield a characterized landscape of biological themes.

## Related tools

- **mosaicMPI** (Provides the program communities and the community-by-sample usage matrix that the metadata statistics are computed on; downstream analyses use community-specific metadata from every input modality.) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Loads the community-by-sample usage matrix and the sample metadata table (categorical and numerical layers) for the over-representation and correlation computations.)
- **NumPy** (Supports numerical computation of the over-representation statistics and Pearson correlations across samples.)
- **AnnData** (Container format for the mosaicMPI datasets that carry sample/cell metadata used in the association analysis.)

## Examples

```
import pandas as pd; usage = pd.read_csv('community_usage.tsv', sep='\t', index_col=0); meta = pd.read_csv('samples.metadata.txt', sep='\t', index_col=0); # over-representation per category per community; Pearson corr for tumor purity ignoring NaNs
```

## Evaluation signals

- enrichment_scores.tsv contains one row per community and columns for every categorical and numerical metadata layer examined.
- Each community's over-representation values reflect observed versus expected category membership computed over the samples assigned to that community.
- Pearson correlations for numerical metadata (e.g. tumor purity) are computed only over samples with non-missing values.
- Known biology is recovered: community C15 shows a one-to-one association with PDGFRA alterations, uniquely linking this driver genotype to a proneural tumor phenotype.
- The community-by-sample usage matrix used as input is identical to the output of the mosaicMPI integration step.

## Limitations

- A potential concern with mosaic integration is loss of relevant information between modalities given that not all features are shared.
- cNMF supports input data that is sparse (i.e. with zeros), but not with missing values — cNMF does not support missing values.
- No explicit code or data availability statement appears in some sections; software version numbers should be checked via mosaicmpi --help.

## Evidence

- [abstract] Clinical and molecular annotations from cohorts are statistically propagated onto this network of programs, yielding a richly characterized landscape of biological themes.: "Clinical and molecular annotations from cohorts are statistically propagated onto this network of programs, yielding a richly characterized landscape of biological themes."
- [discussion] mosaicMPI can utilize community-specific metadata from every input modality: "mosaicMPI can utilize community-specific metadata from every input modality, thereby enabling coordinated multi-dataset annotation of an integrated biological landscape"
- [methods] These can be made from from pandas data matrices or imported from AnnData files/objects.: "These can be made from from pandas data matrices or imported from AnnData files/objects."
- [methods] `pearson.df.npz`: pearson correlation matrix of all programs within and between datasets, compatible with [NumPy]: "`pearson.df.npz`: pearson correlation matrix of all programs within and between datasets, compatible with [NumPy]"
- [readme] Program communities are then interpreted using sample/cell metadata and gene set analyses.: [evidence span withheld — anchor status: external_doc]
