---
name: chi-squared-tail-probability-computation
description: Use when combining independent p-values (or z-scores) per row of a text
  file — e.g. GWAS SNP or gene p-values across k tests — with Fisher's or inverse
  chi-squared methods that need chi2 CDF/quantile evaluation with df = 2k or k.
when_to_use_negative:
- When the p-values to be combined are dependent — the article notes 'the various
  methods implemented are not all suitable for the same analysis', and dependent combinations
  require correlation-adjusted methods (Brown's method, Cauchy Combination Test) rather
  than plain independent chi-squared combination
- When the input statistics are z-scores without conversion — the user must explicitly
  specify 'p-values' or 'z-scores' as the 3rd CLI argument, and Fisher/invchi formulas
  here assume p-value scale
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: metacp
  role: Exposes Fisher's and inverse chi-squared p-value combination (which internally
    require chi-squared CDF/quantile evaluation) via a Python/STATA command-line interface
    taking an input p-value file and writing combined p-values to an output file
  repo: https://github.com/pbagos/metacp
- name: Python
  role: Implementation language for metacp; chi-squared tail probabilities can be
    validated against closed-form values using scipy.stats chi2 survival functions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:e06c9acdaa261c0b9cefc6f3510d534b81ad34f1d707800c8b53b57bbc4d65fa
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
  - Python
  - metacp
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# chi-squared-tail-probability-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes chi-squared cumulative distribution and tail probabilities required by the p-value combination methods implemented in metacp (Fisher's method and the inverse chi-squared method), which are used to combine independent p-values across columns of an n-rows × k-columns input file.

## When to use

Use when combining independent p-values (or z-scores) per row of a text file — e.g. GWAS SNP or gene p-values across k tests — with Fisher's or inverse chi-squared methods that need chi2 CDF/quantile evaluation with df = 2k or k.

## When NOT to use

- When the p-values to be combined are dependent — the article notes 'the various methods implemented are not all suitable for the same analysis', and dependent combinations require correlation-adjusted methods (Brown's method, Cauchy Combination Test) rather than plain independent chi-squared combination
- When the input statistics are z-scores without conversion — the user must explicitly specify 'p-values' or 'z-scores' as the 3rd CLI argument, and Fisher/invchi formulas here assume p-value scale

## Inputs

- Text file of n rows and k columns (plus headers) containing p-values or z-scores, one variable (SNP/gene/probe) per row
- User-selected combination method name(s) such as 'fisher' or 'invchi' passed on the metacp command line

## Outputs

- Combined p-value per row (per variable), written to a user-specified output text file (e.g. output.txt)

## How to apply

Load the p-values from the input text file of n rows and k columns (plus headers), where each row holds the statistics for one SNP/gene/probe. Implement Fisher's method as pc = 1 - F(-2*sum(ln p_i), 2k), i.e. evaluate the chi-squared cumulative distribution with 2k degrees of freedom at -2 times the summed log-transformed p-values. Implement the inverse chi-squared method as pc = 1 - F(sum(F^{-1}(1-p_i, 1)), k), summing the inverse chi-squared statistics with 1 df each and evaluating the chi-squared CDF with k degrees of freedom. Verify each implementation against closed-form reference values (e.g. scipy.stats chi2 survival functions) within a tolerance such as absolute error < 1e-8. These tail computations underpin the 'fisher' and 'invchi' methods exposed by the metacp CLI, which writes the combined p-values to a user-specified output file.

## Related tools

- **metacp** (Exposes Fisher's and inverse chi-squared p-value combination (which internally require chi-squared CDF/quantile evaluation) via a Python/STATA command-line interface taking an input p-value file and writing combined p-values to an output file) — https://github.com/pbagos/metacp
- **Python** (Implementation language for metacp; chi-squared tail probabilities can be validated against closed-form values using scipy.stats chi2 survival functions)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values No fisher meanp bonferroni
```

## Evaluation signals

- Fisher's method on a small test vector reproduces the closed-form value pc = 1 - F(-2*sum(ln p_i), 2k) with absolute error < 1e-8 (e.g. against scipy.stats.chi2.sf)
- Inverse chi-squared method reproduces pc = 1 - F(sum(F^{-1}(1-p_i, 1)), k) within the same numerical tolerance
- Degrees of freedom are correct: 2k for Fisher's method, k for the inverse chi-squared method, where k is the number of columns in the input file
- All methods run on the loaded input file without error and each row of the input yields exactly one combined p-value in output.txt
- Combined p-values fall in [0, 1] and are monotone with respect to the individual p-values in the expected direction

## Limitations

- The various implemented methods are not all suitable for the same analysis; chi-squared-based independent methods assume independent tests
- The article's discussion notes the optimal choice for dependent tests is highly problem dependent, and in some multi-omics studies the correlation structure of the test statistics is hard to decipher

## Evidence

- [readme] fisher: This method uses Fisher's method to combine p-values by summing the log-transformed p-values and applying a chi-squared distribution to the result.: "fisher: This method uses Fisher's method to combine p-values by summing the log-transformed p-values and applying a chi-squared distribution to the result."
- [readme] invchi: This method combines p-values using the inverse chi-squared method, summing the inverse chi-squared statistics and transforming the sum using a chi-squared distribution.: "invchi: This method combines p-values using the inverse chi-squared method, summing the inverse chi-squared statistics and transforming the sum using a chi-squared distribution."
- [intro] metacp is designed to perform meta-analysis of the p-values of the variables specified by the user. The program takes as input a text file of n rows and k columns: "metacp is designed to perform meta-analysis of the p-values of the variables specified by the user. The program takes as input a text file of n rows and k columns, plus the headers, which contain the"
- [discussion] Of course, the various methods implemented are not all suitable for the same analysis.: "Of course, the various methods implemented are not all suitable for the same analysis."
