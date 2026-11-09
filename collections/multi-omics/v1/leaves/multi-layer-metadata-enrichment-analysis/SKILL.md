---
name: multi-layer-metadata-enrichment-analysis
description: Use when you have a community-by-sample usage matrix from a mosaicMPI
  landscape integration plus a sample metadata table, and need to link communities
  to driver genotypes, subtypes, cell types, or purity.
when_to_use_negative:
- When no community-level integration has been performed yet (you only have per-dataset
  cNMF programs, not landscape communities and their sample usage).
- When metadata layers are per-program gene-set results rather than per-sample categorical/numerical
  annotations — pathway or ssGSEA enrichment of programs is a different skill.
- When the data contain missing values in a cNMF factorization step — cNMF supports
  sparse (zeros) but not missing values, so imputation or exclusion must happen before
  program discovery, not in this enrichment step.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: mosaicMPI
  role: Produces the program landscape and community-by-sample usage matrix whose
    communities are annotated with cohort metadata in this skill
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Loads the usage matrix and sample metadata tables and assembles the community-by-metadata
    enrichment table (enrichment_scores.tsv)
- name: NumPy
  role: Pearson correlation computation between numerical metadata layers (e.g. tumor
    purity) and community usage across samples
- name: AnnData
  role: 'Alternative input format: usage matrices and metadata can be imported from
    AnnData files/objects instead of pandas matrices'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:09298739bb4e3becaa7ef3d96777880333273e5e6b27350bcad05cde57a113a5
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
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# multi-layer-metadata-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistically associates mosaicMPI program-community usage with sample metadata: over-representation tests for categorical layers (driver alterations like EGFR/NF1/PDGFRA/BRAF, transcriptional subtypes, cell types) and Pearson correlations for numerical layers (e.g. tumor purity), yielding a community-by-metadata enrichment table.

## When to use

Use when you have a community-by-sample usage matrix from a mosaicMPI landscape integration plus a sample metadata table, and need to link communities to driver genotypes, subtypes, cell types, or purity.

## When NOT to use

- When no community-level integration has been performed yet (you only have per-dataset cNMF programs, not landscape communities and their sample usage).
- When metadata layers are per-program gene-set results rather than per-sample categorical/numerical annotations — pathway or ssGSEA enrichment of programs is a different skill.
- When the data contain missing values in a cNMF factorization step — cNMF supports sparse (zeros) but not missing values, so imputation or exclusion must happen before program discovery, not in this enrichment step.

## Inputs

- Community-by-sample usage matrix from the mosaicMPI landscape integration (e.g. TSV of community usage per sample)
- Sample metadata table (TSV) with categorical layers: transcriptional subtypes, cell-type labels, driver alterations (EGFR/NF1/PDGFRA/BRAF)
- Sample metadata numerical layers: e.g. tumor purity

## Outputs

- enrichment_scores.tsv — community-by-metadata table of over-representation statistics (categorical layers) and Pearson correlations (numerical layers)

## How to apply

Load the community-by-sample usage matrix produced by the mosaicMPI integration step together with the sample metadata table (categorical layers such as transcriptional subtypes, cell-type labels, and driver alterations EGFR/NF1/PDGFRA/BRAF; numerical layers such as tumor purity) using pandas. For each categorical metadata layer, compute an over-representation statistic for each category within each community, comparing observed versus expected category membership across samples assigned to that community — this mirrors how mosaicMPI statistically propagates cohort annotations onto the program network. For each numerical metadata layer, compute the Pearson correlation between metadata values and community usage across samples, ignoring missing numerical values. Assemble both score types into a single community-by-metadata enrichment table and write it as enrichment_scores.tsv. Decision point: a strong one-to-one category–community association (e.g. C15 uniquely tagging PDGFRA alterations) identifies driver-genotype-linked tumor phenotypes within the landscape.

## Related tools

- **mosaicMPI** (Produces the program landscape and community-by-sample usage matrix whose communities are annotated with cohort metadata in this skill) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Loads the usage matrix and sample metadata tables and assembles the community-by-metadata enrichment table (enrichment_scores.tsv))
- **NumPy** (Pearson correlation computation between numerical metadata layers (e.g. tumor purity) and community usage across samples)
- **AnnData** (Alternative input format: usage matrices and metadata can be imported from AnnData files/objects instead of pandas matrices)

## Examples

```
import pandas as pd; usage = pd.read_csv('community_usage.tsv', sep='\t', index_col=0); meta = pd.read_csv('metadata.txt', sep='\t', index_col=0); corr = usage.apply(lambda c: c.corr(meta['tumor_purity']))
```

## Evaluation signals

- enrichment_scores.tsv contains one row per community and one column per metadata category/layer, covering both categorical over-representation and numerical correlation scores.
- Pearson correlations are computed only over samples with non-missing numerical metadata values (no NaN propagation into the table).
- Known biology is recovered: community C15 shows a one-to-one association with PDGFRA alterations, uniquely linking that driver genotype to a proneural tumor phenotype.
- Categorical over-representation compares observed versus expected category membership per community, not raw category counts.
- Community labels and sample identifiers in the output match those of the upstream mosaicMPI usage matrix.

## Limitations

- Mosaic integration may lose relevant information between modalities because not all features are shared, which can affect community composition and hence metadata associations.
- Categorical over-representation requires samples to be assigned to communities; sparse or ambiguous community assignment across datasets weakens the statistics.
- Numerical layers with many missing values reduce correlation power since missing values are ignored rather than imputed.

## Evidence
