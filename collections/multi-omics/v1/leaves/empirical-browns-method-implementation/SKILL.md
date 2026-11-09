---
name: empirical-browns-method-implementation
description: Use when p-values or z-scores to be combined are dependent (e.g. SNPs
  within a gene, correlated tests) and a correlation matrix is available or must be
  estimated, requiring methods like kost, cn, lj, gao, galwey, ebm, yang, corstouffer
  or wcorstouffer.
when_to_use_negative:
- P-values are independent — use metacp's independent methods (fisher, stouffer, edgington-style,
  cct, bonferroni) instead; correlation adjustments are unnecessary and the discussion
  notes the various methods are not all suitable for the same analysis
- The correlation structure of the test statistics is hard to decipher (e.g. some
  multi-omics studies), where the article warns the optimal choice among dependent-test
  methods is highly problem dependent
- Input statistics are raw test statistics rather than p-values or z-scores — metacp
  expects the 3rd argument to specify 'p-values' or 'z-scores'
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0092
tools:
- name: metacp
  role: Reference implementation providing the correlation-aware combination methods
    (kost, cn, lj, gao, galwey, ebm, yang, corstouffer, wcorstouffer) invoked via
    metacp.py with input file, output file, correlation matrix and weight matrix arguments
  repo: github.com/pbagos/metacp
- name: Python
  role: Implementation language for the runnable module reproducing metacp's methods;
    called as 'python metacp.py ...'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:a22e1b76907ad00fc2572f4d25aec7f7afa734a652c26103c6d9499f97bbd9de
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# empirical-browns-method-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implement metacp's correlation-aware p-value combination methods (EBM, Kost, Yang, Cheverud-Nyholt, Li-Ji, Gao, Galwey, correlated Stouffer and weighted correlated Stouffer), which adjust Fisher/Bonferroni/Stouffer statistics for dependence among tests using a user-supplied correlation matrix.

## When to use

Use when p-values or z-scores to be combined are dependent (e.g. SNPs within a gene, correlated tests) and a correlation matrix is available or must be estimated, requiring methods like kost, cn, lj, gao, galwey, ebm, yang, corstouffer or wcorstouffer.

## When NOT to use

- P-values are independent — use metacp's independent methods (fisher, stouffer, edgington-style, cct, bonferroni) instead; correlation adjustments are unnecessary and the discussion notes the various methods are not all suitable for the same analysis
- The correlation structure of the test statistics is hard to decipher (e.g. some multi-omics studies), where the article warns the optimal choice among dependent-test methods is highly problem dependent
- Input statistics are raw test statistics rather than p-values or z-scores — metacp expects the 3rd argument to specify 'p-values' or 'z-scores'

## Inputs

- Text file of p-values or z-scores (n rows x k columns, one row per SNP/gene/probe)
- Correlation matrix text file (5th CLI argument; required for dependent-test methods)
- Weight matrix text file (6th CLI argument, for wcorstouffer)
- Method name(s): kost, cn, lj, gao, galwey, ebm, yang, corstouffer, wcorstouffer

## Outputs

- Output text file (e.g. output.txt) with one combined p-value per row in [0,1]

## How to apply

Load the p-value vector (n rows x k columns text file such as SNP_p_values.txt) and the correlation matrix from the 5th CLI argument; if omitted, metacp calculates a correlation matrix from the observations by default. Implement EBM by extending Fisher's method through an up-scaled distribution Psi ~ c*chi^2_{2f}, Kost's variant with a polynomial fit for the correlation structure, and Yang's Brown-Yang extension p_c = 1 - Gamma(-2*sum(log p_i), v/2, 2*gamma). Implement the effective-number-of-tests corrections (Cheverud-Nyholt, Li-Ji, Gao, Galwey) from the eigenvalues of the correlation matrix, applied via the adjusted Bonferroni formula p_c = min(1, min(p_i) * m). Implement correlated Stouffer as p_c = 1 - Phi(sum(z_i)/sqrt(k + sum_i sum_j r_ij)) and its weighted variant p_c = 1 - Phi(sum(w_i z_i)/sqrt(sum_i sum_j w_i w_j r_ij)) using the weight matrix from the 6th CLI argument. Expose a module API taking the correlation matrix, p-value vector, and method name, returning one combined p-value per row written to the output file.

## Related tools

- **metacp** (Reference implementation providing the correlation-aware combination methods (kost, cn, lj, gao, galwey, ebm, yang, corstouffer, wcorstouffer) invoked via metacp.py with input file, output file, correlation matrix and weight matrix arguments) — github.com/pbagos/metacp
- **Python** (Implementation language for the runnable module reproducing metacp's methods; called as 'python metacp.py ...')

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values Yes correlation_matrix.txt weight_matrix.txt kost ebm corstouffer wcorstouffer
```

## Evaluation signals

- Every returned combined p-value lies in [0,1]
- Module outputs agree with metacp reference behavior, consistent with the paper's reported R2 agreement range of 0.9818 to 1.000 against alternative implementations (GWAR MAX/MIN2 on the 67 type-2 diabetes GWAS SNPs)
- Effective-number-of-tests corrections (cn, lj, gao, galwey) correctly derive the adjusted number of tests m from the eigenvalues of the supplied correlation matrix
- Weighted correlated Stouffer denominator matches sqrt(sum_i sum_j w_i w_j r_ij) when a weight matrix is provided
- Combined p-values are computed without the complex numerical integrations required by GWAR

## Limitations

- The various implemented methods are not all suitable for the same analysis; for dependent tests the optimal choice is highly problem dependent
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher
- No changelog was found for the repository, so version-specific behavior differences cannot be traced

## Evidence
