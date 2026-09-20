---
name: adjusted-rand-index-benchmark-evaluation
description: Use when comparing predicted cell-type cluster assignments to known ground-truth
  labels for a scRNAseq dataset, e.g. benchmarking clustering pipelines on Zheng68K
  PBMC subsamples, Baron Mouse_1, or SC-Mixology datasets.
when_to_use_negative:
- There are no ground-truth cell-type labels — ARI requires reference labels; use
  internal metrics like silhouette score instead (as CHAI does for selecting best_k).
- Comparing clusterings with each other where sample-size corrections of pairwise
  agreement are not desired, or when evaluating number-of-cluster estimation alone
  rather than assignment agreement.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CHAI
  role: Consensus clustering method whose outputs (CHAI-AvgSim / CHAI-SNF cluster
    assignments) are scored by ARI against ground truth
  repo: https://github.com/lodimk2/chai
- name: SAMEclustering
  role: Provides adjustedRandIndex for computing ARI between ensemble cluster assignments
    and true cell-type labels
  repo: https://github.com/yycunc/SAMEclustering
- name: sc_mixology
  role: Source of benchmarking datasets (SC-Mixology) with known cell-line ground
    truth used in ARI-based evaluation
  repo: https://github.com/LuyiTian/sc_mixology
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:c98853a0fdccb2f15035670741f5fbe74ef3b827500f3d1b59b0d0594780e527
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lodimk2/chai
  tools:
  - CHAI
  - CHAI-AvgSim
  - CHAI-SNF
  - GraphST
  - R
  - SAMEclustering
  - SC3
  - SNFtool
  - Seurat
  - Spectrum
  - scSHC
  - sc_mixology
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# adjusted-rand-index-benchmark-evaluation

## Summary

Evaluate scRNAseq clustering results against ground-truth cell-type labels using the Adjusted Rand Index (ARI), the metric CHAI's paper uses to benchmark consensus clustering methods (CHAI-AvgSim, CHAI-SNF) against individual algorithms and SAME-clustering across benchmarking datasets.

## When to use

Use when comparing predicted cell-type cluster assignments to known ground-truth labels for a scRNAseq dataset, e.g. benchmarking clustering pipelines on Zheng68K PBMC subsamples, Baron Mouse_1, or SC-Mixology datasets.

## When NOT to use

- There are no ground-truth cell-type labels — ARI requires reference labels; use internal metrics like silhouette score instead (as CHAI does for selecting best_k).
- Comparing clusterings with each other where sample-size corrections of pairwise agreement are not desired, or when evaluating number-of-cluster estimation alone rather than assignment agreement.

## Inputs

- Predicted cluster assignment vector (one label per cell) from a clustering method (e.g. CHAI-AvgSim/CHAI-SNF Spectral Clustering output, or Seurat/SC3/Spectrum/scSHC assignments)
- Ground-truth cell-type label vector for the same cells (e.g. Zheng.celltype annotations, Zheng68K subsample labels, sc_mixology cell-line labels)

## Outputs

- Adjusted Rand Index (ARI) value in [-1, 1] per dataset/condition
- Comparison tables of ARI across methods and conditions (e.g. ari_simple_vs_challenging.csv, ari_spatial_comparison.csv)
- Derived summaries such as absolute percent difference in ARI between simple and challenging subsample conditions

## How to apply

Run the candidate clustering methods (e.g. CHAI-AvgSim/CHAI-SNF via Spectral Clustering, or individual algorithms like Seurat-Louvain, SC3, Spectrum) to obtain cluster assignments for every cell. Compute the Adjusted Rand Index between the predicted cluster assignments and the ground-truth cell-type labels, treating ARI as a pairwise-agreement statistic corrected for chance (range [-1, 1], with 1 = perfect agreement). In R this is done with the adjustedRandIndex function (as in the SAMEclustering workflow, where the ensemble's BIC-selected clusters are compared to data_SAME$Zheng.celltype). Aggregate ARI values across datasets and conditions — e.g. six Zheng68K subsamples (1000/2500/5000 cells; simple five-group vs challenging randomly selected groupings) — and summarize differences such as absolute percent difference in ARI between simple and challenging conditions. Interpret negative or near-zero ARI as clustering failure, as the paper notes for Spectrum, which 'often display[s] subzero and negative ARI'.

## Related tools

- **CHAI** (Consensus clustering method whose outputs (CHAI-AvgSim / CHAI-SNF cluster assignments) are scored by ARI against ground truth) — https://github.com/lodimk2/chai
- **SAMEclustering** (Provides adjustedRandIndex for computing ARI between ensemble cluster assignments and true cell-type labels) — https://github.com/yycunc/SAMEclustering
- **sc_mixology** (Source of benchmarking datasets (SC-Mixology) with known cell-line ground truth used in ARI-based evaluation) — https://github.com/LuyiTian/sc_mixology

## Examples

```
adjustedRandIndex(cluster.ensemble$BICcluster, data_SAME$Zheng.celltype)
```

## Evaluation signals

- ARI falls within the valid range [-1, 1] for every dataset/condition computed.
- ARI of the ensemble method (e.g. CHAI-SNF) exceeds the ARI of most individual constituent algorithms and of SAME-clustering on the same datasets.
- On Zheng68K subsamples, CHAI-SNF shows the smallest absolute percent difference in ARI between simple and challenging conditions (including ARI gains on challenging sets), reproducing the paper's robustness finding.
- ARI is computed against the correct ground-truth label vector aligned cell-by-cell with the predicted assignments (e.g. adjustedRandIndex against Zheng.celltype).
- Comparison CSV outputs (e.g. ari_simple_vs_challenging.csv) contain one ARI row per dataset and method with consistent method naming.

## Limitations

- Individual algorithms in the consensus show large performance variation; e.g. Spectrum 'often displaying subzero and negative ARI', which can drag down or skew benchmarking summaries.
- SC3 does not run in the SAME-Clustering package due to a bug, so SC3 may be included in CHAI's pipeline but not in SAME-Clustering's in all evaluations, making ARI-based method comparisons not perfectly apples-to-apples.
- There is currently no method to select an ideal set of algorithms for the consensus pipeline, so ARI results depend on the down-selected algorithm set.

## Evidence

- [readme] We can compare the clustering results to the true labels using the ARI.: [evidence span withheld — anchor status: external_doc]
- [readme] adjustedRandIndex call on BIC clusters vs ground truth: [evidence span withheld — anchor status: external_doc]
- [discussion] Spectrum negative ARI: [evidence span withheld — anchor status: not_found]
- [discussion] SC3 bug in SAMEclustering: "SC3 does not run in its package due to a bug (see: https://github.com/yycunc/SAMEclustering/issues/4"
- [abstract] CHAI superior performance claim: "CHAI-AvgSim and CHAI-SNF demonstrate superior performance across several benchmarking datasets"
