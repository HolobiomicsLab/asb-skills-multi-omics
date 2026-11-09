---
name: spectral-clustering-with-silhouette-k-selection
description: Use when a consensus/aggregate similarity matrix (e.g., an SNF matrix
  from CHAI-SNF or an Average Similarity matrix from CHAI-AvgSim) must be converted
  into final cluster assignments, and the true number of cell types is unknown.
when_to_use_negative:
- When the true number of clusters is already known and you want to fix k directly
  — set eval = FALSE and pass best_k instead of running the silhouette search.
- When the input is raw count data rather than an integrated similarity matrix — the
  seven component clustering algorithms must run first to produce binary similarity
  matrices.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CHAI
  role: R package whose CHAI_AvgSim/CHAI_SNF wrappers run Spectral Clustering on the
    integrated matrix and evaluate best_k via silhouette score
  repo: https://github.com/lodimk2/chai
- name: SNFtool
  role: Produces the snf_matrix (Similarity Network Fusion) on which Spectral Clustering
    is run
- name: R
  role: Implementation language of the CHAI package and the Spectral Clustering step
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:f11944e8bf461af8434cd7a5091504e243b4e55132b3ee8ea108c1027a3459c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lodimk2/chai
  tools:
  - CHAI
  - CHAI-SNF
  - GraphST
  - R
  - SNFtool
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Spectral Clustering with Silhouette-based k Selection

## Summary

Final step of the CHAI consensus pipeline: Spectral Clustering is run on the integrated similarity matrix (AvgSim or SNF) to obtain consensus cell-type assignments, with the number of clusters (best_k) chosen by maximizing the silhouette score when the true cluster count is unknown.

## When to use

Use when a consensus/aggregate similarity matrix (e.g., an SNF matrix from CHAI-SNF or an Average Similarity matrix from CHAI-AvgSim) must be converted into final cluster assignments, and the true number of cell types is unknown.

## When NOT to use

- When the true number of clusters is already known and you want to fix k directly — set eval = FALSE and pass best_k instead of running the silhouette search.
- When the input is raw count data rather than an integrated similarity matrix — the seven component clustering algorithms must run first to produce binary similarity matrices.

## Inputs

- Integrated similarity matrix (snf_matrix or AvgSim_matrix) from CHAI-SNF or CHAI-AvgSim
- Candidate k range (2 to k+1) or a pre-specified best_k
- Optional: ground-truth cell-type labels for ARI evaluation

## Outputs

- Consensus cluster assignments per cell from Spectral Clustering
- Selected best_k based on optimal silhouette score
- ARI value against ground-truth labels (when labels are available)

## How to apply

After running the seven CHAI component clustering algorithms and aggregating their binary similarity matrices (same cluster = 1, different cluster = 0) via Average Similarity or the SNF algorithm, run Spectral Clustering on the integrated matrix to determine final cell identities. When the true number of clusters is unknown, conduct a silhouette score evaluation over the range 2 to k + 1 (where k is the true number of clusters present in the dataset) and select the best_k with the optimal silhouette score. In the CHAI R package, this is controlled by the CHAI_AvgSim/CHAI_SNF wrapper functions: passing eval = TRUE makes CHAI evaluate best_k via silhouette score, whereas eval = FALSE uses the user-supplied best_k directly. The output consensus assignments are then scored against ground-truth labels using Adjusted Rand Index (ARI).

## Related tools

- **CHAI** (R package whose CHAI_AvgSim/CHAI_SNF wrappers run Spectral Clustering on the integrated matrix and evaluate best_k via silhouette score) — https://github.com/lodimk2/chai
- **SNFtool** (Produces the snf_matrix (Similarity Network Fusion) on which Spectral Clustering is run)
- **R** (Implementation language of the CHAI package and the Spectral Clustering step)

## Examples

```
sce <- CHAI_SNF(sce, best_k = 15, eval = TRUE)
```

## Evaluation signals

- Selected best_k corresponds to the maximum silhouette score over the evaluated range 2 to k+1
- Final cluster assignments are valid (every cell assigned exactly one cluster; number of clusters equals best_k)
- When ground-truth labels are available, ARI is computed and falls within the valid [-1, 1] range
- With eval = TRUE in CHAI wrappers, silhouette evaluation replaces any user-supplied best_k; confirm behavior matches the intended setting
- On the spatial transcriptomics task, ARI from the spatially integrated SNF matrix should exceed the baseline CHAI-SNF ARI on both SCP2046 and SCP2331 datasets

## Limitations

- Silhouette-based selection requires evaluating the range 2 to k+1 relative to the true cluster count; it is a heuristic, and CHAI's authors note individual component algorithms show large performance variation (e.g., Spectrum often yields subzero/negative ARI), which can affect the integrated matrix quality fed to Spectral Clustering.
- If eval is set to FALSE, the user-supplied best_k is used as-is, so a poor k choice directly degrades the consensus clustering.

## Evidence

- [intro] users can determine the "best_k" for Spectral Clustering by evaluating the optimal silhouette score: "users can determine the "best_k" for Spectral Clustering by evaluating the optimal silhouette score"
- [methods] we conduct a silhouette score evaluation in range 2 to k + 1, with k being the true number of clusters present in the dataset: [evidence span withheld — anchor status: not_found]
- [readme] If "eval" is set to TRUE, CHAI will evaluate the best_k for Spectral Clustering using silhouette score.: "If "eval" is set to TRUE, CHAI will evaluate the best_k for Spectral Clustering using silhouette score. If "eval" is set to false, CHAI will use best_k as the k value for silhouette score."
- [methods] Run Spectral Clustering on either integrated matrix to determine final cell identities.: "(iii) Run Spectral Clustering on either integrated matrix to determine final cell identities."
