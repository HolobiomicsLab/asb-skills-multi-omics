---
name: censored-survival-data-visualization
description: Use when time-to-event (overall survival) data with a censoring flag
  must be compared across predefined patient clusters — e.g., after network-based
  stratification of the 105 breast cancer tumors into four clusters.
when_to_use_negative:
- Time-to-event data are fully observed with no censoring flag — a standard group
  comparison of observed times suffices instead of Kaplan-Meier/log-rank.
- The goal is upstream pyPARAGON network inference or clustering (GGN construction,
  PPR flux scoring, Louvain community detection) — this skill operates downstream
  on already-assigned patient clusters.
- No clinical survival metadata (survival time and censoring status) are available
  for the tumors, so no censored survival comparison can be fit.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2269
tools:
- name: lifelines
  role: KaplanMeierFitter fits per-cluster survival curves; multivariate_logrank tests
    the cluster-4 vs cluster-1 difference; plotting utilities render curves with confidence
    intervals
- name: matplotlib
  role: Backend for lifelines' plotting utilities; the Kaplan-Meier figure with annotated
    log-rank p-value is produced in matplotlib
- name: Python
  role: Runtime environment (venv-based install) in which pandas merging, lifelines
    fitting, and plotting are executed
- name: pyPARAGON
  role: Upstream tool whose network-based stratification of the 105 tumors produces
    the patient cluster assignments that this survival analysis compares
  repo: https://github.com/netlab-ku/pyPARAGON
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:580e3c8db44e5668d5ed1ea623a7aade722702138f45b6444a685bca176321d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - Python
  - lifelines
  - matplotlib
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2
n_claims: 5
---

# censored-survival-data-visualization

## Summary

Reproduces Kaplan–Meier overall-survival comparison of patient clusters (e.g., the four clusters from pyPARAGON-based stratification of 105 CPTAC breast cancer tumors) using lifelines' KaplanMeierFitter and log-rank testing, visualized with confidence intervals and an annotated p-value.

## When to use

Use when time-to-event (overall survival) data with a censoring flag must be compared across predefined patient clusters — e.g., after network-based stratification of the 105 breast cancer tumors into four clusters.

## When NOT to use

- Time-to-event data are fully observed with no censoring flag — a standard group comparison of observed times suffices instead of Kaplan-Meier/log-rank.
- The goal is upstream pyPARAGON network inference or clustering (GGN construction, PPR flux scoring, Louvain community detection) — this skill operates downstream on already-assigned patient clusters.
- No clinical survival metadata (survival time and censoring status) are available for the tumors, so no censored survival comparison can be fit.

## Inputs

- Patient cluster assignments from network-based stratification (4 clusters over 105 CPTAC breast cancer tumors)
- Clinical overall-survival metadata per tumor: survival time and censoring status (event indicator)

## Outputs

- Kaplan-Meier survival curves per cluster with confidence intervals (matplotlib figure)
- Log-rank p-value for the cluster-4 vs cluster-1 survival difference, annotated on the plot
- Fitted KaplanMeierFitter survival functions (survival probability vs time per cluster)

## How to apply

Load the patient cluster assignments from the prior network-based stratification sub-task and merge them with the clinical overall-survival metadata (survival time, censoring status) for the 105 CPTAC breast-cancer tumors using pandas. Fit a KaplanMeierFitter per patient cluster in lifelines, using survival time as the duration and the censoring flag as the event indicator. Compare the clusters of interest (cluster-4 vs cluster-1) with the multivariate_logrank (log-rank) test in lifelines to obtain the p-value. Plot the Kaplan-Meier curves with confidence intervals using lifelines' plotting utilities in matplotlib, annotating the log-rank p-value on the figure. Validate against the paper's Fig. 4C / Fig. S5: cluster-4 must show lower survival probability than cluster-1 with a log-rank p-value below 0.05.

## Related tools

- **lifelines** (KaplanMeierFitter fits per-cluster survival curves; multivariate_logrank tests the cluster-4 vs cluster-1 difference; plotting utilities render curves with confidence intervals)
- **matplotlib** (Backend for lifelines' plotting utilities; the Kaplan-Meier figure with annotated log-rank p-value is produced in matplotlib)
- **Python** (Runtime environment (venv-based install) in which pandas merging, lifelines fitting, and plotting are executed)
- **pyPARAGON** (Upstream tool whose network-based stratification of the 105 tumors produces the patient cluster assignments that this survival analysis compares) — https://github.com/netlab-ku/pyPARAGON

## Examples

```
from lifelines import KaplanMeierFitter; kmf = KaplanMeierFitter(); kmf.fit(durations, event_observed); kmf.plot_survival_function(ci_show=True)
```

## Evaluation signals

- The reproduced figure shows cluster-4 with a lower survival probability than cluster-1, matching Fig. 4C / Fig. S5.
- The log-rank p-value for the cluster-4 vs cluster-1 comparison is below the significance threshold reported in the paper (p < 0.05).
- Kaplan-Meier curves are plotted with confidence intervals and the log-rank p-value is annotated on the figure.
- All 105 tumors are accounted for after merging cluster assignments with clinical survival metadata (survival time and censoring flag present for each sample).
- Each cluster's survival curve is fit by its own KaplanMeierFitter instance with survival time as duration and the censoring flag as the event indicator.

## Limitations

- The cohort is small (105 tumors), so per-cluster survival curves and log-rank power depend on cluster sizes and censoring patterns; the paper's significance claim is specific to the cluster-4 vs cluster-1 contrast at p < 0.05.
- In the pyPARAGON paper, survival analysis is presented only as a downstream application of its community-analysis/annotation integration; no survival-specific parameters beyond the p < 0.05 significance threshold are provided.
- The skill requires prior cluster assignments from the stratification sub-task — it cannot generate or validate the clustering itself.

## Evidence

- [readme] This integration of annotations and inferred subnetworks provides patient stratification, survival analysis, and personalized medicine.: "This integration of annotations and inferred subnetworks provides patient stratification, survival analysis, and personalized medicine."
- [abstract] we demonstrate its utility in network-based stratification of patient tumors by integrating phosphoproteomic data from 105 breast cancer tumors with the interactome: "we demonstrate its utility in network-based stratification of patient tumors by integrating phosphoproteomic data from 105 breast cancer tumors with the interactome"
- [discussion] Patients were clustered based on the network-based similarity between the overrepresented biological processes identified with functional modules.: "Patients were clustered based on the network-based similarity between the overrepresented biological processes identified with functional modules."
