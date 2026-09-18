---
name: gamma-distributed-brown-yang-pvalue-extension
description: Use when combining k correlated p-values (e.g. SNPs or genes in a GWAS/transcriptomics
  row) where Fisher's chi-square sum must be corrected for correlation, and a user-supplied
  correlation matrix is available.
when_to_use_negative:
- When p-values are independent — use Fisher, Stouffer, Edgington or CCT instead;
  a correlation-adjusted method wastes the correlation input and need not improve
  power
- When the correlation structure of the test statistics is hard to decipher (e.g.
  some multi-omics studies) — the paper notes this makes dependent-test combination
  choices unreliable
- When the input statistics are z-scores and the user does not convert/declare them
  correctly via the 3rd CLI argument (p-values/z-scores flag)
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: metacp
  role: Python package whose 'yang' method implements the Brown-Yaw gamma extension
    given the p-value file, correlation matrix file and method name
  repo: https://github.com/pbagos/metacp
- name: Python
  role: Runtime/language for metacp.py; invoked from the command line as 'python metacp.py
    ...'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:05297b0b0420846723cf6837e4779a845637ba00b8839d3c3d20493c7a168f7a
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# gamma-distributed-brown-yang-pvalue-extension

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements Yang's Brown-Yang adjustment for combining dependent p-values, extending Brown's/EBM's chi-square mixture by a gamma-distributed approximation with a polynomial fit and bias correction for the correlation structure.

## When to use

Use when combining k correlated p-values (e.g. SNPs or genes in a GWAS/transcriptomics row) where Fisher's chi-square sum must be corrected for correlation, and a user-supplied correlation matrix is available.

## When NOT to use

- When p-values are independent — use Fisher, Stouffer, Edgington or CCT instead; a correlation-adjusted method wastes the correlation input and need not improve power
- When the correlation structure of the test statistics is hard to decipher (e.g. some multi-omics studies) — the paper notes this makes dependent-test combination choices unreliable
- When the input statistics are z-scores and the user does not convert/declare them correctly via the 3rd CLI argument (p-values/z-scores flag)

## Inputs

- Text file of p-values or z-scores, n rows x k columns with headers (e.g. SNP_p_values.txt)
- Text file containing the correlation matrix of the dependent tests (5th CLI argument)
- Method selector string 'yang' (Brown-Yang adjustment), optionally alongside 'ebm', 'kost', etc.

## Outputs

- Text file (e.g. output.txt) with one combined p-value per input row, each in [0,1]

## How to apply

Load the p-value vector from the input text file (n rows, k columns, e.g. SNP_p_values.txt) and the correlation matrix from the user-supplied text file passed as the 5th CLI argument of metacp.py. Transform the p-values into Fisher's statistic -2*sum(log p_i) and, instead of the naive chi-square_{2k} reference, evaluate the combined p as an upper-tail gamma probability p_c = 1 - Gamma(-2*sum(log p_i), v/2, 2*gamma), where the polynomial fit of the correlation structure determines the scaling and v/gamma are corrected for bias. This is metacp's 'yang' method, a Brown-Yang adjustment alongside 'ebm' (Effective Brown's Method with Psi ~ c*chi^2_{2f}) and 'kost' (EBM with a polynomial fit for correlation). Call the module API with method name 'yang' per row and write the combined p-values to output.txt; validate that each lies in [0,1] and that behavior agrees with the reference (paper reports R2 of 0.9818–1.000 against alternative implementations).

## Related tools

- **metacp** (Python package whose 'yang' method implements the Brown-Yaw gamma extension given the p-value file, correlation matrix file and method name) — https://github.com/pbagos/metacp
- **Python** (Runtime/language for metacp.py; invoked from the command line as 'python metacp.py ...')

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values Yes correlation_matrix.txt weight_matrix.txt yang
```

## Evaluation signals

- Every returned combined p-value lies in [0,1] for each input row
- Module output agrees with metacp reference behavior, consistent with the paper's reported R2 range of 0.9818–1.000 against alternative implementations
- Results differ from uncorrected Fisher's method when the correlation matrix has strong off-diagonal entries, and converge toward Fisher as correlations approach 0
- Rows are processed independently: one combined p per input row, column count k matching the correlation matrix dimension
- Omitting the correlation matrix argument triggers metacp's default correlation calculation without crashing

## Limitations

- The various methods implemented are not all suitable for the same analysis; the optimal choice for dependent tests is highly problem dependent
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher, undermining correlation-fit assumptions
- If the correlation matrix argument is omitted, metacp only calculates a default matrix 'according to the specifics of the requested method', which may not match the true dependence structure

## Evidence

- [abstract] metacp implements methods for combining independent p-values (Fisher's, Stouffer's, Edgington's) and dependent p-values (Brown's method, Cauchy Combination Test): [evidence span withheld — anchor status: not_found]
