---
name: gaussian-noise-injection-for-robustness-benchmarking
description: Use when evaluating how a multi-omics integration method degrades on
  intentionally corrupted inputs — e.g. comparing SSGATE vs baselines on noisy BMNC/SLN111_D1
  anndata pairs under a paper's robustness protocol.
when_to_use_negative:
- Input data is spatial multi-omics with coordinates in anndata.obsm['spatial'] —
  the noise-robustness protocol here targets single-cell paired transcriptome-proteome
  data
- The goal is evaluating raw integration performance on clean data rather than robustness
  to degraded data quality
- Only one omics modality is present — the protocol requires paired transcriptome-proteome
  inputs
edam_operation: http://edamontology.org/operation_0321
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SSGATE
  role: Method under test; runs on each noisy dataset with default hyperparameters
    to produce 30-dimensional joint embeddings
  repo: github.com/Lv-BioInfo/SSGATE
- name: MOFA+
  role: Baseline integration method run on identical noisy inputs for ranking comparison
- name: Seurat v5
  role: Network-based baseline that constructs a weighted nearest neighbor graph on
    the noisy inputs
- name: TotalVI
  role: Probabilistic baseline learning a joint representation of paired gene expression
    and protein data on noisy inputs
- name: ScMM
  role: Mixture-of-experts multimodal VAE baseline run on the same noisy inputs
- name: SCANPY + Leiden algorithm
  role: Clustering of each integration across resolutions 0.1-1.0 to generate the
    10 replicates used for metric computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:52e560f5f6f188d01103c05558c8e0143efacda435a7f14c78a63c4b9c7872c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lv-BioInfo/SSGATE
  tools:
  - MOFA
  - MOFA+
  - SCANPY
  - SCANPY + Leiden algorithm
  - SSGATE
  - ScMM
  - Seurat v5
  - TotalVI
  edam_operation: http://edamontology.org/operation_0321
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# Gaussian noise injection for robustness benchmarking

## Summary

Injects three levels of Gaussian noise into paired transcriptome-proteome single-cell h5ad datasets (BMNC, SLN111_D1) to benchmark the robustness of SSGATE against MOFA+, Seurat v5, totalVI, and scMM, yielding averaged ranking statistics.

## When to use

Use when evaluating how a multi-omics integration method degrades on intentionally corrupted inputs — e.g. comparing SSGATE vs baselines on noisy BMNC/SLN111_D1 anndata pairs under a paper's robustness protocol.

## When NOT to use

- Input data is spatial multi-omics with coordinates in anndata.obsm['spatial'] — the noise-robustness protocol here targets single-cell paired transcriptome-proteome data
- The goal is evaluating raw integration performance on clean data rather than robustness to degraded data quality
- Only one omics modality is present — the protocol requires paired transcriptome-proteome inputs

## Inputs

- Paired transcriptome-proteome anndata (h5ad) files: BMNC and SLN111_D1 from the SSGATE repository (BGI cloud disk, extract password nrEB)
- Ground-truth cell-type labels for each dataset
- Noise specification: three levels of Gaussian noise per dataset

## Outputs

- noise_robustness_results.csv with Purity, Homogeneity Score, ARI, and NMI per method, noise level, and replicate
- Comprehensive average rankings per method (expected: scMM 4.88, totalVI 3.63, Seurat v5 2.38, MOFA+ 2.25, SSGATE 1.88)
- Robustness comparison reproducing Fig. 3 (SSGATE best ARI and NMI on both noisy datasets)

## How to apply

Load the paired transcriptome-proteome BMNC and SLN111_D1 h5ad datasets from the SSGATE repository's BGI cloud disk into anndata objects, with expression in anndata.X. Inject three different levels of Gaussian noise into each dataset to simulate degraded data quality, as described in the paper's robustness protocol. Run SSGATE on each noisy dataset with its default hyperparameters (number of neighbors 15, balance parameter 0.1, embedding dimension 30, maximum 300 epochs, learning rate 0.001) and run MOFA+, Seurat v5, totalVI, and scMM on the same noisy inputs. Cluster each integration with the Leiden algorithm in SCANPY across resolutions 0.1, 0.2, ..., 1.0 to obtain 10 independent replicates per method. Compute Purity, Homogeneity Score, Adjusted Rand Index, and Normalized Mutual Information against ground-truth labels for every method, noise level, and replicate, then average across the three noise levels to derive comprehensive average rankings. Correct application reproduces rankings scMM 4.88, totalVI 3.63, Seurat v5 2.38, MOFA+ 2.25, SSGATE 1.88 (lower is better), with SSGATE achieving the best ARI and NMI on both noisy datasets as in Fig. 3.

## Related tools

- **SSGATE** (Method under test; runs on each noisy dataset with default hyperparameters to produce 30-dimensional joint embeddings) — github.com/Lv-BioInfo/SSGATE
- **MOFA+** (Baseline integration method run on identical noisy inputs for ranking comparison)
- **Seurat v5** (Network-based baseline that constructs a weighted nearest neighbor graph on the noisy inputs)
- **TotalVI** (Probabilistic baseline learning a joint representation of paired gene expression and protein data on noisy inputs)
- **ScMM** (Mixture-of-experts multimodal VAE baseline run on the same noisy inputs)
- **SCANPY + Leiden algorithm** (Clustering of each integration across resolutions 0.1-1.0 to generate the 10 replicates used for metric computation)

## Examples

```
adata_st, adata_sp = ssgate.train(adata_st, adata_sp, out_dims=30, n_epochs=300, lr=0.001)
```

## Evaluation signals

- Comprehensive average rankings match reported values: scMM 4.88, totalVI 3.63, Seurat v5 2.38, MOFA+ 2.25, SSGATE 1.88
- SSGATE attains the highest (best) average ranking and the best ARI and NMI on both BMNC and SLN111_D1 noisy datasets, matching Fig. 3
- Metrics (Purity, Homogeneity Score, ARI, NMI) are computed for every combination of method, noise level, and replicate (10 Leiden resolutions per method) before averaging across noise levels
- All results are written to noise_robustness_results.csv with the expected per-method/per-replicate schema

## Limitations

- SSGATE hyperparameters are set 'based on experience' as defaults rather than systematically derived, so robustness results may be sensitive to the chosen defaults
- No software versions are reported for the compared methods (e.g. scMM, totalVI, MOFA+), complicating exact reproduction of the ranking table
- The protocol requires expression in anndata.X; data stored elsewhere cannot be read correctly by SSGATE

## Evidence

- [abstract] SSGATE shows better performance and stronger robustness than competitive methods and facilitates downstream analysis: "SSGATE shows better performance and stronger robustness than competitive methods and facilitates downstream analysis."
- [readme] SSGATE requires data in anndata format as inputs: "SSGATE requires data in [anndata](https://anndata.readthedocs.io/en/latest/) format as inputs."
- [readme] Datasets used in the study are hosted on BGI cloud disk: "We provide the datasets used in this study, which are stored in the BGI cloud disk."
- [intro] TotalVI models paired gene expression and protein data: "TotalVI utilizes a modeling strategy similar to scVI [20] to learn a joint probabilistic representation of paired gene expression and protein data"
