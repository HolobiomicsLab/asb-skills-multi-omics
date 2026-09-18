---
name: spearman-correlation-validation
description: Use when a single-cell or ensemble-derived continuous genomic track (e.g.,
  compartment eigenvector E1) must be quantitatively validated against a matching
  bulk assay track on a common genome assembly and bin grid.
when_to_use_negative:
- Input tracks are categorical or binary calls (e.g., peak sets or labels) rather
  than continuous per-bin eigenvector values — a rank correlation of eigenvectors
  is not the appropriate validation.
- Contact maps are not yet aggregated to a common resolution and bin grid (e.g., 1-kb
  cool bins versus 500-kb eigenvector bins) — misaligned bins will invalidate the
  correlation.
- The comparison target lacks compartment structure or a phasing track, so E1 sign
  is arbitrary and correlation sign becomes meaningless without consistent phasing.
edam_operation: http://edamontology.org/operation_0573
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: cooltools
  role: eigs_cis computes the compartment eigenvector E1 from cool files at 500 kb
    resolution with H3K4me3-based phasing
- name: scipy
  role: scipy.stats.spearmanr computes the Spearman rank correlation between sci-L3-Hi-C
    ensemble and bulk GM12878 eigenvectors
- name: Python
  role: Glue language for loading cool files, aggregating bins, aligning eigenvector
    tracks, and writing compartment_correlation.tsv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkaf101
    title: Generalization of the sci-L3 method to achieve high-throughput linear amplification
      for replication template strand sequencing, genome conformation capture, and
      the joint profiling of RNA and chromatin accessibility
  source_content_hash: sha256:ac4cbe63c87c3e5a34808015684f1af9b27b8d1a6048408c30918142ba07647d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/recombinationlab/sciL3Pipe
  tools:
  - Python
  - cooltools
  - scipy
  edam_operation: http://edamontology.org/operation_0573
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# spearman-correlation-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that single-cell-derived genomic tracks reproduce a bulk reference signal by computing rank correlation between aligned per-bin values — here, sci-L3-Hi-C ensemble A/B compartment eigenvectors versus bulk GM12878 Hi-C eigenvectors at 500 kb resolution.

## When to use

Use when a single-cell or ensemble-derived continuous genomic track (e.g., compartment eigenvector E1) must be quantitatively validated against a matching bulk assay track on a common genome assembly and bin grid.

## When NOT to use

- Input tracks are categorical or binary calls (e.g., peak sets or labels) rather than continuous per-bin eigenvector values — a rank correlation of eigenvectors is not the appropriate validation.
- Contact maps are not yet aggregated to a common resolution and bin grid (e.g., 1-kb cool bins versus 500-kb eigenvector bins) — misaligned bins will invalidate the correlation.
- The comparison target lacks compartment structure or a phasing track, so E1 sign is arbitrary and correlation sign becomes meaningless without consistent phasing.

## Inputs

- Ensemble of sci-L3-Hi-C single-cell contact maps stored as 1-kb cool files (hg38)
- Bulk GM12878 Hi-C cool file produced by the same sci-L3 processing pipeline
- ENCODE H3K4me3 ChIP-seq signal track (ENCFF818GNV) for E1 phasing

## Outputs

- 500-kb compartment eigenvector table for sci-L3-Hi-C ensemble and bulk GM12878, aligned by genomic bin
- Spearman correlation statistics (R and P-value), expected R = 0.87, P < 2.2e-16
- compartment_correlation.tsv containing the eigenvector table and correlation statistics

## How to apply

Compute the first cis-eigenvector (E1) from each contact map with cooltools eigs_cis, aggregating 1-kb cooler bins to 500 kb resolution, and phase E1 with the ENCODE H3K4me3 ChIP-seq track (ENCFF818GNV) so positive values denote the A compartment. Aggregate single-cell eigenvector tracks across the sci-L3-Hi-C ensemble and align them to bulk GM12878 eigenvectors by genomic bin on hg38 so the two vectors share identical bin ordering. Compute Spearman rank correlation with scipy.stats.spearmanr, which is rank-based and therefore robust to the nonlinear bin-level differences expected between sparse single-cell ensembles and deeply sequenced bulk maps. Judge success against the published benchmark: Spearman R = 0.87 with P < 2.2e-16, reproducing the canonical plaid A/B compartment pattern. Persist the per-bin eigenvector table and correlation statistics (e.g., compartment_correlation.tsv) so the comparison is auditable.

## Related tools

- **cooltools** (eigs_cis computes the compartment eigenvector E1 from cool files at 500 kb resolution with H3K4me3-based phasing)
- **scipy** (scipy.stats.spearmanr computes the Spearman rank correlation between sci-L3-Hi-C ensemble and bulk GM12878 eigenvectors)
- **Python** (Glue language for loading cool files, aggregating bins, aligning eigenvector tracks, and writing compartment_correlation.tsv)

## Examples

```
from scipy.stats import spearmanr; R, P = spearmanr(sciL3_ensemble_eigs['E1'], bulk_gm12878_eigs['E1'])  # expected R=0.87, P<2.2e-16
```

## Evaluation signals

- Spearman R between sci-L3-Hi-C ensemble and bulk GM12878 E1 reaches ~0.87 with P < 2.2e-16
- Positive E1 values coincide with ENCODE H3K4me3 (ENCFF818GNV) signal, confirming correct A-compartment phasing
- Eigenvector tracks align 1:1 by genomic bin on hg38 at 500 kb resolution with no bin-count mismatch
- The compartment heatmap exhibits the canonical plaid A/B pattern when visualized
- Output compartment_correlation.tsv exists with a complete per-bin eigenvector table plus R and P columns

## Limitations

- sci-L3-Hi-C uses less frequent cutters than other single-cell Hi-C methods (2-4x less frequent), capturing fewer cis interactions, which can weaken single-cell contact maps and thus ensemble eigenvector estimates
- Eigenvector comparison depends on consistent H3K4me3 phasing; without it, anti-correlated vectors with equal magnitude would appear as strong negative correlation
- The benchmark R = 0.87 is specific to GM12878 at 500 kb resolution on hg38; other cell types, resolutions, or assemblies are not validated by this card

## Evidence

- [results] ensemble correlation benchmark: [evidence span withheld — anchor status: not_found]
