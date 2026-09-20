---
name: paga-graph-abstraction-trajectory-inference
description: Use when an SSGATE-integrated spatial AnnData (e.g. SCS_MT mouse thymus
  with Leiden labels) must be ordered into an anatomical trajectory (outer cortex
  → inner medulla) and checked against Monocle 3 pseudo-time.
when_to_use_negative:
- The AnnData has no SSGATE-integrated embedding or Leiden labels (e.g. raw counts
  only) — this skill's PAGA step is defined on the integrated joint representation,
  not raw expression.
- The tissue lacks a known anatomical polarity (like thymus outer cortex → inner medulla)
  against which the trajectory direction can be validated.
- The goal is benchmarking SSGATE's integration performance against methods like MOFA+/TotalVI/Seurat
  v5 rather than performing downstream developmental trajectory analysis.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: SCANPY
  role: Implements PAGA (Partition-based graph abstraction) to build and visualize
    the trajectory graph on the SSGATE-integrated embedding using the Leiden cluster
    labels.
- name: Monocle 3
  role: v1.0.0 pseudo-time inference on the transcriptome expression profile with
    the same cluster labels, used to cross-check the PAGA trajectory direction for
    consistency.
- name: SSGATE
  role: Upstream producer of the integrated joint feature representation and Leiden
    cluster labels from the SCS_MT spatial transcriptome-proteome data on which PAGA
    and Monocle 3 are run.
  repo: https://github.com/Linliu-Bioinf/SSGATE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:572694e16df834d56e057ea7f5715ac1ccfde7e5f2976b030adf72ee4d31fda8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Linliu-Bioinf/SSGATE
  tools:
  - Monocle 3
  - PAGA
  - SCANPY
  - SSGATE
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# paga-graph-abstraction-trajectory-inference

## Summary

Orders an SSGATE-integrated mouse thymus (SCS_MT) spatial multi-omics embedding into a developmental trajectory using Partition-based graph abstraction (PAGA) in SCANPY, cross-validated against Monocle 3 pseudo-time and marker-gene trends (Dntt/Rag1/Trbc1 decreasing, Fcgbp/Gpx3/H2-K1 increasing). This reproduces the paper's Fig. 4C/4D result that SSGATE integration facilitates downstream trajectory analysis.

## When to use

Use when an SSGATE-integrated spatial AnnData (e.g. SCS_MT mouse thymus with Leiden labels) must be ordered into an anatomical trajectory (outer cortex → inner medulla) and checked against Monocle 3 pseudo-time.

## When NOT to use

- The AnnData has no SSGATE-integrated embedding or Leiden labels (e.g. raw counts only) — this skill's PAGA step is defined on the integrated joint representation, not raw expression.
- The tissue lacks a known anatomical polarity (like thymus outer cortex → inner medulla) against which the trajectory direction can be validated.
- The goal is benchmarking SSGATE's integration performance against methods like MOFA+/TotalVI/Seurat v5 rather than performing downstream developmental trajectory analysis.

## Inputs

- SCS_MT mouse thymus AnnData (.h5ad) containing the SSGATE-integrated joint embedding (default out_dims=30) and expression in anndata.X
- Leiden cluster labels extracted from the SSGATE-integrated expression profile
- Transcriptome expression profile (input to Monocle 3 v1.0.0 pseudo-time)
- Marker gene set: Dntt, Rag1, Trbc1 (early/cortical) and Fcgbp, Gpx3, H2-K1 (late/medullary)

## Outputs

- PAGA trajectory graph over Leiden clusters plotted on the SSGATE-integrated embedding (outer cortex → inner medulla path)
- Monocle 3 pseudo-time values per cell/spot computed on the transcriptome profile
- Pseudo-time trend plots for Dntt, Rag1, Trbc1 (decreasing) and Fcgbp, Gpx3, H2-K1 (increasing)
- Consistency assessment of PAGA vs. Monocle 3 trajectories (Fig. 4C-style) and gene-trend verification (Fig. 4D-style)

## How to apply

Load the SCS_MT mouse thymus AnnData carrying the SSGATE-integrated joint feature representation (SSGATE default out_dims=30) and the Leiden cluster labels extracted from the integrated expression profile, with expression stored in anndata.X per the SSGATE README. Run Partition-based graph abstraction (PAGA, via SCANPY) on that integrated embedding using the Leiden labels to build the trajectory graph and visualize the path from the outer cortex to the inner medulla region. Independently run Monocle 3 (v1.0.0) pseudo-time on the transcriptome expression profile with the same cluster labels and compare its inferred trajectory with the PAGA graph for consistency (Fig. 4C). Extract the expression of Dntt, Rag1, and Trbc1 (expected early decrease) and of Fcgbp, Gpx3, and H2-K1 (expected late increase) along the inferred pseudo-time and plot their trends (Fig. 4D). Decision rule: accept the reproduction only if the PAGA trajectory direction matches the Monocle 3 result and the six genes show the reported decreasing/increasing patterns — agreement between embedding-based graph abstraction and expression-based pseudo-time indicates the integration preserved developmental biology.

## Related tools

- **SCANPY** (Implements PAGA (Partition-based graph abstraction) to build and visualize the trajectory graph on the SSGATE-integrated embedding using the Leiden cluster labels.)
- **Monocle 3** (v1.0.0 pseudo-time inference on the transcriptome expression profile with the same cluster labels, used to cross-check the PAGA trajectory direction for consistency.)
- **SSGATE** (Upstream producer of the integrated joint feature representation and Leiden cluster labels from the SCS_MT spatial transcriptome-proteome data on which PAGA and Monocle 3 are run.) — https://github.com/Linliu-Bioinf/SSGATE

## Examples

```
import scanpy as sc; sc.tl.paga(adata_scs_mt, groups='leiden'); sc.pl.paga(adata_scs_mt)
```

## Evaluation signals

- PAGA graph on the SSGATE-integrated embedding traces the developmental path from outer cortex to inner medulla, matching the Fig. 4C layout.
- Monocle 3 (v1.0.0) pseudo-time run on the transcriptome profile with the identical Leiden labels reproduces the same trajectory ordering as the PAGA graph (high consistency, Fig. 4C).
- Dntt, Rag1, and Trbc1 expression decreases along pseudo-time (early-decrease pattern, Fig. 4D).
- Fcgbp, Gpx3, and H2-K1 expression increases along pseudo-time (late-increase pattern, Fig. 4D).
- The cluster label set passed to PAGA and Monocle 3 is identical and traceable to the SSGATE-integrated expression profile; mismatched labels invalidate the comparison.

## Limitations

- Trajectory direction is anchored to known thymus anatomy (outer cortex → inner medulla); without such anatomical polarity, PAGA branch orientation is ambiguous.
- The PAGA input embedding comes from SSGATE hyperparameters set 'based on experience' as defaults rather than systematically derived, so trajectory reproduction depends on those defaults.
- Demonstrated on transcriptome-proteome integration (SCS_MT mouse thymus); SSGATE's normalization and dimensionality reduction for other omics types are deferred to future work.
- Source datasets are provided via a BGI cloud-disk link with an extract password, so long-term data availability for reproduction is not guaranteed.

## Evidence

- [other] The PAGA graph derived from SSGATE's integration exhibits a developmental trajectory from outer cortex to inner medulla region, showing high consistency with Monocle 3 results.: [evidence span withheld — anchor status: not_found]
- [other] Run Partition-based graph abstraction (PAGA) on the integrated embedding with the cluster labels to build the trajectory graph: [evidence span withheld — anchor status: not_found]
- [other] Run Monocle 3 (v1.0.0) pseudo-time analysis on the transcriptome expression profile with the same cluster labels and compare its inferred trajectory with the PAGA graph for consistency.: [evidence span withheld — anchor status: not_found]
- [other] Extract the expression of Dntt, Rag1, and Trbc1 and of Fcgbp, Gpx3, and H2-K1 along the inferred pseudo-time and plot their trends: [evidence span withheld — anchor status: not_found]
- [readme] After multiple rounds of training, the joint feature representation of the two omics can be obtained.: "After multiple rounds of training, the joint feature representation of the two omics can be obtained."
- [readme] The expression profiles are required to be stored in anndata.X, otherwise the data cannot be read correctly: "The expression profiles are required to be stored in anndata.X, otherwise the data cannot be read correctly"
- [discussion] Based on experience, we provide a set of hyperparameters suitable for most datasets as default settings: "Based on experience, we provide a set of hyperparameters suitable for most datasets as default settings"
