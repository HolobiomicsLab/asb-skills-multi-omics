---
name: benjamini-hochberg-fdr-correction
description: Use when you have per-feature or per-group p-values (e.g., from VI-VS
  CRT with K Monte Carlo replicates) and need to call discoveries at target FDR levels
  such as alpha=0.1 while controlling the realized false discovery rate.
when_to_use_negative:
- When p-values come from procedures that are not calibrated (e.g., OLS-based or marginal
  association p-values under misspecified noise models), BH applied on top of them
  will not guarantee FDR control, as shown by the paper's OLS failure under Poisson-generated
  responses.
- When the question is marginal co-expression or association rather than conditional
  dependency — BH cannot fix spurious associations inflated by batch effects, sequencing
  depth, or cell cycle.
- When hypotheses are not numerous or independent enough for BH's assumptions and
  a per-hypothesis decision without FDR framing is needed.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: VI-VS
  role: Produces the calibrated CRT p-values that are corrected with Benjamini-Hochberg
    before calling discoveries at target FDR levels
  repo: github.com/YosefLab/VIVS
- name: VIVS
  role: Open-source Python implementation whose model.get_hier_importance outputs
    p-values that are BH-corrected for multi-resolution testing
  repo: github.com/YosefLab/VIVS
- name: GENIE3
  role: Supplies prediction-error importance scores whose VI-VS-wrapped p-values are
    BH-corrected and evaluated for FDR/TPR
- name: Jax
  role: Backend used to run the Monte Carlo replicates of the CRT whose p-values feed
    into BH correction
- name: scanpy
  role: AnnData container for the input gene expression counts from which p-values
    are computed
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_006
  - task_002
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:12e800cfaf9f49fd2c36ae985823ec3de57357ebaa98481a0e72f96c200246b2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GENIE3
  - Jax
  - PyTorch
  - Python
  - VI-VS
  - VIVS
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# benjamini-hochberg-fdr-correction

## Summary

Applies the Benjamini-Hochberg procedure to VI-VS conditional randomization test (CRT) p-values so that feature/group discoveries in single-cell data are called at controlled target false discovery rates. It is the calibration step that turns per-feature p-values from the CRT into decision rules with a theoretical FDR guarantee.

## When to use

Use when you have per-feature or per-group p-values (e.g., from VI-VS CRT with K Monte Carlo replicates) and need to call discoveries at target FDR levels such as alpha=0.1 while controlling the realized false discovery rate.

## When NOT to use

- When p-values come from procedures that are not calibrated (e.g., OLS-based or marginal association p-values under misspecified noise models), BH applied on top of them will not guarantee FDR control, as shown by the paper's OLS failure under Poisson-generated responses.
- When the question is marginal co-expression or association rather than conditional dependency — BH cannot fix spurious associations inflated by batch effects, sequencing depth, or cell cycle.
- When hypotheses are not numerous or independent enough for BH's assumptions and a per-hypothesis decision without FDR framing is needed.

## Inputs

- Per-feature or per-group CRT p-values from VI-VS (e.g., output of model.get_hier_importance)
- Target FDR levels (e.g., alpha=0.1, or a grid of target levels)
- Optional ground-truth positive labels (e.g., two-sample t-tests of perturbed vs unperturbed Perturb-seq cells) for computing empirical FDR/TPR

## Outputs

- Sets of discovered features/gene groups at each target FDR level
- Empirical FDR and TPR (or power) tables per target level, e.g., genie3_fdr_tpr.csv, fdr_power_multiresolution.csv, perturbational_screen_fdr_tpr.csv

## How to apply

First obtain p-values for each feature (gene) or feature group from the VI-VS conditional randomization test, run with K Monte Carlo randomized replicates using the VIVS package (Jax backend). Then apply the Benjamini-Hochberg correction across all tested hypotheses at each target FDR level (the paper evaluates several target levels, e.g., alpha=0.1 in the Perturb-seq benchmark). Call discoveries as the BH-rejected hypotheses, then compute empirical FDR and true positive rate (or power) against ground truth, such as t-tests of perturbed vs unperturbed cells, and record results per level (e.g., genie3_fdr_tpr.csv, perturbational_screen_fdr_tpr.csv). Validation: because VI-VS p-values stochastically dominate the uniform distribution under the null, the realized FDR at each target level should not exceed the nominal threshold (e.g., 70% observed FDR for VI-VS vs 95% for OLS at alpha=0.1 in the Perturb-seq experiment).

## Related tools

- **VI-VS** (Produces the calibrated CRT p-values that are corrected with Benjamini-Hochberg before calling discoveries at target FDR levels) — github.com/YosefLab/VIVS
- **VIVS** (Open-source Python implementation whose model.get_hier_importance outputs p-values that are BH-corrected for multi-resolution testing) — github.com/YosefLab/VIVS
- **GENIE3** (Supplies prediction-error importance scores whose VI-VS-wrapped p-values are BH-corrected and evaluated for FDR/TPR)
- **Jax** (Backend used to run the Monte Carlo replicates of the CRT whose p-values feed into BH correction)
- **scanpy** (AnnData container for the input gene expression counts from which p-values are computed)

## Examples

```
res = model.get_hier_importance(n_clusters_list=[200, 300]); # then apply Benjamini-Hochberg to res p-values and select discoveries at target FDR alpha=0.1
```

## Evaluation signals

- Empirical FDR at each target level should not exceed the nominal threshold (calibration), consistent with VI-VS p-values stochastically dominating the uniform distribution under the null.
- In the Perturb-seq benchmark at alpha=0.1, expect VI-VS estimated FDR near/below the nominal level (observed 70% relative reduction vs OLS's 95% FDR) while TPR is similar to OLS.
- Realized FDR-vs-target-FDR curve should track the diagonal (target level), mirroring Additional file 1: Figure S1 for Poisson-generated responses.
- Multi-resolution analysis: discoveries at coarser resolutions (e.g., n_clusters_list=[200, 300]) should be consistent with gene-level detections and maintain calibrated FDR when a group counts as a true positive if it contains at least one gene-level true positive.
- Check that the number of discoveries increases (or stays stable) as target FDR level increases, with no FDR inflation.

## Limitations

- BH control inherits the validity of the underlying p-values: it requires calibrated p-values such as those from the CRT; applying it to OLS p-values with a misspecified noise model (e.g., Poisson response counts) leads to FDR inflation.
- VI-VS discoveries have no guarantee of functional relevance in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries even with valid FDR control.
- For chromatin accessibility data, a generative model properly accounting for ATAC-seq sparsity is needed to keep p-values (and hence BH-corrected decisions) valid.

## Evidence

- [other] Correct p-values with the Benjamini-Hochberg procedure and call discoveries at several target FDR levels.: [evidence span withheld — anchor status: not_found]
- [intro] We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate.: "We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate."
- [other] VI-VS p-values stochastically dominate the uniform distribution under the null: [evidence span withheld — anchor status: not_found]
- [other] Apply the Benjamini-Hochberg procedure at target FDR alpha=0.1 to the VI-VS p-values: [evidence span withheld — anchor status: not_found]
