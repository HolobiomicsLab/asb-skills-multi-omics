---
name: cancer-patient-cluster-survival-association
description: Use when you have patient cluster assignments from network-based stratification
  (e.g., pyPARAGON Louvain communities) plus clinical survival metadata (survival
  time, censoring status) and need to test survival differences between clusters.
when_to_use_negative:
- You have no survival metadata (survival time and censoring status) for the clustered
  patients — clustering alone cannot be evaluated for survival association
- You need patient stratification itself (network inference, clustering); this skill
  only evaluates survival association of pre-existing cluster assignments
- The outcome of interest is not time-to-event (e.g., binary response or continuous
  biomarker) — Kaplan-Meier/log-rank does not apply
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: pyPARAGON
  role: 'Source of the patient cluster assignments being evaluated: its community
    analysis (Louvain) integrates annotations and inferred subnetworks to provide
    patient stratification for survival analysis'
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Implementation environment for merging cluster and survival metadata (pandas),
    fitting Kaplan-Meier curves (lifelines KaplanMeierFitter), log-rank testing, and
    plotting (matplotlib)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:f96c004c3bb97b0256ad0b310144efe6a0f06a302b330d61fe8c903495613c02
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# cancer-patient-cluster-survival-association

## Summary

Reproduces Kaplan-Meier overall survival analysis with log-rank testing for patient clusters obtained from network-based stratification, testing whether a specific cluster (e.g., cluster-4 from pyPARAGON stratification of 105 CPTAC breast cancer tumors) shows significantly worse survival than another (e.g., cluster-1).

## When to use

Use when you have patient cluster assignments from network-based stratification (e.g., pyPARAGON Louvain communities) plus clinical survival metadata (survival time, censoring status) and need to test survival differences between clusters.

## When NOT to use

- You have no survival metadata (survival time and censoring status) for the clustered patients — clustering alone cannot be evaluated for survival association
- You need patient stratification itself (network inference, clustering); this skill only evaluates survival association of pre-existing cluster assignments
- The outcome of interest is not time-to-event (e.g., binary response or continuous biomarker) — Kaplan-Meier/log-rank does not apply

## Inputs

- Patient cluster assignment table (cluster labels for the 105 breast cancer tumors from network-based stratification)
- Clinical overall-survival metadata: survival time and censoring/event status per patient
- Merged pandas DataFrame keyed by patient/sample ID

## Outputs

- Kaplan-Meier survival curves per cluster (cluster-4 and cluster-1) with confidence intervals, annotated with the log-rank p-value (reproducing Fig. 4C / Fig. S5)
- Log-rank test p-value for the cluster-4 vs cluster-1 survival difference
- Per-cluster survival probability estimates

## How to apply

Load the patient cluster assignments from the network-based stratification of the 105 CPTAC breast-cancer tumors and merge them with the clinical overall-survival metadata (survival time, censoring flag) using pandas. Fit a KaplanMeierFitter from lifelines per patient cluster, using survival time as the duration column and the censoring flag as the event indicator. Compare the cluster of interest versus the reference cluster (cluster-4 vs cluster-1) with the log-rank test (multivariate_logrank_test in lifelines) to obtain the p-value. Plot the Kaplan-Meier curves for the two clusters with confidence intervals using lifelines' plotting utilities in matplotlib, annotating the log-rank p-value on the figure. Validation: cluster-4 must show lower survival probability than cluster-1 and the log-rank p-value must be below the paper's significance threshold of p < 0.05 (Fig. 4C / Fig. S5).

## Related tools

- **pyPARAGON** (Source of the patient cluster assignments being evaluated: its community analysis (Louvain) integrates annotations and inferred subnetworks to provide patient stratification for survival analysis) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Implementation environment for merging cluster and survival metadata (pandas), fitting Kaplan-Meier curves (lifelines KaplanMeierFitter), log-rank testing, and plotting (matplotlib))

## Examples

```
from lifelines import KaplanMeierFitter; from lifelines.statistics import multivariate_logrank_test; kmf4 = KaplanMeierFitter().fit(df[df.cluster==4].time, df[df.cluster==4].event); kmf1 = KaplanMeierFitter().fit(df[df.cluster==1].time, df[df.cluster==1].event); lr = multivariate_logrank_test(df.time, df.cluster)
```

## Evaluation signals

- The reproduced figure shows cluster-4 with a lower survival probability curve than cluster-1 across follow-up
- The multivariate_logrank (log-rank) p-value for cluster-4 vs cluster-1 is below 0.05, matching the paper's significance threshold (Fig. 4C / Fig. S5)
- The merged DataFrame contains survival time and censoring status for the 105 CPTAC breast-cancer tumors, with no dropped patient IDs from the cluster assignments
- Kaplan-Meier curves are fit per cluster with the censoring flag used as the event indicator (not survival time as the event column)
- Confidence intervals are plotted around each Kaplan-Meier curve and the p-value annotation is present on the figure

## Limitations

- The survival comparison is only significant for a specific cluster pair (cluster-4 worse than cluster-1); other cluster pairs may not differ significantly
- With 105 tumors, statistical power is limited and Kaplan-Meier estimates have wide confidence intervals; results depend on censoring patterns
- The clinical significance of the log-rank result depends on correct cluster assignments from the upstream network-based stratification sub-task

## Evidence

- [results] Kaplan–Meier analysis with the log-rank test of overall survival showed that patients in cluster-4 have a significantly worse survival probability than patients in cluster-1: [evidence span withheld — anchor status: not_found]
- [readme] integration of annotations and inferred subnetworks provides patient stratification, survival analysis, and personalized medicine: "This integration of annotations and inferred subnetworks provides patient stratification, survival analysis, and personalized medicine."
- [intro] breast cancer dataset comprising 105 tumors and associated phosphoproteomic data: "We demonstrated the utility of pyPARAGON in patient stratification using a breast cancer dataset comprising 105 tumors and associated phosphoproteomic data"
- [discussion] Patients were clustered based on the network-based similarity between the overrepresented biological processes identified with functional modules: "Patients were clustered based on the network-based similarity between the overrepresented biological processes identified with functional modules."
