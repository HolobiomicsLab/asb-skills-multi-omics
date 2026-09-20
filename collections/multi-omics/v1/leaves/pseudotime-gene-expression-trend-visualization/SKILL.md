---
name: pseudotime-gene-expression-trend-visualization
description: Use when an SSGATE-integrated spatial AnnData with a Leiden clustering
  exists and the goal is to verify a spatial developmental trajectory (e.g., outer
  cortex to inner medulla) and its gene expression dynamics.
when_to_use_negative:
- Input AnnData lacks an SSGATE integration embedding or spatial coordinates in obsm['spatial']
  — the trajectory should be built on the integrated representation, not raw counts
- The biological system has no known spatial axis or Leiden clusters to map the trajectory
  onto (PAGA requires precomputed cluster labels)
- Gene-set trend verification is needed for omics types other than transcriptome/proteome
  — the article notes normalization and dimensionality reduction for other omics is
  not yet available
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCANPY
  role: Runs PAGA (partition-based graph abstraction) on the SSGATE-integrated embedding
    with Leiden cluster labels and visualizes the trajectory
- name: Monocle 3
  role: Performs pseudo-time analysis (v1.0.0) on the transcriptome expression profile
    to independently validate the PAGA trajectory direction
- name: SSGATE
  role: Produces the integrated embedding and the Leiden labels from which the trajectory
    is derived; installs via python setup.py install
  repo: https://github.com/Linliu-Bioinf/SSGATE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:c8ab39c302e403c6b32f7583062e68e4f0f79d9e89a1808f3b6a134ded5df7b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Linliu-Bioinf/SSGATE
  tools:
  - Monocle 3
  - SCANPY
  - SSGATE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# pseudotime-gene-expression-trend-visualization

## Summary

Builds a developmental trajectory on SSGATE-integrated spatial multi-omics data (SCS_MT mouse thymus) by running PAGA on the integrated embedding with Leiden cluster labels, cross-validating direction with Monocle 3 pseudo-time, and plotting gene expression trends (Dntt/Rag1/Trbc1 decreasing; Fcgbp/Gpx3/H2-K1 increasing) along pseudo-time.

## When to use

Use when an SSGATE-integrated spatial AnnData with a Leiden clustering exists and the goal is to verify a spatial developmental trajectory (e.g., outer cortex to inner medulla) and its gene expression dynamics.

## When NOT to use

- Input AnnData lacks an SSGATE integration embedding or spatial coordinates in obsm['spatial'] — the trajectory should be built on the integrated representation, not raw counts
- The biological system has no known spatial axis or Leiden clusters to map the trajectory onto (PAGA requires precomputed cluster labels)
- Gene-set trend verification is needed for omics types other than transcriptome/proteome — the article notes normalization and dimensionality reduction for other omics is not yet available

## Inputs

- SCS_MT mouse thymus AnnData (.h5ad) with SSGATE-integrated embedding in anndata.X
- Leiden cluster labels extracted from the SSGATE-integrated expression profile
- Transcriptome expression profile (for Monocle 3 pseudo-time)

## Outputs

- PAGA trajectory graph and visualization of the outer-cortex-to-inner-medulla path
- Monocle 3 pseudo-time trajectory for the same clusters
- Pseudo-time trend plots for Dntt, Rag1, Trbc1 (decreasing) and Fcgbp, Gpx3, H2-K1 (increasing)

## How to apply

Load the SCS_MT mouse thymus AnnData holding the SSGATE-integrated embedding (stored in .X per AnnData convention; spatial coordinates in adata.obsm['spatial']) and Leiden labels extracted from the integrated expression profile. Run Partition-based graph abstraction (PAGA, via SCANPY) on the integrated embedding using the cluster labels to build the trajectory graph and visualize the outer-cortex-to-inner-medulla path. Independently run Monocle 3 v1.0.0 pseudo-time on the transcriptome expression profile with the same cluster labels and compare its inferred trajectory with the PAGA graph for directional consistency. Extract expression of Dntt, Rag1, and Trbc1 and of Fcgbp, Gpx3, and H2-K1 along the pseudo-time axis and plot their trends. Accept the result only if PAGA and Monocle 3 agree on direction and the gene trends reproduce the reported early-decrease/late-increase patterns (Fig. 4C/4D).

## Related tools

- **SCANPY** (Runs PAGA (partition-based graph abstraction) on the SSGATE-integrated embedding with Leiden cluster labels and visualizes the trajectory)
- **Monocle 3** (Performs pseudo-time analysis (v1.0.0) on the transcriptome expression profile to independently validate the PAGA trajectory direction)
- **SSGATE** (Produces the integrated embedding and the Leiden labels from which the trajectory is derived; installs via python setup.py install) — https://github.com/Linliu-Bioinf/SSGATE

## Examples

```
import scanpy as sc; sc.tl.paga(adata, groups='leiden'); sc.pl.paga(adata, layout='fr')
```

## Evaluation signals

- PAGA trajectory direction (outer cortex to inner medulla) is consistent with the Monocle 3 result, as in Fig. 4C
- Dntt, Rag1, and Trbc1 expression decreases along the inferred pseudo-time, matching Fig. 4D
- Fcgbp, Gpx3, and H2-K1 expression increases along the inferred pseudo-time, matching Fig. 4D
- The AnnData read correctly: expression in anndata.X and spatial coordinates in adata.obsm['spatial'], otherwise SSGATE treats the data as single-cell and spatial information is lost

## Limitations

- Hyperparameters are set 'based on experience' as defaults rather than systematically derived, which may affect the integration embedding feeding the trajectory
- The validation depends on the reported Fig. 4C/4D patterns; no software versions are reported for compared methods, so reproducibility of the PAGA/Monocle 3 comparison may vary
- The SCS_MT datasets used in the study are distributed via a BGI cloud disk link with an extract password, not a programmatic accession

## Evidence

- [results] PAGA graph derived from SSGATE's integration exhibits a developmental trajectory from outer cortex to inner medulla region, showing high consistency with Monocle 3 results.: [evidence span withheld — anchor status: not_found]
- [readme] AnnData input requirements for SSGATE: "The expression profiles are required to be stored in anndata.X, otherwise the data cannot be read correctly."
- [readme] Spatial coordinates storage requirement: "The spatial coordinates from spatial multi-omics data are required to be stored in `anndata.obsm['spatial']`, otherwise it will be processed as single-cell multi-omics data"
- [abstract] SSGATE integration enables downstream analysis: "SSGATE shows better performance and stronger robustness than competitive methods and facilitates downstream analysis."
