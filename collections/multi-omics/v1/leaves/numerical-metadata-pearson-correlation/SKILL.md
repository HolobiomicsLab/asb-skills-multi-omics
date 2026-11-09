---
name: numerical-metadata-pearson-correlation
description: Use when a community-by-sample usage matrix from mosaicMPI integration
  must be associated with numerical metadata layers (tumor purity, continuous scores)
  across samples to characterize landscape communities.
when_to_use_negative:
- When the goal is program discovery or integration itself — use cNMF factorization
  and the mosaicMPI landscape construction first; this skill only annotates existing
  communities.
- When metadata layers are purely categorical with no numerical values — over-representation
  testing alone applies, and no Pearson correlation can be computed.
- 'When datasets contain missing values for cNMF input: cNMF supports sparse (zero)
  data but not missing values, so imputation or exclusion must precede factorization.'
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0091
tools:
- name: mosaicMPI
  role: Provides the community-by-sample usage matrix from the integrated program
    landscape that is the basis for metadata association
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Loads the usage matrix and sample metadata table; data matrices can be made
    from pandas or imported from AnnData files
- name: NumPy
  role: Pearson correlation computation compatible with NumPy outputs (e.g. pearson.df.npz
    of programs within and between datasets)
- name: AnnData
  role: Container format from which matrices and metadata objects can be imported
    for the association analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:2849c96ab2b01273308ea828f6ed553050795cb5db3e9a733f2ba6c0244a447e
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# numerical-metadata-pearson-correlation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Associates mosaicMPI program-community usage with continuous sample metadata (e.g. tumor purity) by computing Pearson correlations between metadata values and community usage across samples, alongside categorical over-representation testing, to link communities to molecular/clinical phenotypes.

## When to use

Use when a community-by-sample usage matrix from mosaicMPI integration must be associated with numerical metadata layers (tumor purity, continuous scores) across samples to characterize landscape communities.

## When NOT to use

- When the goal is program discovery or integration itself — use cNMF factorization and the mosaicMPI landscape construction first; this skill only annotates existing communities.
- When metadata layers are purely categorical with no numerical values — over-representation testing alone applies, and no Pearson correlation can be computed.
- When datasets contain missing values for cNMF input: cNMF supports sparse (zero) data but not missing values, so imputation or exclusion must precede factorization.

## Inputs

- community-by-sample usage matrix produced by the mosaicMPI integration step
- sample metadata table with categorical layers (transcriptional subtypes, cell-type labels, driver alterations EGFR/NF1/PDGFRA/BRAF) and numerical layers (tumor purity)
- AnnData (.h5ad) or pandas data matrices as supported input formats

## Outputs

- enrichment_scores.tsv: community-by-metadata table combining Pearson correlations (numerical layers) and over-representation scores (categorical layers)
- identified community-metadata associations (e.g. C15–PDGFRA one-to-one association)

## How to apply

Load the community-by-sample usage matrix produced by the mosaicMPI integration step together with the sample metadata table using pandas, keeping categorical layers (driver alterations EGFR/NF1/PDGFRA/BRAF, transcriptional subtypes, cell types) separate from numerical layers (tumor purity). For each numerical metadata layer, compute the Pearson correlation between the metadata values and each community's usage across samples, dropping samples with missing numerical values before correlation. Assemble these correlation scores together with the categorical over-representation statistics into a single community-by-metadata enrichment table written as enrichment_scores.tsv. This statistical propagation of cohort annotations onto the program network is how mosaicMPI yields a characterized landscape of biological themes, such as linking community C15 one-to-one with PDGFRA alterations and a proneural phenotype.

## Related tools

- **mosaicMPI** (Provides the community-by-sample usage matrix from the integrated program landscape that is the basis for metadata association) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Loads the usage matrix and sample metadata table; data matrices can be made from pandas or imported from AnnData files)
- **NumPy** (Pearson correlation computation compatible with NumPy outputs (e.g. pearson.df.npz of programs within and between datasets))
- **AnnData** (Container format from which matrices and metadata objects can be imported for the association analysis)

## Examples

```
import pandas as pd; usage = pd.read_csv('usage.tsv', sep='\t', index_col=0); meta = pd.read_csv('metadata.txt', sep='\t', index_col=0); corr = usage.apply(lambda c: c.corr(meta['tumor_purity'].dropna(index=c.index)))
```

## Evaluation signals

- enrichment_scores.tsv is produced containing both over-representation scores (categorical layers) and Pearson correlations (numerical layers) for every community
- Pearson correlations for numerical layers such as tumor purity are computed over samples with non-missing values only
- Known biology is recovered, e.g. community C15 shows a one-to-one association with PDGFRA alterations consistent with a proneural tumor phenotype
- The usage matrix dimensions (communities x samples) match the metadata table sample identifiers before scoring

## Limitations

- cNMF supports sparse input data (with zeros) but not missing values, so metadata or expression matrices with missing entries need handling before factorization
- Mosaic integration may lose relevant information between modalities because not all features are shared across datasets
- Associations are statistical propagations of cohort annotations and depend on the quality/coverage of the input metadata layers

## Evidence

- [other] For each numerical metadata layer, compute the Pearson correlation between the metadata values and community usage across samples, ignoring missing numerical values.: [evidence span withheld — anchor status: not_found]
- [other] Assemble the over-representation and correlation scores into a single community-by-metadata enrichment table and write it as enrichment_scores.tsv.: [evidence span withheld — anchor status: not_found]
