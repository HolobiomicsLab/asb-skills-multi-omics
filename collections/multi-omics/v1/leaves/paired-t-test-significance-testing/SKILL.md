---
name: paired-t-test-significance-testing
description: Use when comparing the clustering performance (AMI) of two methods evaluated
  on the same matched set of datasets or simulation replicates — e.g., 12 paired section-level
  AMI values — and you need a p-value for whether the improvement is statistically
  significant.
when_to_use_negative:
- When methods are evaluated on unmatched datasets or independent replicates — the
  paired design (same 12 sections per method) is what justifies the paired t-test
- When the outcome is not a per-dataset continuous score (e.g., only a single aggregated
  AMI value per method with no per-section breakdown), since the paired test requires
  matched per-section AMI vectors
- When AMI has not yet been computed against reference labels; the test operates on
  pre-computed clustering accuracy scores, not on raw cluster assignments
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: SMOPCA
  role: Method under test whose per-section AMI values are compared against baselines
    via the paired t-test
- name: SpatialGlue_3M
  role: Extended three-modality version of SpatialGlue serving as a baseline competitor;
    its AMI vector enters the significance test
  repo: https://github.com/JinmiaoChenLab/SpatialGlue_3M
- name: SpatialPCA
  role: Spatially aware dimension reduction baseline whose AMI values are tested against
    SMOPCA
  repo: https://github.com/shangll123/SpatialPCA
- name: SpaVAE
  role: Spatial variational autoencoder baseline included in the per-section AMI comparison
  repo: https://github.com/ttgump/spaVAE
- name: scikit-learn KMeans
  role: PCA + K-means baseline (sklearn.cluster.KMeans) producing cluster labels scored
    by AMI before significance testing
- name: SRTsim
  role: Generates the 12 simulated three-omics DLPFC sections whose matched per-section
    design enables the paired test
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:bac35b24b9e30df920cd0d11706b6509e33565ff4776413599a110e0a8a6df6e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hemberg-lab/SC3
  repo_url: https://github.com/JinmiaoChenLab/SpatialGlue_3M
  tools:
  - SMOPCA
  - SRTsim
  - SpaVAE
  - SpatialGlue_3M
  - SpatialPCA
  - scikit-learn KMeans
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# paired-t-test-significance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistical significance testing of clustering performance differences between methods by applying a paired t-test over per-section adjusted mutual information (AMI) scores, as used in the SMOPCA three-omics DLPFC simulation benchmark across 12 SRTsim-simulated sections.

## When to use

Use when comparing the clustering performance (AMI) of two methods evaluated on the same matched set of datasets or simulation replicates — e.g., 12 paired section-level AMI values — and you need a p-value for whether the improvement is statistically significant.

## When NOT to use

- When methods are evaluated on unmatched datasets or independent replicates — the paired design (same 12 sections per method) is what justifies the paired t-test
- When the outcome is not a per-dataset continuous score (e.g., only a single aggregated AMI value per method with no per-section breakdown), since the paired test requires matched per-section AMI vectors
- When AMI has not yet been computed against reference labels; the test operates on pre-computed clustering accuracy scores, not on raw cluster assignments

## Inputs

- Per-method AMI vectors (one AMI score per simulated section) for SMOPCA and each baseline method, computed from clusterings on the 12 simulated three-omics DLPFC sections
- Reference spatial domain labels for the 12 LIBD DLPFC sections (151507-151510, 151669-151676)
- Cluster label assignments from SMOPCA, PCA + K-means, SpatialPCA, SpaVAE, and SpatialGlue_3M on each simulated section

## Outputs

- AMI comparison table with per-section and per-method AMI values plus paired t-test statistic and p-value, written to ami_comparison_dlpfc.csv
- Summary AMI statistics (e.g., mean per method) across the 12 sections

## How to apply

First compute each method's clustering quality as adjusted mutual information (AMI) against reference labels on each of the 12 simulated DLPFC sections, yielding one AMI vector per method (SMOPCA, PCA + K-means via sklearn.cluster.KMeans, SpatialPCA, SpaVAE, SpatialGlue_3M). Because all methods run on the same SRTsim-simulated sections, observations are matched, so a paired t-test across the 12 sections is appropriate rather than an independent-samples test. Summarize AMI per method (e.g., mean) and perform the paired t-test with SMOPCA's AMI vector as one sample and each competitor's as the other. A significant result (small p-value) supports the claim that SMOPCA's performance difference is statistically significant, not just numerically higher. Record the t-statistic and p-value alongside the AMI table in the output file ami_comparison_dlpfc.csv.

## Related tools

- **SMOPCA** (Method under test whose per-section AMI values are compared against baselines via the paired t-test)
- **SpatialGlue_3M** (Extended three-modality version of SpatialGlue serving as a baseline competitor; its AMI vector enters the significance test) — https://github.com/JinmiaoChenLab/SpatialGlue_3M
- **SpatialPCA** (Spatially aware dimension reduction baseline whose AMI values are tested against SMOPCA) — https://github.com/shangll123/SpatialPCA
- **SpaVAE** (Spatial variational autoencoder baseline included in the per-section AMI comparison) — https://github.com/ttgump/spaVAE
- **scikit-learn KMeans** (PCA + K-means baseline (sklearn.cluster.KMeans) producing cluster labels scored by AMI before significance testing)
- **SRTsim** (Generates the 12 simulated three-omics DLPFC sections whose matched per-section design enables the paired test)

## Examples

```
from scipy.stats import ttest_rel; t, p = ttest_rel(ami_smopca, ami_spatialpca)  # ami_* = 12 per-section AMI values; write t, p to ami_comparison_dlpfc.csv
```

## Evaluation signals

- Exactly 12 paired AMI observations per method — one per simulated DLPFC section (151507-151510, 151669-151676) — with no missing sections, since pairing is violated otherwise
- SMOPCA's mean AMI should exceed that of PCA + K-means, SpatialPCA, SpaVAE, and SpatialGlue_3M before the test can confirm significance
- Output file ami_comparison_dlpfc.csv contains both the AMI table and the paired t-test statistic and p-value
- p-values below 0.05 for the SMOPCA-vs-baseline contrasts reproduce the paper's conclusion of statistically significant improvement under the paired t-test
- Identical section ordering across methods' AMI vectors (pairing must map section i to section i)

## Limitations

- With only 12 paired sections, the t-test relies on normality assumptions for AMI differences and has limited power; it is a per-section summary rather than per-spot inference
- Statistical significance of the AMI difference does not by itself guarantee biologically superior spatial domain detection — the conclusion is tied to the SRTsim simulation design and the 5000-HVG feature space

## Evidence

- [full_text] Paired t-test across 12 simulated sections: [evidence span withheld — anchor status: not_found]
- [full_text] AMI scoring protocol: [evidence span withheld — anchor status: not_found]
- [full_text] Simulation setup justifying matched sections: [evidence span withheld — anchor status: not_found]
