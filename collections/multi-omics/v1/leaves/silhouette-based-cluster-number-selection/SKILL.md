---
name: silhouette-based-cluster-number-selection
description: Use when running CHAI-AvgSim or CHAI-SNF on scRNAseq data and the true
  number of clusters is unknown, so highest-ranked_k for Spectral Clustering must
  be estimated from the integrated similarity matrix.
when_to_use_negative:
- The true number of clusters is already known and should be imposed directly (set
  eval = FALSE in CHAI_AvgSim/CHAI_SNF rather than re-estimating k)
- Clustering method other than CHAI's Spectral Clustering step that uses its own model-selection
  criterion (e.g., SAMEclustering selects clusters via AIC/BIC, not silhouette)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CHAI
  role: R package whose CHAI_AvgSim and CHAI_SNF wrappers evaluate best_k via silhouette
    score (eval = TRUE) before running Spectral Clustering
  repo: https://github.com/lodimk2/chai
- name: CHAI-AvgSim
  role: Runs silhouette-based best_k selection on the Average Similarity matrix aggregated
    from binary similarity matrices
  repo: https://github.com/lodimk2/chai
- name: CHAI-SNF
  role: Runs silhouette-based best_k selection on the SNF-integrated matrix (SNFtool,
    default parameters) before Spectral Clustering
  repo: https://github.com/lodimk2/chai
- name: R
  role: Language in which CHAI and its silhouette/spectral-clustering selection are
    implemented
  repo: https://github.com/lodimk2/chai
merged_aliases:
- silhouette-score-cluster-number-selection
merged_alias_records:
- alias: silhouette-score-cluster-number-selection
  slug: silhouette-score-cluster-number-selection
  jaccard_score: 0.6667
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae411
    title: 'CHAI: consensus clustering through similarity matrix integration for cell-type
      identification'
  source_content_hash: sha256:c3b2e4caee1f761f0d01707886872b3914c9873647351374a5e8be67dbf5b1fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lodimk2/chai
  tools:
  - CHAI
  - CHAI-AvgSim
  - CHAI-SNF
  - R
  - SC3
  - SNFtool
  - Seurat
  - Spectrum
  - scSHC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# silhouette-based-cluster-number-selection

## Summary

Selects the number of clusters (best_k) for Spectral Clustering in the CHAI consensus pipeline by maximizing the average silhouette score evaluated over a candidate range, used when the true number of cell types in an scRNAseq dataset is unknown.

## When to use

Use when running CHAI-AvgSim or CHAI-SNF on scRNAseq data and the true number of clusters is unknown, so best_k for Spectral Clustering must be estimated from the integrated similarity matrix.

## When NOT to use

- The true number of clusters is already known and should be imposed directly (set eval = FALSE in CHAI_AvgSim/CHAI_SNF rather than re-estimating k)
- Clustering method other than CHAI's Spectral Clustering step that uses its own model-selection criterion (e.g., SAMEclustering selects clusters via AIC/BIC, not silhouette)

## Inputs

- Integrated similarity matrix from CHAI (CHAI-AvgSim Average Similarity matrix or CHAI-SNF snf_matrix) derived from binary similarity matrices of the seven constituent clustering algorithms
- Candidate k range (2 to k + 1, with k the true number of clusters when known)
- SingleCellExperiment object with counts and logcounts assays (as used by the chai R package)

## Outputs

- best_k value for Spectral Clustering (integer)
- Final consensus cluster assignments for cells obtained by Spectral Clustering at best_k

## How to apply

In CHAI, after aggregating the seven individual clustering solutions into an integrated matrix (Average Similarity matrix for CHAI-AvgSim, or the SNF matrix via SNFtool for CHAI-SNF), evaluate candidate cluster counts with a silhouette score evaluation in the range 2 to k + 1, where k is the true number of clusters if known. Pick the best_k that maximizes the average silhouette score, then run Spectral Clustering on the integrated matrix with that k to obtain final cell identities. If the true cluster number is known (as in the Zheng68K subsampling benchmark), pass it directly by setting eval = FALSE; otherwise leave the default eval = TRUE so CHAI evaluates best_k automatically. In benchmarking workflows, this same silhouette criterion is used on each subsample (1000, 2500, 5000 cells) before computing ARI against ground-truth labels.

## Related tools

- **CHAI** (R package whose CHAI_AvgSim and CHAI_SNF wrappers evaluate best_k via silhouette score (eval = TRUE) before running Spectral Clustering) — https://github.com/lodimk2/chai
- **CHAI-AvgSim** (Runs silhouette-based best_k selection on the Average Similarity matrix aggregated from binary similarity matrices) — https://github.com/lodimk2/chai
- **CHAI-SNF** (Runs silhouette-based best_k selection on the SNF-integrated matrix (SNFtool, default parameters) before Spectral Clustering) — https://github.com/lodimk2/chai
- **R** (Language in which CHAI and its silhouette/spectral-clustering selection are implemented) — https://github.com/lodimk2/chai

## Examples

```
sce <- CHAI_AvgSim(sce, best_k = 15, eval = TRUE)
```

## Evaluation signals

- The chosen best_k falls within the evaluated range 2 to k + 1 (k = true cluster number) and corresponds to the maximum average silhouette score
- Downstream ARI/NMI against ground-truth labels (e.g., on Baron, Muraro, SC-Mixology, Zheng68K benchmark datasets) is computed using clusters from Spectral Clustering at best_k
- Reproducibility check: setting eval = TRUE yields silhouette-estimated k; setting eval = FALSE uses the supplied best_k directly, so ARI should change accordingly
- In the Zheng68K robustness analysis, silhouette-selected k is applied per subsample (1000/2500/5000 cells, simple and challenging) before comparing simple-vs-challenging ARI percent differences

## Limitations

- Silhouette-based selection presumes the candidate range brackets the true cluster number; misspecifying k shifts the 2 to k + 1 search window
- CHAI currently offers no method to select an ideal subset of constituent algorithms, so noisy components (e.g., Spectrum often showing subzero or negative ARI) can degrade the similarity matrix and the silhouette estimate

## Evidence

- [methods] silhouette evaluation range: [evidence span withheld — anchor status: not_found]
- [intro] best_k determination in CHAI: "users can determine the "best_k" for Spectral Clustering by evaluating the optimal silhouette score"
- [readme] eval flag behavior in README: "If "eval" is set to TRUE, CHAI will evaluate the best_k for Spectral Clustering using silhouette score. If "eval" is set to false, CHAI will use best_k as the k value"
- [methods] spectral clustering on integrated matrix: "(iii) Run Spectral Clustering on either integrated matrix to determine final cell identities."
