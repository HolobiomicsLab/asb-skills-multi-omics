---
name: consensus-nmf-factorization
description: Use when performing unbiased multi-resolution program discovery within
  a single modality (bulk RNA-Seq, MS proteomics, or snRNA-Seq) prior to cross-dataset
  integration, starting from over-dispersed feature sets.
when_to_use_negative:
- The input data matrix contains missing values (NaNs) rather than sparse zeros, since
  cNMF supports sparse input but not missing values
- The goal is integration or comparison of programs across datasets — cNMF factorization
  here is performed independently within each dataset; cross-dataset grouping is a
  later mosaicMPI step
- A single fixed rank is already determined and multi-resolution exploration is not
  needed — mosaicMPI's strength is spanning coarse-grained to nuanced resolutions
  via multiple ranks
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3520
tools:
- name: mosaicMPI
  role: Framework providing the CLI subcommands (initialize-cnmf, factorize, postprocess)
    that wrap and orchestrate the cNMF runs and write per-dataset results
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: cNMF
  role: Consensus non-negative matrix factorization engine that discovers the molecular
    programs at each rank using 200 replicates and KL beta-loss
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: AnnData
  role: File/object format used to store each dataset's input data plus cNMF program
    feature scores and usages as .h5ad
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Data-matrix format from which mosaicMPI objects and AnnData files can be constructed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:6f76217a0df6fdd275df5d8641ae0b5e67c74a01c2e122d6aa5c23b031071dae
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
  - cNMF
  - mosaicMPI
  - pandas
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# consensus-nmf-factorization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs consensus non-negative matrix factorization (cNMF) independently on each dataset across a range of ranks (k = 2-60) to discover low- to high-resolution molecular programs that serve as anchors for mosaic integration in mosaicMPI.

## When to use

Use when performing unbiased multi-resolution program discovery within a single modality (bulk RNA-Seq, MS proteomics, or snRNA-Seq) prior to cross-dataset integration, starting from over-dispersed feature sets.

## When NOT to use

- The input data matrix contains missing values (NaNs) rather than sparse zeros, since cNMF supports sparse input but not missing values
- The goal is integration or comparison of programs across datasets — cNMF factorization here is performed independently within each dataset; cross-dataset grouping is a later mosaicMPI step
- A single fixed rank is already determined and multi-resolution exploration is not needed — mosaicMPI's strength is spanning coarse-grained to nuanced resolutions via multiple ranks

## Inputs

- Over-dispersed (highly-variable) feature sets produced by select-hvf for each dataset
- Normalized expression matrices for bulk RNA-Seq, MS proteomics, and snRNA-Seq (sparse matrices with zeros supported; missing values not supported)
- Initialized cNMF run directories per dataset

## Outputs

- cNMF program feature scores and usages at each retained rank (k = 2-60) per dataset
- Per-dataset AnnData .h5ad files (e.g. results/cptac_snrna/cptac_snrna.h5ad) containing input data plus cNMF solutions

## How to apply

Initialize cNMF for each dataset with `mosaicmpi initialize-cnmf` over the rank range k = 2-60 (e.g. `--k_range 2 60 1`), restricting ranks above k = 10 to those with mod(k,5) = 0, using the over-dispersed feature sets from prior HVF selection. Factorize each initialized run with `mosaicmpi factorize` using 200 replicates and the KL beta-loss, optionally parallelizing per-rank jobs via the mosaicMPI SLURM scripts (slurm_factorize.sh). Run `mosaicmpi postprocess` to verify all factorizations completed, then build consensus programs and usages and perform the non-negative least-squares re-fit of the identified programs across all features of each dataset. Finally, write each dataset's results (input data plus cNMF program feature scores and usages at each retained rank) to a per-dataset AnnData .h5ad file such as results/cptac_snrna/cptac_snrna.h5ad.

## Related tools

- **mosaicMPI** (Framework providing the CLI subcommands (initialize-cnmf, factorize, postprocess) that wrap and orchestrate the cNMF runs and write per-dataset results) — https://github.com/MorrissyLab/mosaicMPI
- **cNMF** (Consensus non-negative matrix factorization engine that discovers the molecular programs at each rank using 200 replicates and KL beta-loss) — https://github.com/MorrissyLab/mosaicMPI
- **AnnData** (File/object format used to store each dataset's input data plus cNMF program feature scores and usages as .h5ad) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Data-matrix format from which mosaicMPI objects and AnnData files can be constructed)

## Examples

```
mosaicmpi initialize-cnmf -o results/cptac_snrna -n cptac_snrna.h5ad --k_range 2 60 1 && mosaicmpi factorize -o results/cptac_snrna --cores 8
```

## Evaluation signals

- `mosaicmpi postprocess` confirms that every factorization job completed successfully
- Each output .h5ad file (e.g. cptac_snrna.h5ad) contains the cNMF solution — program feature scores and usages for each retained rank
- The rank range covers k = 2-60, with ranks above k = 10 restricted to mod(k,5) = 0 (i.e. 15, 20, 25, ...)
- Consensus step includes the non-negative least-squares re-fit of identified programs across all features, not just the over-dispersed subset
- Ranks that failed quality control are removed via the max_k_filter based on max_k_median_corr before consensus

## Limitations

- cNMF supports input data that is sparse (with zeros) but not with missing values
- Ranks may fail the max_k_filter based on max_k_median_corr, so not every rank in the specified range yields a retained consensus solution
- Memory usage depends on the size and number of datasets; factorization can be compute-intensive, requiring parallel per-rank SLURM jobs for large datasets

## Evidence

- [methods] workflow: [evidence span withheld — anchor status: not_found]
- [readme] readme: [evidence span withheld — anchor status: external_doc]
