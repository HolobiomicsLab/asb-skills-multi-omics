---
name: multi-omics-program-discovery
description: Use when starting mosaic integration of datasets with mismatched features
  or modalities, and each dataset must first be factorized independently into programs
  across a range of resolutions before cross-dataset integration.
when_to_use_negative:
- Datasets contain missing values — cNMF supports sparse (zero-containing) input but
  not missing values, so imputation or exclusion is needed first
- The goal is direct feature-level integration requiring a shared gene/feature intersection
  — mosaicMPI is designed to avoid subsetting features to an intersection or highly-variable
  subset
- Only a single fixed rank/resolution is of interest — this skill's value is spanning
  low to high resolutions across ranks 2-60
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: mosaicMPI
  role: CLI framework providing initialize-cnmf, factorize, postprocess, and txt-to-h5ad
    subcommands that run and package the per-dataset cNMF factorizations
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: cNMF
  role: Consensus non-negative matrix factorization engine that discovers programs
    within each dataset across ranks 2-60 with 200 replicates and KL beta-loss
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: AnnData
  role: Container format for the per-dataset .h5ad outputs holding input data plus
    program feature scores and usages
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Source format for data matrices that can be converted into AnnData objects
    for the workflow
  repo: https://github.com/MorrissyLab/mosaicMPI
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:b4c642f6ab11add3828b18f21d280abceb4a1226371d0d833e85d27473bc85d4
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# multi-omics-program-discovery

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Unbiased, multi-resolution discovery of molecular programs within each modality (bulk RNA-Seq, MS proteomics, snRNA-Seq) by running consensus non-negative matrix factorization (cNMF) independently per dataset across ranks 2-60, yielding the program anchors for mosaicMPI integration.

## When to use

Use when starting mosaic integration of datasets with mismatched features or modalities, and each dataset must first be factorized independently into programs across a range of resolutions before cross-dataset integration.

## When NOT to use

- Datasets contain missing values — cNMF supports sparse (zero-containing) input but not missing values, so imputation or exclusion is needed first
- The goal is direct feature-level integration requiring a shared gene/feature intersection — mosaicMPI is designed to avoid subsetting features to an intersection or highly-variable subset
- Only a single fixed rank/resolution is of interest — this skill's value is spanning low to high resolutions across ranks 2-60

## Inputs

- Normalized expression matrix per dataset (e.g. cptac_snRNA_subsampled.txt converted to AnnData .h5ad via mosaicmpi txt-to-h5ad)
- Overdispersed/HVF-selected feature sets per dataset (from select-hvf)
- CPTAC modality datasets: bulk RNA-Seq, MS proteomics, snRNA-Seq
- Rank range specification (k = 2-60, ranks >10 restricted to mod(k,5) = 0)

## Outputs

- cNMF consensus program feature scores and usages at each retained rank per dataset
- Per-dataset AnnData .h5ad files containing input data plus cNMF solutions (e.g. results/cptac_snrna/cptac_snrna.h5ad)
- Postprocess report confirming all factorizations completed (with max_k_filter / max_k_median_corr diagnostics)

## How to apply

Initialize cNMF independently for each dataset with `mosaicmpi initialize-cnmf` over the rank range k = 2-60 (e.g. --k_range 2 60 1), restricting ranks above k = 10 to those satisfying mod(k,5) = 0, using the overdispersed feature sets selected in the preceding HVF-selection step. Factorize each initialized run with `mosaicmpi factorize` using 200 replicates and the KL beta-loss, optionally parallelizing per-rank jobs through the repository's slurm_factorize.sh SLURM scripts. Run `mosaicmpi postprocess` to confirm all factorizations completed, build consensus programs and usages, and perform the non-negative least-squares re-fit of the identified programs across all features of each dataset (not just the over-dispersed subset). Finally write each dataset's input data plus cNMF program feature scores and usages at each retained rank to a per-dataset AnnData .h5ad file (e.g. results/cptac_snrna/cptac_snrna.h5ad). The rationale is that deconvolution is performed separately per dataset so integration remains incremental and does not require subsetting features to a shared intersection.

## Related tools

- **mosaicMPI** (CLI framework providing initialize-cnmf, factorize, postprocess, and txt-to-h5ad subcommands that run and package the per-dataset cNMF factorizations) — https://github.com/MorrissyLab/mosaicMPI
- **cNMF** (Consensus non-negative matrix factorization engine that discovers programs within each dataset across ranks 2-60 with 200 replicates and KL beta-loss) — https://github.com/MorrissyLab/mosaicMPI
- **AnnData** (Container format for the per-dataset .h5ad outputs holding input data plus program feature scores and usages) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Source format for data matrices that can be converted into AnnData objects for the workflow) — https://github.com/MorrissyLab/mosaicMPI

## Examples

```
mosaicmpi initialize-cnmf -n cptac_snrna.h5ad --k_range 2 60 1
```

## Evaluation signals

- `mosaicmpi postprocess` confirms that every factorization job completed successfully
- Each output .h5ad file contains the cNMF solution (program feature scores and usages) alongside the input data
- Retained ranks follow the intended scheme: all ranks 2-10 plus ranks >10 with mod(k,5) = 0 up to 60
- Consensus solutions exist for each dataset at each retained rank after the 200-replicate KL beta-loss factorization
- Programs are re-fit across all features per dataset via non-negative least squares, not restricted to the over-dispersed feature subset

## Limitations

- cNMF supports input data that is sparse (with zeros) but not with missing values
- Factorization can be computationally heavy across ranks 2-60; parallel per-rank SLURM jobs (slurm_factorize.sh, slurm_postprocess.sh) are recommended
- A potential concern with mosaic integration is loss of relevant information between modalities given that not all features are shared

## Evidence

- [readme] Programs are first discovered using unsupervised deconvolution (consensus non-negative matrix factorization, cNMF) across multiple ranks separately for each dataset.: [evidence span withheld — anchor status: external_doc]
