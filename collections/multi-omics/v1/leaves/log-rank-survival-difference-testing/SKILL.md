---
name: log-rank-survival-difference-testing
description: Use when patient samples have both cluster assignments from network-based
  stratification (e.g., pyPARAGON clusters of 105 breast cancer tumors) and clinical
  metadata with survival time and censoring status, and the goal is to test whether
  clusters differ in overall survival.
when_to_use_negative:
- Input lacks per-patient censoring status or survival times — Kaplan-Meier fitting
  and the log-rank test are not defined
- The task is to obtain the clusters themselves (network propagation, GGN construction,
  PageRank-flux edge scoring, Louvain community detection) rather than to test survival
  differences of already-assigned clusters
- Samples are not stratified into discrete groups (e.g., a single cohort with no cluster
  labels) — there is no between-group contrast for the log-rank test
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: pyPARAGON
  role: Produces the network-based patient stratification (patient clusters) whose
    overall survival is then compared with Kaplan-Meier/log-rank analysis
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: 'Implementation environment: lifelines KaplanMeierFitter and multivariate_logrank
    test plus pandas merging and matplotlib plotting'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:5ccfc7d1047cb26c03bf592f20a69e3af0945cf1b84ba3e2b805fc709cbc09ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# log-rank-survival-difference-testing

## Summary

Kaplan-Meier estimation and log-rank (multivariate_logrank) testing to compare overall survival between patient clusters, applied here to the four clusters from pyPARAGON network-based stratification of 105 CPTAC breast cancer tumors. It yields per-cluster survival curves with confidence intervals and a p-value quantifying the survival difference between specific clusters.

## When to use

Use when patient samples have both cluster assignments from network-based stratification (e.g., pyPARAGON clusters of 105 breast cancer tumors) and clinical metadata with survival time and censoring status, and the goal is to test whether clusters differ in overall survival.

## When NOT to use

- Input lacks per-patient censoring status or survival times — Kaplan-Meier fitting and the log-rank test are not defined
- The task is to obtain the clusters themselves (network propagation, GGN construction, PageRank-flux edge scoring, Louvain community detection) rather than to test survival differences of already-assigned clusters
- Samples are not stratified into discrete groups (e.g., a single cohort with no cluster labels) — there is no between-group contrast for the log-rank test

## Inputs

- Patient cluster assignments (cluster labels 1-4 for the 105 CPTAC breast cancer tumors from pyPARAGON network-based stratification)
- Clinical overall-survival metadata table with survival time and censoring status per patient

## Outputs

- Per-cluster Kaplan-Meier survival curves with 95% confidence intervals (matplotlib figure)
- Log-rank test p-value for the cluster-4 vs cluster-1 survival comparison
- Verdict on whether one cluster shows significantly worse survival (p < 0.05)

## How to apply

Load the patient cluster assignments from the network-based stratification sub-task and merge them with clinical overall-survival metadata (survival time, censoring flag) for the 105 CPTAC breast cancer tumors using pandas. Fit a KaplanMeierFitter (lifelines) per cluster using survival time as duration and the censoring flag as event indicator. Compare the clusters of interest (in the pyPARAGON breast cancer case, cluster-4 vs cluster-1) with lifelines' multivariate_logrank (log-rank) test to obtain the p-value for the survival difference. Plot the Kaplan-Meier curves with confidence intervals using lifelines' plotting utilities in matplotlib, annotating the log-rank p-value on the figure. Judgment: the reproduction is correct if cluster-4 shows lower survival probability than cluster-1 and the log-rank p-value falls below the paper's significance threshold of p < 0.05 (as in Fig. 4C / Fig. S5).

## Related tools

- **pyPARAGON** (Produces the network-based patient stratification (patient clusters) whose overall survival is then compared with Kaplan-Meier/log-rank analysis) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Implementation environment: lifelines KaplanMeierFitter and multivariate_logrank test plus pandas merging and matplotlib plotting)

## Examples

```
from lifelines.statistics import multivariate_logrank_test; results = multivariate_logrank_test(df['survival_time'], groups=df['cluster'], event_observed=df['censored']); print(results.p_value)
```

## Evaluation signals

- Cluster-4 Kaplan-Meier curve lies below cluster-1 curve over follow-up (lower survival probability), matching Fig. 4C / Fig. S5
- multivariate_logrank p-value for cluster-4 vs cluster-1 is below the reported significance threshold (p < 0.05)
- Merged survival table covers all 105 tumors with a valid censoring indicator (event/censored) per patient
- Fitted curves include confidence intervals and the figure annotates the log-rank p-value
- Curve step-downs occur only at times with observed events (deaths), not at censored times

## Limitations

- The comparison inherits any errors from the upstream clustering: pyPARAGON's inferred networks can contain generic biological processes (transcription, replication), making patient stratification and its biological interpretation challenging
- Reference interactome incompleteness biases inferred networks toward protein complexes over metabolic pathways, which can affect the clusters whose survival is being compared
- pyPARAGON does not handle alternative copies of hits such as protein isoforms and post-translational modifications; PTM information is generalized and potentially lost in the network

## Evidence

- [readme] This integration of annotations and inferred subnetworks provides patient stratification, survival analysis, and personalized medicine.: "This integration of annotations and inferred subnetworks provides patient stratification, survival analysis, and personalized medicine."
- [discussion] Patients were clustered based on the network-based similarity between the overrepresented biological processes identified with functional modules.: "Patients were clustered based on the network-based similarity between the overrepresented biological processes identified with functional modules."
- [intro] We demonstrated the utility of pyPARAGON in patient stratification using a breast cancer dataset comprising 105 tumors and associated phosphoproteomic data: "We demonstrated the utility of pyPARAGON in patient stratification using a breast cancer dataset comprising 105 tumors and associated phosphoproteomic data"
