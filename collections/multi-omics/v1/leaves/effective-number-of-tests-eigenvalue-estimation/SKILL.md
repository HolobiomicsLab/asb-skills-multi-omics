---
name: effective-number-of-tests-eigenvalue-estimation
description: Use when combining dependent p-values where a correlation matrix is available
  and you want a Bonferroni-style correction that replaces the raw number of tests
  with an eigenvalue-based effective number of tests (cn, lj, gao, galwey methods).
when_to_use_negative:
- P-values are independent — use plain Bonferroni instead of correlation-adjusted
  effective-number-of-tests methods
- You need methods that model the full correlation structure (Kost, EBM, Yang, correlated
  Stouffer) rather than a Bonferroni-style minimum-p adjustment
- No correlation matrix is available and the default correlation matrix computed from
  the observations is not appropriate for the requested method
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3517
tools:
- name: metacp
  role: Python/STATA package that computes the eigenvalue-based effective number of
    tests (cn, lj, gao, galwey methods) and writes the combined p-values to the output
    file
  repo: github.com/pbagos/metacp
- name: Python
  role: Runtime environment for metacp.py, invoked from the command line with input
    file, output file, p-values/z-scores flag, Yes/No meta-analysis flag, correlation
    matrix and method names
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:ee55cdcf2d169808ef66e1478678fd7c48db92f53769ca1a15b279b39ab2343e
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
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# effective-number-of-tests-eigenvalue-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates the effective number of (dependent) tests from the eigenvalues of a correlation matrix and applies it as an adjusted Bonferroni correction, p_c = min(1, min(p_i) * m_eff). In metacp this is realized by four estimators: Cheverud-Nyholt (cn), Li-Ji (lj), Gao (gao) and Galwey (galwey).

## When to use

Use when combining dependent p-values where a correlation matrix is available and you want a Bonferroni-style correction that replaces the raw number of tests with an eigenvalue-based effective number of tests (cn, lj, gao, galwey methods).

## When NOT to use

- P-values are independent — use plain Bonferroni instead of correlation-adjusted effective-number-of-tests methods
- You need methods that model the full correlation structure (Kost, EBM, Yang, correlated Stouffer) rather than a Bonferroni-style minimum-p adjustment
- No correlation matrix is available and the default correlation matrix computed from the observations is not appropriate for the requested method

## Inputs

- Input text file of n rows and k columns containing p-values (or z-scores) to be combined, plus headers (e.g. SNP_p_values.txt)
- Correlation matrix text file for the dependent p-values (5th CLI argument, e.g. correlation_matrix.txt)
- Optional weight matrix text file (6th CLI argument; not required by cn/lj/gao/galwey themselves)
- User-selected method names: cn, lj, gao, galwey

## Outputs

- Combined p-value per row, written to the user-provided output text file (e.g. output.txt), each value lying in [0,1]

## How to apply

Load the p-value vector from the input text file (n rows, k columns of p-values or z-scores) and the user-supplied correlation matrix text file (the 5th CLI argument of metacp.py). Compute the eigenvalues of the correlation matrix, then derive the effective number of tests with one of the four estimators: Cheverud-Nyholt and Li-Ji from the eigenvalues of the correlation matrix, Gao from a threshold cumulative eigenvalue sum, and Galwey from the ratio of the sum of squared eigenvalues to the sum of eigenvalues. Apply the adjusted Bonferroni formula p_c = min(1, min(p_i) * m_eff) to produce one combined p-value per row. Select estimators by name (cn, lj, gao, galwey) among the methods passed after the weight-matrix argument; if the correlation matrix argument is omitted, metacp calculates a default correlation matrix from the observations according to the specifics of the requested method.

## Related tools

- **metacp** (Python/STATA package that computes the eigenvalue-based effective number of tests (cn, lj, gao, galwey methods) and writes the combined p-values to the output file) — github.com/pbagos/metacp
- **Python** (Runtime environment for metacp.py, invoked from the command line with input file, output file, p-values/z-scores flag, Yes/No meta-analysis flag, correlation matrix and method names)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values No correlation_matrix.txt cn lj gao galwey
```

## Evaluation signals

- Each returned combined p-value lies in [0,1]
- Combined p-values from cn/lj/gao/galwey agree with the metacp reference implementation, consistent with the paper's reported R2 agreement range of 0.9818 to 1.000 against alternative implementations (GWAR)
- For a perfectly correlated set of tests the effective number of tests approaches 1, so p_c approaches min(p_i); for orthogonal tests it approaches the number of tests k
- Eigenvalues of the supplied correlation matrix are non-negative (positive semidefinite) before computing the effective number of tests
- Output file contains one combined p-value per input row, matching the row order of SNP_p_values.txt

## Limitations

- These methods require a correlation matrix; if the 5th argument is omitted, metacp computes a default correlation matrix from the observations, which may not match the true dependence structure
- The various methods implemented are not all suitable for the same analysis; for dependent tests the optimal choice is highly problem dependent
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher, undermining eigenvalue-based estimation

## Evidence
