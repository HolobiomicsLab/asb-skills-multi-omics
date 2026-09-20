---
name: z-score-transformation-and-quantile-functions
description: Use when combining p-values from an n-row by k-column text file (e.g.
  SNP_p_values.txt) with metacp methods such as stouffer, wstouffer, corstouffer,
  meanp, logit, or cct that require p-to-Z or quantile transformations.
when_to_use_negative:
- The input already contains z-scores but you declare the 'p-values' flag — Stouffer-family
  methods assume p-values are converted to Z-scores, so the statistic type must match
  the file contents
- You need GWAR-style combination that relies on numerical integrations — metacp's
  quantile-function approach was designed to avoid exactly those complex and time-consuming
  numerical integrations
- Only Bonferroni-style effective-number-of-tests corrections (bonferroni, adbonferroni,
  cn, lj, gao, galwey) are needed — those methods adjust the minimum p-value using
  eigenvalues of the correlation matrix rather than p-to-Z or Cauchy transformations
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: metacp
  role: Python package invoked via 'python metacp.py' that implements the Stouffer-family,
    meanp, logit, and CCT p-value combination methods using normal, t, and Cauchy
    quantile functions
  repo: github.com/pbagos/metacp
- name: Python
  role: Runtime for metacp.py, called from the command line with input/output files,
    p-values/z-scores flag, and method keywords
- name: STATA
  role: Alternative implementation of metacp; see the metacp.hlp file within STATA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:0faa976ea51dc07b09998ecec0ba8ce3c793b9f7294801074f475f1b13a9f566
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
  - STATA
  - metacp
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# z-score transformation and quantile functions

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts one-sided p-values to Z-scores and evaluates combined statistics against normal, chi-squared, t, and Cauchy quantile functions, as implemented in metacp's Stouffer-family, meanp, logit, and CCT methods.

## When to use

Use when combining p-values from an n-row by k-column text file (e.g. SNP_p_values.txt) with metacp methods such as stouffer, wstouffer, corstouffer, meanp, logit, or cct that require p-to-Z or quantile transformations.

## When NOT to use

- The input already contains z-scores but you declare the 'p-values' flag — Stouffer-family methods assume p-values are converted to Z-scores, so the statistic type must match the file contents
- You need GWAR-style combination that relies on numerical integrations — metacp's quantile-function approach was designed to avoid exactly those complex and time-consuming numerical integrations
- Only Bonferroni-style effective-number-of-tests corrections (bonferroni, adbonferroni, cn, lj, gao, galwey) are needed — those methods adjust the minimum p-value using eigenvalues of the correlation matrix rather than p-to-Z or Cauchy transformations

## Inputs

- Text file of n rows and k columns (plus headers) containing p-values or z-scores, e.g. SNP_p_values.txt
- CLI flag declaring the statistic type: 'p-values' or 'z-scores'
- Optional correlation matrix text file (for dependent-test methods such as corstouffer)
- Optional weight matrix text file (for weighted methods such as wstouffer and wcorstouffer)

## Outputs

- Output text file (e.g. output.txt) with one combined p-value per input row for every requested method

## How to apply

Provide the input file (n rows and k columns of p-values or z-scores) and declare the statistic type via metacp's third argument (p-values/z-scores). For Stouffer-family methods, each p-value is converted to a Z-score, the Z-scores are summed, and the sum is transformed using the normal distribution; wstouffer divides by the square root of the sum of squared weights, while corstouffer and wcorstouffer use the sum of variances of all z-scores (adjusted by correlation coefficients) as the denominator for dependent tests. meanp takes the mean of the individual p-values and transforms it using a normal distribution (scaled by sqrt(12k)); logit sums the logit-transformed p-values and applies a t-distribution; cct transforms each p-value with the tangent function and evaluates the sum with the inverse Cauchy cumulative distribution function. Request the desired method keywords in the 7th+ arguments so metacp dispatches the correct quantile function per method.

## Related tools

- **metacp** (Python package invoked via 'python metacp.py' that implements the Stouffer-family, meanp, logit, and CCT p-value combination methods using normal, t, and Cauchy quantile functions) — github.com/pbagos/metacp
- **Python** (Runtime for metacp.py, called from the command line with input/output files, p-values/z-scores flag, and method keywords)
- **STATA** (Alternative implementation of metacp; see the metacp.hlp file within STATA)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values Yes correlation_matrix.txt weight_matrix.txt stouffer wstouffer cct meanp
```

## Evaluation signals

- The output file contains exactly one combined p-value per input row for every method keyword requested
- stouffer/wstouffer results correspond to a normal-distribution transformation of summed Z-scores, with wstouffer's denominator equal to the square root of the sum of squared weights
- cct outputs match an inverse-Cauchy-CDF transformation of tangent-transformed p-values and agree closely with reference tools (R2 between 0.9818 and 1.000 vs GWAR on 67 SNPs)
- Declaring 'z-scores' instead of 'p-values' in the third argument changes results consistently, confirming the transformation flag is honored
- Cross-check a subset of rows against poolr or metap, which implement a subset of the same methods

## Limitations

- If the correlation matrix argument is omitted for dependent-p-value methods, metacp calculates a default correlation matrix from the observations, which may not reflect the true correlation structure
- The correlation structure of test statistics can be hard to decipher in some multi-omics studies, and the optimal choice among dependent-test methods is highly problem dependent
- The various implemented methods are not all suitable for the same analysis

## Evidence
