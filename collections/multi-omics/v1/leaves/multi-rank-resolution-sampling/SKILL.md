---
name: multi-rank-resolution-sampling
description: Use when you have a normalized expression matrix (bulk RNA-Seq, MS proteomics,
  or snRNA-Seq) with over-dispersed features already selected, and you need unbiased
  program anchors spanning coarse to fine resolutions rather than a single fixed k.
when_to_use_negative:
- Your dataset contains missing values — cNMF supports sparse (zero) input but not
  NaNs, so impute or mask features first
- You only need programs at a single known resolution/k — the multi-rank sweep and
  mod(k,5)=0 rank filtering would be unnecessary overhead
- Over-dispersed features have not been selected yet (no select-hvf output) — factorize
  should be run on the over-dispersed feature subset, not raw features
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3307
tools:
- name: mosaicMPI
  role: Provides the CLI workflow (initialize-cnmf, factorize, postprocess subcommands)
    and SLURM scripts that execute the multi-rank cNMF sweep per dataset
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: cNMF
  role: Consensus non-negative matrix factorization engine used to discover programs
    at each rank, with NNLS re-fit across all features
- name: AnnData
  role: Container format for the per-dataset output .h5ad files holding program feature
    scores and usages
- name: pandas
  role: Data matrix source that can be converted into AnnData/h5ad inputs for the
    workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:faa19fb5d8cb7eb56e41615feacf4f78042cb924e244b6c39807900e9567d93d
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# multi-rank-resolution-sampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Discover molecular programs at multiple resolutions by running cNMF independently over a rank sweep (k = 2–60, with ranks above k = 10 restricted to mod(k,5) = 0) for each dataset, using mosaicMPI's initialize-cnmf/factorize/postprocess workflow before any cross-dataset integration.

## When to use

Use when you have a normalized expression matrix (bulk RNA-Seq, MS proteomics, or snRNA-Seq) with over-dispersed features already selected, and you need unbiased program anchors spanning coarse to fine resolutions rather than a single fixed k.

## When NOT to use

- Your dataset contains missing values — cNMF supports sparse (zero) input but not NaNs, so impute or mask features first
- You only need programs at a single known resolution/k — the multi-rank sweep and mod(k,5)=0 rank filtering would be unnecessary overhead
- Over-dispersed features have not been selected yet (no select-hvf output) — factorize should be run on the over-dispersed feature subset, not raw features

## Inputs

- Normalized per-dataset expression matrix (bulk RNA-Seq, MS proteomics, or snRNA-Seq; sparse zeros allowed, no missing values)
- Over-dispersed (HVF) feature set produced by mosaicmpi select-hvf
- Rank-range parameters (k = 2–60; mod(k,5)=0 filter above k=10); 200 replicates; KL beta-loss

## Outputs

- Per-dataset AnnData .h5ad file (e.g. results/cptac_snrna/cptac_snrna.h5ad) containing input data plus cNMF program feature scores and usages at each retained rank
- Consensus programs and usage matrices across ranks 2–60
- postprocess verification that all factorization jobs completed

## How to apply

Initialize cNMF per dataset with mosaicmpi initialize-cnmf over the rank range k = 2–60 (e.g. --k_range 2 60 1), keeping only ranks above k = 10 that satisfy mod(k,5) = 0, and using the over-dispersed feature sets from the HVF selection step. Factorize each initialized run with mosaicmpi factorize using 200 replicates and the KL beta-loss, optionally parallelizing per-rank jobs via the repo's slurm_factorize.sh submission script. Run mosaicmpi postprocess to confirm every factorization completed, then build consensus programs and usages and perform the non-negative least-squares re-fit of the identified programs across all features of each dataset. Finally write each dataset's results (input data plus cNMF program feature scores and usages at each retained rank) to a per-dataset AnnData .h5ad file such as results/cptac_snrna/cptac_snrna.h5ad. The rationale is that deconvolving each modality independently across many ranks yields both coarse-grained (cell-type-like) and nuanced (state-like) programs that can later be integrated without re-analyzing existing datasets.

## Related tools

- **mosaicMPI** (Provides the CLI workflow (initialize-cnmf, factorize, postprocess subcommands) and SLURM scripts that execute the multi-rank cNMF sweep per dataset) — https://github.com/MorrissyLab/mosaicMPI
- **cNMF** (Consensus non-negative matrix factorization engine used to discover programs at each rank, with NNLS re-fit across all features)
- **AnnData** (Container format for the per-dataset output .h5ad files holding program feature scores and usages)
- **pandas** (Data matrix source that can be converted into AnnData/h5ad inputs for the workflow)

## Examples

```
mosaicmpi initialize-cnmf -n cptac_snrna.h5ad -o results/cptac_snrna --k_range 2 60 1
```

## Evaluation signals

- mosaicmpi postprocess confirms that every factorization job across the k = 2–60 range completed successfully
- Ranks above k = 10 in the run satisfy mod(k,5) = 0; each rank used 200 replicates with the KL beta-loss
- Each output .h5ad file contains the input data plus cNMF program feature scores and usages at every retained rank
- NNLS re-fit of programs was performed across all features of each dataset, not just the over-dispersed subset
- Per-rank factorization jobs can be traced in SLURM logs from slurm_factorize.sh submissions

## Limitations

- cNMF supports sparse data (zeros) but not missing values
- Sub-sampled or small datasets (e.g. cptac_snRNA_subsampled tutorial data) may limit the range of stable high-rank programs
- No software version information is stated for the tools in the paper; check installed versions when reproducing

## Evidence

- [readme] README multi-rank statement: [evidence span withheld — anchor status: external_doc]
