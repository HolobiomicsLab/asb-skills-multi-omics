---
name: spatial-multiomics-integrated-embedding-analysis
description: Use when an AnnData object contains an SSGATE-integrated embedding and
  Leiden cluster labels for spatial multi-omics data and you need to infer and cross-validate
  a developmental trajectory with PAGA and Monocle 3 pseudo-time.
when_to_use_negative:
- Input is a single-omics dataset without an integrated multi-omics embedding — SSGATE's
  neighborhood graphs are built from transcriptome/proteome profiles or spatial coordinates
  and the trajectory workflow expects the joint representation
- Spatial coordinates are not stored in anndata.obsm['spatial'] — the README warns
  the data will then be processed as single-cell multi-omics because the spatial information
  cannot be read
- The biological question is method benchmarking (e.g., comparing SSGATE against TotalVI/Seurat
  v5/MOFA+), not developmental-trajectory validation on an already-integrated embedding
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SSGATE
  role: Produces the dual-path graph attention auto-encoder joint embedding and Leiden
    cluster labels that the trajectory analysis is run on; installed via python setup.py
    install with Python 3.9 conda environment
  repo: https://github.com/Linliu-Bioinf/SSGATE
- name: SCANPY
  role: Runs Partition-based graph abstraction (PAGA) on the SSGATE-integrated embedding
    using the Leiden cluster labels to build the trajectory graph
- name: Monocle 3
  role: Performs independent pseudo-time analysis (v1.0.0) on the transcriptome expression
    profile with the same cluster labels to cross-validate the PAGA trajectory
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:524f7e5a030b539505badc1a86be8dfa5e9e3d8980b3f09840cde26d76eb0d7e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Linliu-Bioinf/SSGATE
  tools:
  - Monocle 3
  - SCANPY
  - SSGATE
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# spatial-multiomics-integrated-embedding-analysis

## Summary

Downstream developmental-trajectory analysis (PAGA graph abstraction plus Monocle 3 pseudo-time) on the SSGATE-integrated embedding of mouse thymus spatial transcriptome/proteome data, validating that both methods recover a consistent outer-cortex-to-inner-medulla trajectory and the expected spatially patterned gene trends.

## When to use

Use when an AnnData object contains an SSGATE-integrated embedding and Leiden cluster labels for spatial multi-omics data and you need to infer and cross-validate a developmental trajectory with PAGA and Monocle 3 pseudo-time.

## When NOT to use

- Input is a single-omics dataset without an integrated multi-omics embedding — SSGATE's neighborhood graphs are built from transcriptome/proteome profiles or spatial coordinates and the trajectory workflow expects the joint representation
- Spatial coordinates are not stored in anndata.obsm['spatial'] — the README warns the data will then be processed as single-cell multi-omics because the spatial information cannot be read
- The biological question is method benchmarking (e.g., comparing SSGATE against TotalVI/Seurat v5/MOFA+), not developmental-trajectory validation on an already-integrated embedding

## Inputs

- AnnData (h5ad) object with the SSGATE-integrated embedding and Leiden cluster labels in obs (SCS_MT mouse thymus dataset)
- Transcriptome expression profile stored in anndata.X (required for Monocle 3 pseudo-time)
- Spatial coordinates stored in anndata.obsm['spatial'] (required by SSGATE to build the spatial neighborhood graph)

## Outputs

- PAGA trajectory graph computed on the integrated embedding showing the outer-cortex-to-inner-medulla path
- Monocle 3 pseudo-time values per cell/spatial spot with an inferred trajectory comparable to the PAGA graph
- Trend plots of Dntt/Rag1/Trbc1 (decreasing) and Fcgbp/Gpx3/H2-K1 (increasing) along pseudo-time

## How to apply

Load the SCS_MT mouse thymus AnnData holding the SSGATE-integrated joint embedding (30-dimensional output from the dual-path graph attention auto-encoder) and the Leiden cluster labels extracted from the integrated expression profile. Run Partition-based graph abstraction (PAGA, via SCANPY) on the integrated embedding with the cluster labels to build the trajectory graph and trace the path from outer cortex to inner medulla. Independently run Monocle 3 (v1.0.0) pseudo-time on the transcriptome expression profile with the same cluster labels and compare its inferred trajectory with the PAGA graph for directional consistency. Finally, extract Dntt, Rag1, and Trbc1 (cortex-associated) and Fcgbp, Gpx3, and H2-K1 (medulla-associated) expression along pseudo-time and plot trends to confirm the reported early-decrease and late-increase patterns.

## Related tools

- **SSGATE** (Produces the dual-path graph attention auto-encoder joint embedding and Leiden cluster labels that the trajectory analysis is run on; installed via python setup.py install with Python 3.9 conda environment) — https://github.com/Linliu-Bioinf/SSGATE
- **SCANPY** (Runs Partition-based graph abstraction (PAGA) on the SSGATE-integrated embedding using the Leiden cluster labels to build the trajectory graph)
- **Monocle 3** (Performs independent pseudo-time analysis (v1.0.0) on the transcriptome expression profile with the same cluster labels to cross-validate the PAGA trajectory)

## Examples

```
ssgate.train(adata_st, adata_sp, hidden_dims1=128, hidden_dims2=128, out_dims=30, cluster_update_epoch=20, epochs_init=50, n_epochs=300, lr=0.001, sigma=0.1)
```

## Evaluation signals

- PAGA trajectory direction (outer cortex to inner medulla) is consistent with the Monocle 3 result, as in Fig. 4C of the paper
- Dntt, Rag1, and Trbc1 expression decreases along the inferred pseudo-time, as in Fig. 4D
- Fcgbp, Gpx3, and H2-K1 expression increases along the inferred pseudo-time, as in Fig. 4D
- Input AnnData passes the SSGATE format contract: expression profiles in anndata.X and spatial coordinates in anndata.obsm['spatial']
- The integrated embedding has the expected dimensionality (out_dims = 30, the SSGATE default output layer size)

## Limitations

- SSGATE hyperparameters are set 'based on experience' as defaults rather than systematically derived (e.g., hidden_dims 128/128, out_dims 30, n_epochs 300, lr 0.001, sigma 0.1)
- No software versions are reported for the compared methods in the paper, so reproducibility of the integration step relies on the provided repository and data links
- Normalization and dimensionality-reduction methods for integrating omics beyond transcriptome/proteome are not yet available in SSGATE

## Evidence

- [other] Load the SCS_MT mouse thymus AnnData containing the SSGATE-integrated embedding and the Leiden cluster labels extracted from the integrated expression profile.: [evidence span withheld — anchor status: not_found]
- [results] The PAGA graph derived from SSGATE's integration exhibits a developmental trajectory from outer cortex to inner medulla region, showing high consistency with Monocle 3 results.: [evidence span withheld — anchor status: not_found]
- [readme] SSGATE requires data in anndata format as inputs.: "SSGATE requires data in [anndata](https://anndata.readthedocs.io/en/latest/) format as inputs."
- [readme] Spatial coordinates must be in obsm['spatial']: "The spatial coordinates from spatial multi-omics data are required to be stored in 
`anndata.obsm['spatial']`"
- [abstract] SSGATE shows better performance and stronger robustness than competitive methods and facilitates downstream analysis.: "SSGATE shows better performance and stronger robustness than competitive methods and facilitates downstream analysis."
- [discussion] Hyperparameters are defaults based on experience: "Based on experience, we provide a set of hyperparameters suitable for most datasets as default settings"
