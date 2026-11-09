---
name: log10-pvalue-transformation
description: Use when comparing combined p-values produced by different p-value combination
  methods (e.g., metacp vs GWAR on the 67 type-2 diabetes GWAS SNPs) and you need
  an R2 agreement measure on a comparable scale.
when_to_use_negative:
- The statistics to compare are z-scores rather than p-values, where a -log10 transform
  of p-values is not meaningful without first converting to p-values
- You need absolute significance calibration of a single method rather than cross-method
  agreement — R2 on -log10 p-values measures concordance, not correctness of either
  method
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: metacp
  role: Produces the combined p-values (CCT, MinP, MCM, CMC) for the 67 SNPs that
    are then transformed to -log10 scale for R2 agreement computation
  repo: github.com/pbagos/metacp
- name: Python
  role: Runs metacp from the command line and performs the -log10 transformation and
    R2 computation between metacp and GWAR p-values
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:dd849fc58a555be7bce6f6a6bddfe5e9d87dc7983d4bf5cafea5fd5de36696f2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/pbagos/metacp
  tools:
  - Python
  - metacp
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# log10-pvalue-transformation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Transforms combined p-values (e.g., from metacp's CCT, MinP, MCM, CMC and GWAR's MIN2, MAX) onto the -log10 scale so that agreement between methods can be quantified with an R2 correlation coefficient on comparable, tail-weighted values.

## When to use

Use when comparing combined p-values produced by different p-value combination methods (e.g., metacp vs GWAR on the 67 type-2 diabetes GWAS SNPs) and you need an R2 agreement measure on a comparable scale.

## When NOT to use

- The statistics to compare are z-scores rather than p-values, where a -log10 transform of p-values is not meaningful without first converting to p-values
- You need absolute significance calibration of a single method rather than cross-method agreement — R2 on -log10 p-values measures concordance, not correctness of either method

## Inputs

- Combined p-values per SNP from metacp (CCT, MinP, MCM, CMC) written to output.txt
- Reference combined p-values per SNP from GWAR under MIN2 and MAX conditions
- Input p-value file for the 67 type-2 diabetes GWAS SNPs (SNP_p_values.txt format, one row per SNP)

## Outputs

- -log10-transformed combined p-values for each method/condition
- R2 correlation coefficients between each metacp condition and each GWAR reference condition
- Per-condition agreement table (gwar_agreement_table.csv) covering MIN2, MAX, CCT, MinP, MCM and CMC

## How to apply

After running metacp (e.g., 'python metacp.py SNP_p_values.txt output.txt p-values Yes correlation_matrix.txt cct minp mcm cmc') and collecting the reference combined p-values from GWAR under MIN2 and MAX, load both sets of combined p-values for the same 67 SNPs. Transform every combined p-value — metacp's CCT, MinP, MCM (2*min(pCCT, pMinP, 0.5)) and CMC (CCT applied to pCCT and pMinP), plus GWAR's MIN2 and MAX — to -log10(p). Then compute the R2 correlation coefficient between each metacp condition and each GWAR reference condition on the transformed values, assembling a per-condition agreement table (MIN2, MAX, CCT, MinP, MCM, CMC) such as gwar_agreement_table.csv. The -log10 transformation emphasizes the small, tail-region p-values that matter for GWAS significance, making cross-method agreement interpretable.

## Related tools

- **metacp** (Produces the combined p-values (CCT, MinP, MCM, CMC) for the 67 SNPs that are then transformed to -log10 scale for R2 agreement computation) — github.com/pbagos/metacp
- **Python** (Runs metacp from the command line and performs the -log10 transformation and R2 computation between metacp and GWAR p-values)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values Yes correlation_matrix.txt cct minp mcm cmc
```

## Evaluation signals

- R2 values computed on -log10 p-values fall in the range 0.9818 to 1.000 across all metacp-vs-GWAR condition pairs
- The agreement table (gwar_agreement_table.csv) contains rows for all six conditions: MIN2, MAX, CCT, MinP, MCM and CMC
- Every combined p-value transformed is within (0, 1]; -log10 values are non-negative
- The same 67 SNPs from the type-2 diabetes GWAS are present in both the metacp output and the GWAR reference before pairing

## Limitations

- R2 on -log10 p-values measures rank/scale agreement between methods, not which method is more statistically accurate
- The various methods implemented in metacp are not all suitable for the same analysis; agreement does not imply the optimal choice for a given dependency structure
- The benchmark is specific to the 67-SNP type-2 diabetes GWAS dataset; ranges may differ for other data

## Evidence
