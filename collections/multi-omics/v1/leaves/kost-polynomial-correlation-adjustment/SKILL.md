---
name: kost-polynomial-correlation-adjustment
description: Use when combining k dependent p-values (or z-scores) per row of an n
  x k input file where a correlation matrix for the tests is available or can be estimated,
  and a correlation-aware alternative to Fisher's or Bonferroni's method is needed.
when_to_use_negative:
- The p-values to combine are independent — independent-test methods (fisher, stouffer,
  edgington-style) suffice and no correlation matrix is needed
- The correlation structure of the test statistics is hard to decipher, as in some
  multi-omics studies — the optimal choice among dependent-test methods is highly
  problem dependent and an ill-estimated correlation matrix will mislead kost's adjustment
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: metacp
  role: Python/STATA package exposing kost as a selectable combination method invoked
    from the command line with input p-values, correlation matrix and output file
  repo: https://github.com/pbagos/metacp
- name: Python
  role: Implementation language in which metacp.py runs the kost combination on loaded
    p-value and correlation-matrix files
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:600c831ef8a333ce71f528f42c73321c8c30f0523595ccdb4eae8ac2854e76a1
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# kost-polynomial-correlation-adjustment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Kost's method, implemented in metacp, combines dependent p-values by extending Empirical Brown's Method with a polynomial fit for the correlation structure, yielding a combined p-value per row (e.g. SNP or gene) from a vector of p-values and a correlation matrix.

## When to use

Use when combining k dependent p-values (or z-scores) per row of an n x k input file where a correlation matrix for the tests is available or can be estimated, and a correlation-aware alternative to Fisher's or Bonferroni's method is needed.

## When NOT to use

- The p-values to combine are independent — independent-test methods (fisher, stouffer, edgington-style) suffice and no correlation matrix is needed
- The correlation structure of the test statistics is hard to decipher, as in some multi-omics studies — the optimal choice among dependent-test methods is highly problem dependent and an ill-estimated correlation matrix will mislead kost's adjustment

## Inputs

- Text file of n rows x k columns of p-values or z-scores (e.g. SNP_p_values.txt)
- Correlation matrix text file for the dependent tests (5th CLI argument; if omitted, metacp computes one by default from the observations)

## Outputs

- Text file (e.g. output.txt) containing one combined p-value per input row, computed with the kost method

## How to apply

Load the p-value vector (e.g. from SNP_p_values.txt) and the user-supplied correlation matrix text file (the 5th CLI argument of metacp.py). Convert p-values to z-scores and apply Kost's adjustment to the Empirical Brown's Method framework, where the combined statistic follows an up-scaled chi-square distribution (Psi ~ c*chi^2_{2f}) with the correlation structure captured by a polynomial fit of the pairwise correlations rather than by direct covariance integration. Call the method with the method name 'kost' among the accepted method list (kost, cn, lj, gao, galwey, ebm, yang, corstouffer, wcorstouffer) and write the resulting combined p-value per row to the output file. Validate that each returned combined p-value lies in [0,1] and, where possible, that outputs agree with the metacp reference behavior (paper-reported R2 agreement of 0.9818 to 1.000 against alternative implementations).

## Related tools

- **metacp** (Python/STATA package exposing kost as a selectable combination method invoked from the command line with input p-values, correlation matrix and output file) — https://github.com/pbagos/metacp
- **Python** (Implementation language in which metacp.py runs the kost combination on loaded p-value and correlation-matrix files)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values No correlation_matrix.txt kost
```

## Evaluation signals

- Every returned combined p-value lies within [0,1]
- Module outputs agree with the metacp reference implementation, consistent with the paper's reported R2 agreement range of 0.9818 to 1.000 against alternative implementations
- Combined p-values are produced for every row of the input file and written to the user-specified output file
- Providing no correlation matrix file causes metacp to compute a default correlation matrix from the observations instead of failing
- Running with an independent set of p-values (identity correlation) gives a sane boundary result comparable to unadjusted combination

## Limitations

- The various implemented methods are not all suitable for the same analysis; for dependent tests the optimal choice is highly problem dependent
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher, undermining correlation-based adjustments like kost
- No changelog was found for the metacp repository

## Evidence

- [methods] Implement Empirical Brown's Method (EBM) extending Fisher's method via the up-scaled distribution Psi ~ c*chi^2_{2f}, Kost's EBM variant with a polynomial fit: [evidence span withheld — anchor status: not_found]
