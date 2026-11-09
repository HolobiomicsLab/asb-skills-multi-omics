---
name: tsne-subcluster-verification
description: Use when a spatial transcriptomics method assigns spots to a domain (e.g.,
  region Y spanning tumor and connective tissue) and you must confirm internal heterogeneity,
  such as a reported sub-cluster count, before trusting the domain annotation.
when_to_use_negative:
- The spatial clustering has not yet been run or spots lack domain labels — t-SNE
  on unlabeled spots cannot verify a specific domain's sub-structure
- The domain under inspection is expected to be homogeneous (no reported sub-cluster
  count), so a multi-cluster t-SNE split would be uninterpretable rather than confirmatory
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MCGAE
  role: Produces the per-spot spatial domain labels (tumor-invasion region Y) whose
    sub-cluster structure the t-SNE check verifies
  repo: github.com/yiwen-yang/MCGAE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:fce02475e9a422bc4376918245a7a107d59b442e302d7abb66b1cd38d3fb2b0d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/yiwen-yang/MCGAE
  tools:
  - MCGAE
  - ResNet50
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# t-SNE sub-cluster verification of a spatial domain

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Verifies that a spatial domain identified by MCGAE (e.g., tumor-invasion region Y in the colorectal cancer liver metastasis Visium dataset) biologically coheres by embedding only its spots with t-SNE and confirming expected sub-cluster structure (three main clusters, largest dominated by tumor cells).

## When to use

Use when a spatial transcriptomics method assigns spots to a domain (e.g., region Y spanning tumor and connective tissue) and you must confirm internal heterogeneity, such as a reported sub-cluster count, before trusting the domain annotation.

## When NOT to use

- The spatial clustering has not yet been run or spots lack domain labels — t-SNE on unlabeled spots cannot verify a specific domain's sub-structure
- The domain under inspection is expected to be homogeneous (no reported sub-cluster count), so a multi-cluster t-SNE split would be uninterpretable rather than confirmatory

## Inputs

- Per-spot spatial domain-label table from MCGAE clustering (e.g., crlm_spatial_domains.csv) covering all 10x Visium spots of the CRLM dataset
- Gene expression count matrix and MCGAE unified multimodal embedding for spots assigned to tumor-invasion region Y

## Outputs

- t-SNE embedding of region-Y spots showing three main clusters, with Cluster 0 dominated by tumor cells
- Verification record that the domain-Y sub-cluster count matches the Fig. 4 result (value in expected range)

## How to apply

After MCGAE clusters the unified multimodal embedding and every 10x Visium spot in the CRLM dataset carries a domain label, extract the spots belonging to the annotated tumor-invasion region Y. Run t-SNE on these spots alone (using their expression/embedding features) to visualize whether the domain decomposes into the expected sub-clusters rather than a single homogeneous mass. The article's validation criterion is that region Y resolves three main clusters, with the largest cluster (Cluster 0) consisting mainly of tumor cells, and that lymphatic structures separate from tumor domains, reproducing the Fig. 4 result. Accept the domain annotation only if the sub-cluster count falls in the expected range and the per-spot domain-label table (crlm_spatial_domains.csv) covers all Visium spots with the domain field present.

## Related tools

- **MCGAE** (Produces the per-spot spatial domain labels (tumor-invasion region Y) whose sub-cluster structure the t-SNE check verifies) — github.com/yiwen-yang/MCGAE

## Examples

```
region_y = adata[adata.obs['domain'] == 'Y']; tsne = TSNE(n_components=2).fit_transform(region_y.X)
```

## Evaluation signals

- t-SNE of region-Y spots alone resolves 3 main sub-clusters (value_in_range), matching the Fig. 4 result reported in the paper
- The largest sub-cluster (Cluster 0) consists mainly of tumor cells
- Lymphatic structures separate from tumor domains in the embedding
- The output table crlm_spatial_domains.csv covers all Visium spots (row_count_equals) with the domain field present (field_present)
- The pipeline script runs end-to-end without error

## Limitations

- The paper does not specify the exact t-SNE hyperparameters (perplexity, iterations) or the feature matrix used for the region-Y embedding, so exact reproduction of the 3-cluster split may require tuning
- No version is specified for the pre-trained ResNet50 network used for image feature extraction, which feeds the upstream MCGAE embedding and could shift sub-cluster boundaries
- MCGAE's adjacency-matrix computation has O(n^2) complexity, which limits how many spots can be clustered (and hence verified) on large datasets

## Evidence

- [results] t-SNE analysis of the spots within region Y alone revealed three main clusters, with the largest cluster (Cluster 0) consisting mainly of tumor cells: [evidence span withheld — anchor status: not_found]
