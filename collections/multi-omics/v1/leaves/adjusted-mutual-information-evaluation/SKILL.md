---
name: adjusted-mutual-information-evaluation
description: Use when benchmarking spatial domain detection or multi-omics clustering
  methods against ground-truth domain labels across multiple sections/replicates and
  a significance test of the ranking is needed.
when_to_use_negative:
- No reference/ground-truth domain labels exist for the tissue sections, since AMI
  is an external-validity metric requiring true labels
- Methods were run on different sets of spots/sections, so the per-section AMI values
  are not paired and the paired t-test is invalid
- Evaluating method stability on spatial structure alone (e.g., preserving neighboring
  similarity) rather than agreement with annotated domains — use spatial correlation
  metrics instead
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SMOPCA
  role: Method under test; its low-dimensional representation is clustered with sklearn
    K-means to yield labels scored by AMI
- name: SpatialGlue_3M
  role: Three-modality comparator for spatial domain detection on the simulated DLPFC
    sections
  repo: https://github.com/JinmiaoChenLab/SpatialGlue_3M
- name: SpatialPCA
  role: Spatially aware dimension reduction comparator producing clusterings evaluated
    by AMI
  repo: https://github.com/shangll123/SpatialPCA
- name: SpaVAE
  role: Spatial variational autoencoder comparator producing clusterings evaluated
    by AMI
  repo: https://github.com/ttgump/spaVAE
- name: scikit-learn KMeans
  role: Applies the final K-means clustering step on low-dimensional vectors to obtain
    comparable label vectors
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:c414767a177ba953042a616c4bb27fa300fc63bc71fc99d7caeec5781caf0faf
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# adjusted-mutual-information-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores predicted spatial domain clusterings against reference labels using adjusted mutual information (AMI), aggregated across replicates (e.g., 12 SRTsim-simulated DLPFC sections) with a paired t-test to establish statistical significance between methods.

## When to use

Use when benchmarking spatial domain detection or multi-omics clustering methods against ground-truth domain labels across multiple sections/replicates and a significance test of the ranking is needed.

## When NOT to use

- No reference/ground-truth domain labels exist for the tissue sections, since AMI is an external-validity metric requiring true labels
- Methods were run on different sets of spots/sections, so the per-section AMI values are not paired and the paired t-test is invalid
- Evaluating method stability on spatial structure alone (e.g., preserving neighboring similarity) rather than agreement with annotated domains — use spatial correlation metrics instead

## Inputs

- Per-section predicted cluster label vectors from each method (e.g., SMOPCA, SpatialGlue_3M, SpaVAE, SpatialPCA, PCA + K-means)
- Reference/ground-truth spatial domain label vectors for each of the 12 SRTsim-simulated DLPFC sections (151507-151510, 151669-151676)

## Outputs

- AMI score per method per section (method x 12 sections matrix)
- Paired t-test statistic and p-value comparing SMOPCA vs. each competing method
- AMI comparison table written to ami_comparison_dlpfc.csv

## How to apply

For each simulated or real section, run every candidate method (here SMOPCA, SpatialGlue_3M, SpaVAE, SpatialPCA, and a PCA + sklearn.cluster.KMeans baseline) to obtain spatial domain cluster labels, then compute adjusted mutual information between each method's predicted labels and the reference labels. Because SMOPCA's final labels come from K-means on its low-dimensional vectors, apply the same sklearn.cluster.KMeans labeling step to all methods' embeddings for a fair comparison. Collect the per-section AMI values into a method-by-section table (12 sections in the DLPFC/SRTsim benchmark) and apply a paired t-test across sections, which respects the paired structure since all methods run on identical inputs. Write the AMI comparison table together with the paired t-test statistic and p-value to an output file such as ami_comparison_dlpfc.csv.

## Related tools

- **SMOPCA** (Method under test; its low-dimensional representation is clustered with sklearn K-means to yield labels scored by AMI)
- **SpatialGlue_3M** (Three-modality comparator for spatial domain detection on the simulated DLPFC sections) — https://github.com/JinmiaoChenLab/SpatialGlue_3M
- **SpatialPCA** (Spatially aware dimension reduction comparator producing clusterings evaluated by AMI) — https://github.com/shangll123/SpatialPCA
- **SpaVAE** (Spatial variational autoencoder comparator producing clusterings evaluated by AMI) — https://github.com/ttgump/spaVAE
- **scikit-learn KMeans** (Applies the final K-means clustering step on low-dimensional vectors to obtain comparable label vectors)

## Examples

```
from sklearn.metrics import adjusted_mutual_info_score; ami = adjusted_mutual_info_score(reference_labels, predicted_labels, average_method='arithmetic')
```

## Evaluation signals

- AMI values fall in [0, 1] (chance-adjusted) for every method-section pair; negative or NaN values indicate a labeling/alignment bug
- Number of scored sections equals 12 (36 simulated three-omics matrices from 12 DLPFC sections), one AMI per method per section
- Paired t-test is computed across the 12 sections per method pair (not an unpaired test), so degrees of freedom should be 11
- The reported conclusion should reproduce: SMOPCA outperforms PCA + K-means, SpatialPCA, SpaVAE, and SpatialGlue_3M with statistically significant differences
- Output CSV (ami_comparison_dlpfc.csv) contains per-method AMI summaries plus the t-test statistic and p-value

## Limitations

- Significance rests on only 12 paired sections, so the paired t-test has limited power and depends on the SRTsim simulation faithfully preserving DLPFC spatial patterns
- AMI depends on the choice of final clustering step (K-means on low-dimensional vectors); differences could partly reflect clustering rather than embedding quality
- Reference labels for the simulated sections derive from the simulation procedure, so AMI measures recovery of simulated domains, not biological truth

## Evidence

- [results] AMI scoring across sections: [evidence span withheld — anchor status: not_found]
- [methods] Paired t-test for significance: [evidence span withheld — anchor status: not_found]
- [methods] Common clustering step for comparability: "K-means clustering (“sklearn.cluster.KMeans”) is performed on the low-dimensional vectors to predict the final clustering labels for SMOPCA."
- [methods] Output file: [evidence span withheld — anchor status: not_found]
