---
name: thymocyte-development-trajectory-interpretation
description: Use when an SSGATE-integrated spatial multi-omics AnnData (e.g. SCS_MT
  mouse thymus) with Leiden clusters from the integrated expression profile needs
  its developmental trajectory reconstructed (PAGA + Monocle 3) and trajectory-varying
  gene trends verified.
when_to_use_negative:
- The AnnData lacks spatial coordinates in anndata.obsm['spatial'] and its embedding
  was not produced by SSGATE integration, so cluster connectivity and trajectory direction
  would not be interpretable for the cortex-medulla comparison
- The tissue is not thymus or the analysis does not involve developmental ordering
  — the validation genes (Dntt, Rag1, Trbc1, Fcgbp, Gpx3, H2-K1) are thymocyte-stage-specific
  and provide no signal elsewhere
- Only a single trajectory method will be run — the skill's finding depends on cross-method
  consistency between PAGA and Monocle 3, which cannot be established from one method
  alone
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0121
tools:
- name: SCANPY
  role: Runs PAGA on the SSGATE-integrated embedding with Leiden cluster labels to
    build and visualize the cortex-to-medulla trajectory graph
- name: Monocle 3
  role: Infers pseudo-time (v1.0.0) on the transcriptome expression profile with the
    same cluster labels to cross-check the PAGA trajectory direction
- name: SSGATE
  role: Produces the dual-path graph attention auto-encoder integration of transcriptomics
    and proteomics from which the embedding and Leiden labels used by PAGA/Monocle
    3 are derived
  repo: github.com/Lv-BioInfo/SSGATE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:28a6ce63a5d7e3bb755223b8d1ee114ab48558f9a6557cc9e3a9831450aff765
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Monocle 3
  - SCANPY
  - SSGATE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# thymocyte-development-trajectory-interpretation

## Summary

Validates that SSGATE's integration of the SCS_MT mouse thymus spatial transcriptome-proteome data yields a cortex-to-medulla developmental trajectory by cross-checking PAGA (SCANPY) against Monocle 3 pseudo-time and inspecting marker-gene trends along pseudo-time.

## When to use

Use when an SSGATE-integrated spatial multi-omics AnnData (e.g. SCS_MT mouse thymus) with Leiden clusters from the integrated expression profile needs its developmental trajectory reconstructed (PAGA + Monocle 3) and trajectory-varying gene trends verified.

## When NOT to use

- The AnnData lacks spatial coordinates in anndata.obsm['spatial'] and its embedding was not produced by SSGATE integration, so cluster connectivity and trajectory direction would not be interpretable for the cortex-medulla comparison
- The tissue is not thymus or the analysis does not involve developmental ordering — the validation genes (Dntt, Rag1, Trbc1, Fcgbp, Gpx3, H2-K1) are thymocyte-stage-specific and provide no signal elsewhere
- Only a single trajectory method will be run — the skill's finding depends on cross-method consistency between PAGA and Monocle 3, which cannot be established from one method alone

## Inputs

- SCS_MT mouse thymus AnnData containing the SSGATE-integrated embedding (expression profiles in anndata.X, spatial coordinates in anndata.obsm['spatial'])
- Leiden cluster labels extracted from the SSGATE-integrated expression profile
- Transcriptome expression profile for Monocle 3 pseudo-time analysis (Monocle 3 v1.0.0)

## Outputs

- PAGA trajectory graph on the integrated embedding showing the outer-cortex-to-inner-medulla path
- Monocle 3 pseudo-time ordering of the same clusters for consistency comparison (Fig. 4C analogue)
- Trajectory-varying gene trend plots: Dntt/Rag1/Trbc1 decreasing and Fcgbp/Gpx3/H2-K1 increasing along pseudo-time (Fig. 4D analogue)

## How to apply

Load the SCS_MT mouse thymus AnnData holding the SSGATE-integrated embedding and the Leiden cluster labels extracted from the integrated expression profile (expression in anndata.X, spatial coordinates in anndata.obsm['spatial']). Run PAGA in SCANPY on the integrated embedding grouped by the Leiden labels to build the trajectory graph and visualize the path from the outer cortex to the inner medulla. Independently run Monocle 3 (v1.0.0) pseudo-time on the transcriptome expression profile with the same cluster labels and compare its inferred trajectory with the PAGA graph. Extract expression of Dntt, Rag1, and Trbc1 (early markers) and Fcgbp, Gpx3, and H2-K1 (late markers) along the inferred pseudo-time and plot their trends. Accept the result only if the PAGA direction (outer cortex to inner medulla) matches Monocle 3 and the early markers decrease while the late markers increase along pseudo-time, as in Fig. 4C–D of the SSGATE paper (DOI: 10.1093/bib/bbae450).

## Related tools

- **SCANPY** (Runs PAGA on the SSGATE-integrated embedding with Leiden cluster labels to build and visualize the cortex-to-medulla trajectory graph)
- **Monocle 3** (Infers pseudo-time (v1.0.0) on the transcriptome expression profile with the same cluster labels to cross-check the PAGA trajectory direction)
- **SSGATE** (Produces the dual-path graph attention auto-encoder integration of transcriptomics and proteomics from which the embedding and Leiden labels used by PAGA/Monocle 3 are derived) — github.com/Lv-BioInfo/SSGATE

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('SCS_MT_ssgate.h5ad'); sc.pp.neighbors(adata, use_rep='ssgate'); sc.tl.paga(adata, groups='leiden'); sc.pl.paga(adata)
```

## Evaluation signals

- PAGA trajectory direction on the SSGATE-integrated embedding proceeds from outer cortex to inner medulla and is consistent with the Monocle 3 result (Fig. 4C analogue)
- Dntt, Rag1, and Trbc1 expression decreases along the inferred pseudo-time
- Fcgbp, Gpx3, and H2-K1 expression increases along the inferred pseudo-time
- Input AnnData passes the SSGATE data contract: expression profiles in anndata.X and spatial coordinates in anndata.obsm['spatial']
- PAGA and Monocle 3 are run on the identical Leiden cluster labels extracted from the integrated expression profile

## Limitations

- Validation is limited to the six reported marker genes; other trajectory-varying patterns are not covered by the acceptance criteria
- Monocle 3 is run only on the transcriptome expression profile, so pseudo-time ignores proteomic information present in the spatial multi-omics data
- SSGATE hyperparameters are defaults chosen 'based on experience' rather than systematically derived, so the integrated embedding feeding the trajectory may vary with dataset-specific tuning

## Evidence

- [results] The PAGA graph derived from SSGATE's integration exhibits a developmental trajectory from outer cortex to inner medulla region, showing high consistency with Monocle 3 results.: [evidence span withheld — anchor status: not_found]
- [readme] AnnData data contract for SSGATE inputs: "The spatial coordinates from spatial multi-omics data are required to be stored in `anndata.obsm['spatial']`, otherwise it will be processed as single-cell multi-omics data"
- [readme] SSGATE facilitates downstream analyses: "SSGATE shows better performance and stronger robustness than competitive methods and facilitates downstream analysis."
- [full_text] Validation criteria from the source card: [evidence span withheld — anchor status: not_found]
- [discussion] Hyperparameter caveat from the discussion: "Based on experience, we provide a set of hyperparameters suitable for most datasets as default settings"
