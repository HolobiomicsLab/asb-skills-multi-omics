---
name: closed-form-unit-testing
description: Use when reimplementing or wrapping p-value combination methods (e.g.
  as in metacp) and you need numerical proof that each method reproduces its statistical
  definition on small independent p-value test vectors.
when_to_use_negative:
- When combining dependent p-values requiring simulation or numerical integration
  (e.g. Brown's method, Cauchy Combination Test) where no simple closed form exists
  for the reference
- When the goal is running metacp itself on real GWAS data rather than verifying an
  implementation
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0091
tools:
- name: metacp
  role: Reference implementation of independent p-value combination methods (fisher,
    stouffer, wstouffer, meanp, logit, invchi, binomial, bonferroni, adbonferroni)
    whose outputs are compared against closed-form values
  repo: github.com/pbagos/metacp
- name: Python
  role: Language for implementing the methods and computing closed-form reference
    values with scipy.stats
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:bc1e524602a8dfe05c6bf3b37cf731034c1261d251134b6847b27c466d766171
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.0
n_claims: 6
---

# closed-form-unit-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validate implementations of p-value combination methods (fisher, stouffer, wstouffer, logitp, meanp, invchi, binomial, bonferroni, adbonferroni) by comparing outputs on small test vectors against closed-form reference values computed with scipy.stats chi2/norm/t/binom distribution functions, asserting agreement to ~1e-8 absolute error.

## When to use

Use when reimplementing or wrapping p-value combination methods (e.g. as in metacp) and you need numerical proof that each method reproduces its statistical definition on small independent p-value test vectors.

## When NOT to use

- When combining dependent p-values requiring simulation or numerical integration (e.g. Brown's method, Cauchy Combination Test) where no simple closed form exists for the reference
- When the goal is running metacp itself on real GWAS data rather than verifying an implementation

## Inputs

- Text file of n rows x k columns (plus headers) containing p-values or z-scores, one variable per row
- Small hand-computable test vectors of independent p-values
- Reference values from scipy.stats distribution functions (chi2, norm, t, binom survival/CDF functions)

## Outputs

- Per-method combined p-values for each test vector
- Pass/fail assertions with absolute error < 1e-8 against closed-form references
- Confirmation that all methods run on the loaded input file without error

## How to apply

Load a small text file of independent p-values in the n-rows x k-columns format metacp accepts (each row is one variable combined across columns). Implement each method from its closed form: Fisher as pc = 1 - F(-2*sum(ln p_i), 2k); inverse chi-squared as pc = 1 - F(sum(F^{-1}(1-p_i,1)), k); Stouffer as pc = 1 - Phi(sum(z_i)/sqrt(k)) with z_i = Phi^{-1}(1-p_i); weighted Stouffer dividing by sqrt(sum(w_i^2)); meanp as pc = 1 - Phi((0.5 - mean(p))*sqrt(12k)); logit via the t-distribution with 2k degrees of freedom; Lancaster via inverse chi-squared quantiles with chosen n_i; binomial via Binomial(k, alpha) on the count of rejections; Bonferroni as min(1, min(p_i)*k) and the adjusted variant replacing k with the effective number of tests m (Cheverud-Nyholt, Li-Ji, Galwey, Gao). Compute reference values with scipy.stats survival functions (chi2, norm, t, binom) and assert absolute error < 1e-8 for each method. The module passes when every method matches its closed-form value on the reference vectors and all methods run on the full input file without error.

## Related tools

- **metacp** (Reference implementation of independent p-value combination methods (fisher, stouffer, wstouffer, meanp, logit, invchi, binomial, bonferroni, adbonferroni) whose outputs are compared against closed-form values) — github.com/pbagos/metacp
- **Python** (Language for implementing the methods and computing closed-form reference values with scipy.stats)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values No fisher meanp bonferroni
```

## Evaluation signals

- Each implemented method's output for the reference test vectors matches the closed-form value (scipy.stats chi2/norm/t/binom survival functions) with absolute error < 1e-8
- Fisher output equals 1 - F(-2*sum(ln p_i), 2k) and inverse chi-squared equals 1 - F(sum(F^{-1}(1-p_i,1)), k) exactly to tolerance
- Stouffer, weighted Stouffer, and meanp reproduce 1 - Phi(sum(z_i)/sqrt(k)), 1 - Phi(sum(w_i z_i)/sqrt(sum(w_i^2))), and 1 - Phi((0.5 - mean(p))*sqrt(12k)) respectively
- Bonferroni output equals min(1, min(p_i)*k), and adbonferroni replaces k with the effective number of tests m (Cheverud-Nyholt, Li-Ji, Galwey, Gao)
- All methods run on the loaded n-rows x k-columns input file without error

## Limitations

- The various implemented methods are not all suitable for the same analysis; closed-form testing only validates correct implementation, not statistical appropriateness
- For combination of dependent tests the optimal choice is highly problem dependent, and closed-form unit tests cover only independent-p-value methods
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher, limiting applicability of correlation-adjusted Bonferroni variants

## Evidence

- [readme] The program takes as input a text file of n rows and k columns, plus the headers, which contain the p-values or z-scores to be combined.: "The program takes as input a text file of n rows and k columns, plus the headers, which contain the p-values or z-scores to be combined."
