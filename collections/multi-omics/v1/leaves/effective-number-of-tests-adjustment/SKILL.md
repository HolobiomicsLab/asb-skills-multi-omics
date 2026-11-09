---
name: effective-number-of-tests-adjustment
description: Use when combining dependent p-values from correlated tests (e.g., SNPs
  in LD, correlated gene probes) and Bonferroni with the raw number of tests would
  over-correct; a correlation matrix for the test statistics is available or estimable.
when_to_use_negative:
- P-values are independent — plain bonferroni, fisher, stouffer or the independent-p-value
  methods suffice, and an effective-number adjustment would needlessly estimate correlations
- The correlation structure of the test statistics is hard to decipher (e.g. some
  multi-omics studies), making eigenvalue- or covariance-based adjustment unreliable
- The optimal choice for dependent tests is highly problem-dependent, so results from
  a single adjustment method should not be treated as definitive
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: metacp
  role: Python/STATA command-line package that implements the effective-number-of-tests
    adjustments (adbonferroni, cn, lj, gao, galwey) and covariance-based adjustments
    (ebm, kost, yang) from a correlation matrix
  repo: github.com/pbagos/metacp
- name: Python
  role: Runtime for the metacp.py command-line script that reads the p-value input,
    optional correlation/weight matrix files, and method flags
- name: STATA
  role: Alternative implementation of metacp (see the metacp.hlp file within STATA)
    providing the same adjustment methods
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:a2c0693439f01618656defbb09ef88c9c067f577c56bb2b2f41adf16db2987b1
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
  - poolr
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# effective-number-of-tests-adjustment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Adjust the number of independent tests (Bonferroni-style corrections or Stouffer variance) using the correlation structure — via eigenvalue-based methods (Cheverud-Nyholt, Li-Ji, Gao, Galwey) or covariance/polynomial methods (EBM, Kost, Brown-Yang) — as implemented in metacp.

## When to use

Use when combining dependent p-values from correlated tests (e.g., SNPs in LD, correlated gene probes) and Bonferroni with the raw number of tests would over-correct; a correlation matrix for the test statistics is available or estimable.

## When NOT to use

- P-values are independent — plain bonferroni, fisher, stouffer or the independent-p-value methods suffice, and an effective-number adjustment would needlessly estimate correlations
- The correlation structure of the test statistics is hard to decipher (e.g. some multi-omics studies), making eigenvalue- or covariance-based adjustment unreliable
- The optimal choice for dependent tests is highly problem-dependent, so results from a single adjustment method should not be treated as definitive

## Inputs

- Text file of n rows x k columns (plus headers) containing p-values or z-scores, one variable (e.g. SNP/gene) per row
- Optional text file with the correlation matrix of the tests (for dependent p-values)
- Optional text file with the weight matrix (for weighted methods)
- Command-line method flags selecting the adjustment: adbonferroni, cn, lj, gao, galwey, ebm, kost, yang, corstouffer, wcorstouffer

## Outputs

- Text output file containing the combined p-value per variable after effective-number-of-tests adjustment for each requested method

## How to apply

Run metacp via 'python metacp.py <input file> <output file> p-values <Yes/No> <correlation matrix file> <weight matrix file>' followed by the adjustment method(s): bonferroni and adbonferroni adjust the minimum p-value by the number of tests (adbonferroni by the maximum correlation), cn (Cheverud-Nyholt), lj (Li-Ji) and gao (threshold cumulative eigenvalue sum) derive the effective number of tests m from the eigenvalues of the correlation matrix, galwey uses the ratio of the sum of squared eigenvalues to the sum of eigenvalues, and ebm/kost/yang (Effective Brownian Motion and its Kost and Brown-Yang adjustments) account for the covariance structure, with kost incorporating a polynomial fit. If no correlation matrix file is supplied, metacp calculates one by default from the observations according to the specifics of the requested method. The corrected combined p-values are written to the user-specified output text file. Rationale: replacing the raw test count k with the effective number m avoids the conservative bias of Bonferroni when tests are correlated, as in the 67-SNP type-2 diabetes GWAS re-analysis.

## Related tools

- **metacp** (Python/STATA command-line package that implements the effective-number-of-tests adjustments (adbonferroni, cn, lj, gao, galwey) and covariance-based adjustments (ebm, kost, yang) from a correlation matrix) — github.com/pbagos/metacp
- **Python** (Runtime for the metacp.py command-line script that reads the p-value input, optional correlation/weight matrix files, and method flags)
- **STATA** (Alternative implementation of metacp (see the metacp.hlp file within STATA) providing the same adjustment methods)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values Yes correlation_matrix.txt weight_matrix.txt adbonferroni galwey kost
```

## Evaluation signals

- For independent tests (identity correlation matrix), adjusted methods should converge toward the plain Bonferroni result pc = min(1, min(p_i)*k)
- Effective number of tests m must satisfy 1 <= m <= k; eigenvalue-based cn/lj/gao/galwey results should reflect this bound
- Running with and without a supplied correlation_matrix.txt should succeed — when omitted, metacp must calculate a default correlation matrix from the observations
- Output file must contain one combined p-value per input row (variable) for every requested method flag, in the same row order as the input
- Cross-package check: combined p-values should match equivalent implementations in poolr or the Robust tests package for shared methods

## Limitations

- The various implemented methods are not all suitable for the same analysis, and for dependent tests the optimal choice is highly problem dependent
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher, weakening eigenvalue- and covariance-based adjustments
- When no correlation matrix is provided, metacp calculates one by default from the observations 'according to the specifics of the requested method', which may not match the user's intended dependency structure

## Evidence
