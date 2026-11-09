---
name: monocle3-pseudotime-inference
description: Use when an SSGATE (or similar) multi-omics integration result in AnnData
  form, with cluster labels and an integrated embedding, must be validated by ordering
  cells along pseudo-time and comparing against a PAGA trajectory graph.
when_to_use_negative:
- The AnnData does not carry cluster labels or an integrated embedding from SSGATE,
  so no shared labels exist to anchor the PAGA-vs-Monocle 3 comparison
- The analysis goal is de novo trajectory discovery rather than validating an already-integrated
  spatial multi-omics embedding against PAGA
- Expression data are not stored in anndata.X as required by the SSGATE workflow,
  so the AnnData cannot be read correctly downstream
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Monocle 3
  role: Pseudo-time (trajectory) inference on the transcriptome expression profile
    using the same Leiden cluster labels used for PAGA (version v1.0.0)
- name: SCANPY
  role: Runs PAGA on the SSGATE-integrated embedding with the cluster labels to build
    the reference trajectory graph from outer cortex to inner medulla
- name: SSGATE
  role: Provides the integrated embedding and cluster labels of the mouse thymus spatial
    multi-omics data that the pseudo-time analysis validates
  repo: https://github.com/Lv-BioInfo/SSGATE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:8e3a345d584e4eccc2b8c06869e08c1ac341fd63eb639f8f03a24001c113462d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lv-BioInfo/SSGATE
  tools:
  - Monocle 3
  - SCANPY
  - SSGATE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# monocle3-pseudotime-inference

## Summary

Runs Monocle 3 (v1.0.0) pseudo-time analysis on the transcriptome expression profile of the SSGATE-integrated mouse thymus spatial multi-omics data (SCS_MT) to validate that the developmental trajectory from outer cortex to inner medulla is consistent between PAGA and Monocle 3, and verifies trajectory-varying gene trends for Dntt/Rag1/Trbc1 (early-decrease) and Fcgbp/Gpx3/H2-K1 (late-increase).

## When to use

Use when an SSGATE (or similar) multi-omics integration result in AnnData form, with cluster labels and an integrated embedding, must be validated by ordering cells along pseudo-time and comparing against a PAGA trajectory graph.

## When NOT to use

- The AnnData does not carry cluster labels or an integrated embedding from SSGATE, so no shared labels exist to anchor the PAGA-vs-Monocle 3 comparison
- The analysis goal is de novo trajectory discovery rather than validating an already-integrated spatial multi-omics embedding against PAGA
- Expression data are not stored in anndata.X as required by the SSGATE workflow, so the AnnData cannot be read correctly downstream

## Inputs

- SCS_MT mouse thymus AnnData (.h5ad) with the SSGATE-integrated embedding
- Leiden cluster labels extracted from the SSGATE-integrated expression profile
- Transcriptome expression profile of the same cells (for Monocle 3 pseudo-time)

## Outputs

- Monocle 3 pseudo-time value per cell over the SCS_MT transcriptome profile
- Trajectory consistency comparison between the PAGA graph and the Monocle 3 pseudo-time (Fig. 4C-style result)
- Gene trend plots of Dntt/Rag1/Trbc1 and Fcgbp/Gpx3/H2-K1 along pseudo-time (Fig. 4D-style result)

## How to apply

Load the SCS_MT mouse thymus AnnData containing the SSGATE-integrated embedding and the Leiden cluster labels extracted from the integrated expression profile. First run Partition-based graph abstraction (PAGA, via SCANPY) on the integrated embedding with the cluster labels to build the reference trajectory graph from outer cortex to inner medulla. Then run Monocle 3 (v1.0.0) pseudo-time analysis on the transcriptome expression profile using the same cluster labels, and compare its inferred trajectory with the PAGA graph for directional consistency. Extract the expression of Dntt, Rag1, and Trbc1 and of Fcgbp, Gpx3, and H2-K1 along the inferred pseudo-time and plot their trends to verify the reported early-decrease and late-increase patterns. Accept the output only if the PAGA trajectory direction (outer cortex to inner medulla) matches the Monocle 3 result (as in Fig. 4C) and the two gene sets show the expected decreasing/increasing trends along pseudo-time (as in Fig. 4D).

## Related tools

- **Monocle 3** (Pseudo-time (trajectory) inference on the transcriptome expression profile using the same Leiden cluster labels used for PAGA (version v1.0.0))
- **SCANPY** (Runs PAGA on the SSGATE-integrated embedding with the cluster labels to build the reference trajectory graph from outer cortex to inner medulla)
- **SSGATE** (Provides the integrated embedding and cluster labels of the mouse thymus spatial multi-omics data that the pseudo-time analysis validates) — https://github.com/Lv-BioInfo/SSGATE

## Examples

```
cds <- new_cell_data_set(expr_matrix, cell_metadata=cell_meta, gene_metadata=gene_meta); cds <- preprocess_cds(cds); cds <- learn_graph(cds); cds <- order_cells(cds); plot_cells(cds, color_cells_by="pseudotime")
```

## Evaluation signals

- The PAGA trajectory direction (outer cortex to inner medulla) is consistent with the Monocle 3 pseudo-time result, as in Fig. 4C
- Dntt, Rag1, and Trbc1 show decreasing expression along the inferred pseudo-time (early-decrease pattern)
- Fcgbp, Gpx3, and H2-K1 show increasing expression along the inferred pseudo-time (late-increase pattern), as in Fig. 4D
- The same Leiden cluster labels are used for both PAGA and Monocle 3 so the two trajectories are directly comparable
- Gene trend plots reproduce the reported outer-cortex-to-inner-medulla developmental ordering on SCS_MT

## Limitations

- SSGATE hyperparameters are set 'based on experience' as defaults rather than systematically derived, so the integrated embedding feeding the trajectory analysis may be sensitive to parameter choices
- No software versions are reported for compared methods in the article's discussion section, which limits exact reproducibility beyond the Monocle 3 v1.0.0 version noted in the workflow
- The validation is demonstrated on the mouse thymus spatial multi-omics case (SCS_MT); applicability to other tissues or omics types is not established

## Evidence

- [full_text] Run Monocle 3 (v1.0.0) pseudo-time analysis on the transcriptome expression profile with the same cluster labels and compare its inferred trajectory with the PAGA graph for consistency.: [evidence span withheld — anchor status: not_found]
- [full_text] Extract the expression of Dntt, Rag1, and Trbc1 and of Fcgbp, Gpx3, and H2-K1 along the inferred pseudo-time and plot their trends to verify the reported early-decrease and late-increase patterns.: [evidence span withheld — anchor status: not_found]
- [readme] The expression profiles are required to be stored in anndata.X, otherwise the data cannot be read correctly: "The expression profiles are required to be stored in anndata.X, otherwise the data cannot be read correctly"
- [abstract] SSGATE shows better performance and stronger robustness than competitive methods and facilitates downstream analysis.: "SSGATE shows better performance and stronger robustness than competitive methods and facilitates downstream analysis."
