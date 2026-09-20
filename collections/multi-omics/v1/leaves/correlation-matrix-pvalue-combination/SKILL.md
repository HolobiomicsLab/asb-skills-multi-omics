---
name: correlation-matrix-pvalue-combination
description: Use when input is a text file of n rows × k columns of p-values or z-scores
  whose tests are not independent and a correlation matrix is available or can be
  estimated, and per-row combined p-values are needed.
when_to_use_negative:
- The p-values being combined are independent — use the independent methods (fisher,
  stouffer, edgington-style) instead of correlation-adjusted ones
- The correlation structure of the test statistics is hard to decipher, as can occur
  in some multi-omics studies — the article notes this makes method choice complicated
- Only the MinP/Cauchy family (cct, minp, mcm, cmc) or plain bonferroni is required
  and no correlation matrix is relevant
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: metacp
  role: Reference Python implementation invoked via metacp.py with input file, output
    file, p-values/z-scores flag, Yes/No meta-analysis flag, correlation matrix file,
    weight matrix file and method names; also provides the STATA version (metacp.hlp)
  repo: https://github.com/pbagos/metacp
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:a3ff2f50b04fed2815b8d45643a99f1e69aa4ed723df2c1668a11728eed1b0fb
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# correlation-matrix-pvalue-combination

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Combine dependent p-values (or z-scores) by supplying a correlation matrix and applying correlation-aware methods — Kost, Cheverud-Nyholt, Li-Ji, Gao, Galwey, EBM, Yang, correlated Stouffer and weighted correlated Stouffer — as implemented in metacp.

## When to use

Use when input is a text file of n rows × k columns of p-values or z-scores whose tests are not independent and a correlation matrix is available or can be estimated, and per-row combined p-values are needed.

## When NOT to use

- The p-values being combined are independent — use the independent methods (fisher, stouffer, edgington-style) instead of correlation-adjusted ones
- The correlation structure of the test statistics is hard to decipher, as can occur in some multi-omics studies — the article notes this makes method choice complicated
- Only the MinP/Cauchy family (cct, minp, mcm, cmc) or plain bonferroni is required and no correlation matrix is relevant

## Inputs

- Text file of n rows and k columns of p-values or z-scores (e.g. SNP_p_values.txt) with headers
- Text file with the correlation matrix for dependent p-values (5th CLI argument; optional — computed by default from observations if omitted)
- Text file with the weight matrix for weighted methods such as wcorstouffer (6th CLI argument)
- One or more selected method names (kost, cn, lj, gao, galwey, ebm, yang, corstouffer, wcorstouffer)

## Outputs

- Text file (e.g. output.txt) with one combined p-value per input row for each requested method
- Per-row combined p-values in [0,1] from each correlation-aware method

## How to apply

Load the p-value vector (e.g. SNP_p_values.txt) and the user-supplied correlation matrix (5th CLI argument of metacp.py); if the correlation matrix is omitted, metacp computes one by default from the observations according to the requested method. Implement effective-number-of-tests corrections (Cheverud-Nyholt cn, Li-Ji lj, Gao gao, Galwey galwey) from the eigenvalues of the correlation matrix, applied through the adjusted Bonferroni formula p_c = min(1, min(p_i) * m). Implement Empirical Brown's Method (ebm) extending Fisher's method via the up-scaled distribution Psi ~ c*chi^2_{2f}, Kost's EBM variant with a polynomial fit for the correlation structure, and Yang's extension p_c = 1 - Gamma(-2*sum(log p_i), v/2, 2*gamma). Implement Stouffer's method for correlated tests p_c = 1 - Phi(sum(z_i)/sqrt(k + sum_i sum_j r_ij)) and its weighted variant p_c = 1 - Phi(sum(w_i z_i)/sqrt(sum_i sum_j w_i w_j r_ij)), exposing an API taking a correlation matrix, a p-value vector and a method name (kost, cn, lj, gao, galwey, ebm, yang, corstouffer, wcorstouffer) and returning one combined p-value per row.

## Related tools

- **metacp** (Reference Python implementation invoked via metacp.py with input file, output file, p-values/z-scores flag, Yes/No meta-analysis flag, correlation matrix file, weight matrix file and method names; also provides the STATA version (metacp.hlp)) — https://github.com/pbagos/metacp

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values Yes correlation_matrix.txt weight_matrix.txt kost cn lj gao galwey ebm yang corstouffer wcorstouffer
```

## Evaluation signals

- Each returned combined p-value lies in [0,1] for every method and row
- Module outputs agree with metacp reference behavior, consistent with the paper's reported R2 agreement range of 0.9818 to 1.000 against alternative implementations
- Effective-number-of-tests methods (cn, lj, gao, galwey) reduce the Bonferroni multiplier m relative to the raw number of tests when the correlation matrix has correlated entries
- wcorstouffer output changes appropriately when the weight matrix file is supplied vs omitted
- Omitting the 5th argument produces a default correlation matrix computed from the observations without error

## Limitations

- The various implemented methods are not all suitable for the same analysis; for dependent tests the optimal choice is highly problem dependent
- In some multi-omics studies the correlation structure of the test statistics is hard to decipher
- No changelog was found for the repository
- metacp computes combined p-values without the numerical integrations required by GWAR — a benefit, but agreement with integration-based tools is not exactly 1.0 (R2 ranges 0.9818–1.000)

## Evidence
