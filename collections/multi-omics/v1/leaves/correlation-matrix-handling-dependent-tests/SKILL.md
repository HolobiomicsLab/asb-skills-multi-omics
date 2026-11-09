---
name: correlation-matrix-handling-dependent-tests
description: Use when combining p-values or z-scores whose tests are not independent
  and metacp therefore needs a user-supplied correlation matrix, a weight matrix,
  or its default correlation computed from the observations.
when_to_use_negative:
- The tests being combined are independent and simple methods such as fisher or stouffer
  suffice, so no correlation matrix is needed
- The correlation structure of the test statistics is hard to decipher, as can occur
  in some multi-omics studies, making correlation-adjusted choices unreliable
- The input statistics are not p-values or z-scores in a tabulated n-rows by k-columns
  text file
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: metacp
  role: Python/STATA tool whose 5th and 6th command-line arguments accept the correlation
    and weight matrices and whose dependent-test methods consume them
  repo: github.com/pbagos/metacp
- name: Python
  role: Runtime for invoking metacp.py from the command line with the correlation/weight
    matrix arguments
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:580545381293e1d03a55e63cd45d5c72b5cbe0f0fc2ea802bb453342dd9d7a0a
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
  - poolr
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# correlation-matrix-handling-dependent-tests

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Supplies or defaults the correlation structure required by metacp's dependent p-value combination methods (e.g. corstouffer, wcorstouffer, adbonferroni, cn, lj, gao, galwey, ebm, kost, yang, cct, mcm, cmc, minp) when combining p-values or z-scores from correlated tests.

## When to use

Use when combining p-values or z-scores whose tests are not independent and metacp therefore needs a user-supplied correlation matrix, a weight matrix, or its default correlation computed from the observations.

## When NOT to use

- The tests being combined are independent and simple methods such as fisher or stouffer suffice, so no correlation matrix is needed
- The correlation structure of the test statistics is hard to decipher, as can occur in some multi-omics studies, making correlation-adjusted choices unreliable
- The input statistics are not p-values or z-scores in a tabulated n-rows by k-columns text file

## Inputs

- Text file of n rows and k columns containing p-values or z-scores (e.g. SNP_p_values.txt)
- Text file with a correlation matrix for dependent p-values (optional; computed by default if omitted)
- Text file with a weight matrix for weighted methods such as wstouffer/wcorstouffer
- Command-line flags: input file, output file, p-values/z-scores, Yes/No meta-analysis flag

## Outputs

- Output text file with one combined p-value per input row for each requested dependent-test method

## How to apply

Pass a text file with the correlation matrix as the 5th argument of the metacp command line and, for weighted methods such as wstouffer, a weight matrix as the 6th argument; if the correlation matrix argument is omitted, metacp computes a default correlation matrix from the observations according to the specifics of the requested method. Choose methods that explicitly adjust for correlation: corstouffer and wcorstouffer divide the summed z-scores by the appropriate variance denominator, adbonferroni adjusts the minimum p-value by the maximum correlation, cn/lj/gao/galwey use eigenvalues of the correlation matrix to derive the effective number of tests, and ebm/kost/yang use the covariance matrix (with polynomial fits in kost and yang). Dependent-test methods cct, minp, mcm and cmc also rely on the correlation structure to combine the p-values. If the same SNP/gene appears in multiple rows, decide whether the Yes/No meta-analysis flag should aggregate duplicates before combining columns.

## Related tools

- **metacp** (Python/STATA tool whose 5th and 6th command-line arguments accept the correlation and weight matrices and whose dependent-test methods consume them) — github.com/pbagos/metacp
- **Python** (Runtime for invoking metacp.py from the command line with the correlation/weight matrix arguments)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values Yes correlation_matrix.txt weight_matrix.txt fisher meanp bonferroni wstouffer
```

## Evaluation signals

- For shared dependent methods (CCT, MinP, MCM, CMC), combined p-values should be numerically identical to those from poolr and the Robust tests package on the same input
- Omitting the 5th argument must not error: metacp should compute a default correlation matrix from the observations specific to the requested method
- Output file must contain one combined p-value per input row (n rows) for every requested method
- corstouffer/wcorstouffer results should differ from plain stouffer when correlations are non-zero, reflecting the variance/correlation denominator
- R2 values against alternative tools (e.g. GWAR MAX/MIN2 on 67 SNPs) are expected to range from 0.9818 to 1.000

## Limitations

- If the correlation matrix argument is omitted, metacp computes a default correlation matrix from the observations, which may not reflect the true correlation structure
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher
- For combination of dependent tests the optimal choice of method is highly problem dependent; not all implemented methods are suitable for the same analysis

## Evidence

- [other] For every shared combination method, the combined p-values computed by metacp are identical to those computed by the other packages: [evidence span withheld — anchor status: not_found]
