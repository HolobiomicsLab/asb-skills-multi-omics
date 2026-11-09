---
name: r2-agreement-assessment
description: Use when two tools or methods produce comparable combined p-value sets
  (e.g. metacp CCT/MinP/MCM/CMC vs GWAR MIN2/MAX on the same SNPs) and you need a
  quantitative agreement metric on the -log10 p-value scale.
when_to_use_negative:
- The tools under comparison were run on different sets of SNPs/tests, so their combined
  p-values are not row-wise comparable
- P-values were combined under independent tests and no correlation matrix or reference
  method output exists to validate against
- Only raw individual p-values are available and no combined p-value sets from two
  methods exist to correlate
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: metacp
  role: Generates the combined p-values (CCT, MinP, MCM, CMC) for the 67 GWAS SNPs
    that are correlated against the GWAR reference conditions
  repo: https://github.com/pbagos/metacp
- name: GWAR
  role: Reference tool whose MIN2 and MAX combined p-values serve as the comparison
    target for computing R2 agreement
- name: Python
  role: Runtime for metacp and for transforming p-values to -log10 scale and computing
    R2 correlations
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:af2a0fa513ace437c54af5e33986cfe429833f9daef677703d3f12ff4438c5d5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/pbagos/metacp
  repo_url: https://github.com/pbagos/metacp
  tools:
  - GWAR
  - Python
  - metacp
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# r2-agreement-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-validate combined p-values produced by different meta-analysis tools/conditions by transforming them to the -log10 scale and computing R2 correlation coefficients, as metacp did against GWAR's MIN2 and MAX on 67 SNPs from a type-2 diabetes GWAS.

## When to use

Use when two tools or methods produce comparable combined p-value sets (e.g. metacp CCT/MinP/MCM/CMC vs GWAR MIN2/MAX on the same SNPs) and you need a quantitative agreement metric on the -log10 p-value scale.

## When NOT to use

- The tools under comparison were run on different sets of SNPs/tests, so their combined p-values are not row-wise comparable
- P-values were combined under independent tests and no correlation matrix or reference method output exists to validate against
- Only raw individual p-values are available and no combined p-value sets from two methods exist to correlate

## Inputs

- Input p-value file for the 67 SNPs from the type-2 diabetes GWAS (SNP_p_values.txt format, one row per SNP)
- Correlation matrix text file for the dependent p-values (correlation_matrix.txt)
- GWAR reference combined p-values for the same 67 SNPs under the MIN2 and MAX conditions

## Outputs

- Combined p-values per SNP from each metacp condition (CCT, MinP, MCM, CMC) written to output.txt
- Per-condition agreement table of R2 values on -log10 p-values (gwar_agreement_table.csv)

## How to apply

Run each tool on the identical input — here the SNP_p_values.txt file of 67 SNPs from the type-2 diabetes GWAS, invoking metacp with the correlation matrix for the dependent methods (cct, minp, mcm, cmc) and collecting GWAR's MIN2 and MAX combined p-values for the same SNPs. Transform every combined p-value from all conditions (MIN2, MAX, CCT, MinP, MCM, CMC) to the -log10 scale so agreement is assessed on magnitude of significance rather than raw p-values. Compute the R2 correlation coefficient between each metacp condition and each GWAR reference condition, assembling the per-condition agreement table (e.g. gwar_agreement_table.csv). Rationale: metacp computes combined p-values without the complex, time-consuming numerical integrations required by GWAR, so high R2 demonstrates the cheaper methods reproduce the reference results; validation expects R2 values in the range 0.9818 to 1.000.

## Related tools

- **metacp** (Generates the combined p-values (CCT, MinP, MCM, CMC) for the 67 GWAS SNPs that are correlated against the GWAR reference conditions) — https://github.com/pbagos/metacp
- **GWAR** (Reference tool whose MIN2 and MAX combined p-values serve as the comparison target for computing R2 agreement)
- **Python** (Runtime for metacp and for transforming p-values to -log10 scale and computing R2 correlations)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values Yes correlation_matrix.txt cct minp mcm cmc
```

## Evaluation signals

- R2 values on -log10 combined p-values between metacp (CCT, MinP, MCM, CMC) and GWAR (MIN2, MAX) fall in the range 0.9818 to 1.000
- Agreement table covers all six conditions (MIN2, MAX, CCT, MinP, MCM, CMC) with one R2 per metacp-vs-GWAR pair
- The same 67 SNPs appear in the input, the metacp output, and the GWAR reference, in matching row order
- MCM combined p-values equal 2*min(pCCT, pMinP, 0.5) and CMC equals CCT applied to pCCT and pMinP, matching the workflow definitions
- All p-values are transformed to -log10 scale before correlation, not correlated on the raw p-value scale

## Limitations

- metacp avoids the complex and time-consuming numerical integrations required by GWAR, but this agreement was demonstrated only on the 67-SNP type-2 diabetes GWAS dataset
- The various methods implemented are not all suitable for the same analysis, so agreement across methods does not imply any single method is optimal
- The optimal choice for combining dependent tests is highly problem dependent, and in some multi-omics studies the correlation structure of the test statistics is hard to decipher

## Evidence
