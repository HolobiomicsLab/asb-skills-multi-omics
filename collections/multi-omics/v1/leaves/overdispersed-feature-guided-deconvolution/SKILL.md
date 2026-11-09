---
name: overdispersed-feature-guided-deconvolution
description: Use when each modality (bulk RNA-Seq, MS proteomics, snRNA-Seq) has been
  normalized and overdispersed features selected, and unbiased low- to high-resolution
  molecular programs are needed within each dataset prior to integration.
when_to_use_negative:
- Data contains missing values — cNMF supports sparse (zero) input but not missing
  values
- The goal is integration across datasets rather than within-dataset program discovery
  — deconvolution here is performed independently on each dataset, not on merged data
- Features have not yet been restricted to overdispersed/HVF sets via the preceding
  select-hvf step
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: mosaicMPI
  role: Provides the CLI subcommands (initialize-cnmf, factorize, postprocess) and
    SLURM scripts that drive the rank-range cNMF factorization workflow and write
    per-dataset .h5ad outputs
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: cNMF
  role: Consensus non-negative matrix factorization engine used to discover low- to
    high-resolution programs within each dataset over ranks 2-60
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: AnnData
  role: Container format for writing each dataset's input data plus cNMF program feature
    scores and usages to .h5ad files
- name: pandas
  role: Data matrices can be made from pandas or imported from AnnData files/objects
    as mosaicMPI inputs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:63778b64b7ef79e97e20d00c2d78ed29ef12eda2b1758f98f9a75d0c1cbfcedd
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# overdispersed-feature-guided-deconvolution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs consensus non-negative matrix factorization (cNMF) independently on each dataset over a rank range (k = 2-60) using overdispersed feature sets, producing per-rank program feature scores and usages that serve as multi-resolution anchors for mosaic integration.

## When to use

Use when each modality (bulk RNA-Seq, MS proteomics, snRNA-Seq) has been normalized and overdispersed features selected, and unbiased low- to high-resolution molecular programs are needed within each dataset prior to integration.

## When NOT to use

- Data contains missing values — cNMF supports sparse (zero) input but not missing values
- The goal is integration across datasets rather than within-dataset program discovery — deconvolution here is performed independently on each dataset, not on merged data
- Features have not yet been restricted to overdispersed/HVF sets via the preceding select-hvf step

## Inputs

- Normalized per-dataset expression matrices with overdispersed feature sets selected (e.g. cptac_snRNA_subsampled.h5ad from txt-to-h5ad conversion)
- Per-dataset AnnData .h5ad files produced by mosaicMPI preprocessing
- Rank range specification (k = 2-60, with ranks > 10 restricted to mod(k,5) = 0)

## Outputs

- cNMF program feature scores and usages at each retained rank per dataset
- Per-dataset AnnData .h5ad files (e.g. results/cptac_snrna/cptac_snrna.h5ad) containing input data plus cNMF solutions
- Consensus program refits across all features via non-negative least squares

## How to apply

Initialize cNMF per dataset with `mosaicmpi initialize-cnmf` over k = 2-60 (e.g. --k_range 2 60 1), restricting ranks above k = 10 to those with mod(k,5) = 0, using the overdispersed feature sets from the prior HVF-selection step. Factorize with `mosaicmpi factorize` using 200 replicates and the KL beta-loss, optionally parallelizing per-rank jobs through the provided slurm_factorize.sh script. Then run `mosaicmpi postprocess` to verify all factorizations completed, build consensus programs and usages, and perform the non-negative least-squares re-fit of identified programs across all features of each dataset (not just the overdispersed subset). Finally write each dataset's input data plus cNMF program feature scores and usages at each retained rank to a per-dataset AnnData .h5ad file such as results/cptac_snrna/cptac_snrna.h5ad.

## Related tools

- **mosaicMPI** (Provides the CLI subcommands (initialize-cnmf, factorize, postprocess) and SLURM scripts that drive the rank-range cNMF factorization workflow and write per-dataset .h5ad outputs) — https://github.com/MorrissyLab/mosaicMPI
- **cNMF** (Consensus non-negative matrix factorization engine used to discover low- to high-resolution programs within each dataset over ranks 2-60) — https://github.com/MorrissyLab/mosaicMPI
- **AnnData** (Container format for writing each dataset's input data plus cNMF program feature scores and usages to .h5ad files)
- **pandas** (Data matrices can be made from pandas or imported from AnnData files/objects as mosaicMPI inputs)

## Examples

```
mosaicmpi initialize-cnmf -n cptac_snrna.h5ad --k_range 2 60 1; mosaicmpi factorize -n cptac_snrna.h5ad --replicates 200 --beta_loss KL; mosaicmpi postprocess -n cptac_snrna.h5ad
```

## Evaluation signals

- mosaicmpi postprocess confirms that every factorization completed successfully across all ranks and replicates
- Each output .h5ad file contains the cNMF solution (program feature scores and usages) at each retained rank alongside the original input data
- Rank set matches the specification: k = 2-10 at unit spacing plus ranks > 10 with mod(k,5) = 0 (i.e. 15, 20, ..., 60)
- NNLS refit programs cover all features of each dataset, not only the overdispersed subset used during factorization

## Limitations

- cNMF supports sparse input (zeros) but not missing values in the data matrix
- Factorization for many ranks is computationally heavy; per-rank parallelization via SLURM (slurm_factorize.sh) is recommended, and memory usage depends on dataset size
- No software version information was stated for the tools in the source article

## Evidence

- [methods] cNMF is run by default for ranks 2-60 independently for each dataset, enabling unbiased multi-resolution program discovery within each modality prior to integration.: [evidence span withheld — anchor status: not_found]
- [readme] Programs are first discovered using unsupervised deconvolution (consensus non-negative matrix factorization, cNMF) across multiple ranks separately for each dataset.: [evidence span withheld — anchor status: external_doc]
