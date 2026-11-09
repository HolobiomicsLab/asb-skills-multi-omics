---
name: protein-interaction-score-distribution-visualization
description: Use when comparing discovery sets from conditional dependency tests (e.g.,
  VI-VS at 10% target FDR) against a baseline like OLS on gene-protein data, and you
  want to assess biological plausibility via StringDB combined-score distributions.
when_to_use_negative:
- Input is a raw AnnData with unfiltered gene counts — VIVS may not scale to many
  thousands of genes, so run select_genes(n_top_genes=2000) and the significance pipeline
  before visualizing score distributions
- The response variable distribution or interaction structure is unknown and no calibrated
  FDR control is required — VI-VS's StringDB comparison is meaningful only when both
  methods are thresholded at the same target FDR
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: VIVS
  role: Produces the calibrated (BH, 10% FDR) gene-protein association calls whose
    StringDB combined-score distribution is visualized
  repo: https://github.com/YosefLab/VIVS
- name: scVI
  role: Generative model of gene counts used to condition the conditional randomization
    test before association calls are made
  repo: https://github.com/YosefLab/VIVS
- name: scanpy
  role: AnnData container holding raw gene counts and the 224 protein responses in
    obsm
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:029bcee1731a1b128dbb92399308532b084df19e80c4a4502b67e7c3753124fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VIVS
  tools:
  - Jax
  - Python
  - VI-VS
  - VIVS
  - scVI
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# protein-interaction-score-distribution-visualization

## Summary

Maps VI-VS and OLS gene-protein association calls from the CITE-seq PBMC analysis to StringDB combined scores and plots the per-method score distributions, providing functional-plausibility context for the differential discovery counts (VI-VS: 51 proteins, ~52 interactions each; OLS: all 224 proteins, ~140 interactions each).

## When to use

Use when comparing discovery sets from conditional dependency tests (e.g., VI-VS at 10% target FDR) against a baseline like OLS on gene-protein data, and you want to assess biological plausibility via StringDB combined-score distributions.

## When NOT to use

- Input is a raw AnnData with unfiltered gene counts — VIVS may not scale to many thousands of genes, so run select_genes(n_top_genes=2000) and the significance pipeline before visualizing score distributions
- The response variable distribution or interaction structure is unknown and no calibrated FDR control is required — VI-VS's StringDB comparison is meaningful only when both methods are thresholded at the same target FDR

## Inputs

- Association table of predicted gene-protein pairs per method (VI-VS and OLS) with Benjamini-Hochberg FDR calls at 10% target FDR
- StringDB combined scores for the predicted gene-protein pairs
- Per-protein association counts including the four Rat-IgG isotype negative-control proteins

## Outputs

- cite_seq_pbmc_associations.csv — full gene-protein association table per method
- Per-method StringDB combined-score distribution figure

## How to apply

First run the conditional randomization test pipeline (VIVS model with xy_linear=False and n_hidden=8, scVI generative model conditioning on the 13 batch assignments, BH procedure at 10% target FDR) and fit the OLS baseline with the same nuisance correction. Tabulate per-protein association counts for both methods, including detections on the four Rat-IgG isotype negative-control proteins, and write the full table to cite_seq_pbmc_associations.csv. Then map each predicted gene-protein pair to its StringDB combined score and plot the combined-score distribution per method, so the sparser, more stringent VI-VS calls (51 of 224 proteins, ~52 interactions per protein) can be contrasted with the dense OLS calls (all 224 proteins, ~140 interactions per protein) in terms of interaction-network support.

## Related tools

- **VIVS** (Produces the calibrated (BH, 10% FDR) gene-protein association calls whose StringDB combined-score distribution is visualized) — https://github.com/YosefLab/VIVS
- **scVI** (Generative model of gene counts used to condition the conditional randomization test before association calls are made) — https://github.com/YosefLab/VIVS
- **scanpy** (AnnData container holding raw gene counts and the 224 protein responses in obsm)

## Examples

```
import pandas as pd; df = pd.read_csv('cite_seq_pbmc_associations.csv'); df.groupby('method')['stringdb_combined_score'].hist()
```

## Evaluation signals

- cite_seq_pbmc_associations.csv exists and contains per-protein association counts for both VI-VS and OLS
- VI-VS column shows associations for only 51 of the 224 proteins (~52 interactions per protein) while OLS shows associations for all 224 proteins (~140 interactions per protein)
- The four Rat-IgG isotype negative-control proteins appear in the table and their counts are reported for both methods
- Both methods are thresholded at the same 10% target FDR before counting and mapping to StringDB combined scores
- The exported figure shows one combined-score distribution per method, enabling direct comparison

## Limitations

- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries
- OLS returning a very large number of associations for all 224 proteins reflects its lack of FDR calibration, not greater biological signal — score distributions must be interpreted in that light

## Evidence

- [abstract] It uses deep generative models to identify conditionally dependent features, with false discovery rate control.: "It uses deep generative models to identify conditionally dependent features, with false discovery rate control."
- [discussion] VI-VS can be employed as a meta-algorithm to make the discoveries of existing methods more interpretable by constructing importance scores from their predictions.: "VI-VS can be employed as a meta-algorithm to make the discoveries of existing methods more interpretable by constructing importance scores from their predictions."
- [readme] VIVS may not scale to many thousands of genes. In such a case, it is recommended to filter the genes before running VIVS: "VIVS may not scale to many thousands of genes. In such a case, it is recommended to filter the genes before running VIVS"
- [methods] adata = select_genes(adata, n_top_genes=2000): "adata = select_genes(adata, n_top_genes=2000)"
