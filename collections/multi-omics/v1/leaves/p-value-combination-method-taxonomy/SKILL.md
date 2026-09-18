---
name: p-value-combination-method-taxonomy
description: Use when an analysis requires choosing or cataloguing a method to combine
  p-values or z-scores (independent or correlated, e.g. GWAS SNPs), or when comparing
  method coverage across p-value combination packages.
when_to_use_negative:
- The input statistics are not p-values or z-scores (e.g. raw effect sizes or F-statistics)
  — metacp requires the 3rd argument to specify p-values/z-scores
- Only a single p-value per variable needs adjusting (plain multiple-testing correction
  with no combination across columns/tests) — a simple Bonferroni/FDR step suffices
  without the combination taxonomy
- The correlation structure of the test statistics is hard to decipher (e.g. some
  multi-omics studies), making the dependent-test methods' assumptions untenable
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: metacp
  role: Implements the full method taxonomy for combining independent and dependent
    p-values/z-scores; invoked from the command line with input/output files, p-values/z-scores
    flag, Yes/No meta-analysis flag, correlation and weight matrices, and one or more
    method keywords
  repo: https://github.com/pbagos/metacp
- name: Python
  role: Runtime for metacp.py; the CLI is invoked as 'python metacp.py' with the documented
    argument order
- name: STATA
  role: Alternative metacp implementation; see the metacp.hlp file within STATA for
    usage
- name: poolr
  role: Comparison package in the method-coverage matrix; implements a subset of the
    p-value combination methods
- name: metap
  role: Comparison package in the method-coverage matrix; implements a subset of the
    p-value combination methods
merged_aliases:
- p-value-combination-statistical-methods
merged_alias_records:
- alias: p-value-combination-statistical-methods
  slug: p-value-combination-statistical-methods
  cosine_score: 0.8267
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:819aaedc95319c9f9c3c02e293cf91f3462ededccdfb7c31843784cafd686a4e
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
  - STATA
  - metacp
  - metap
  - poolr
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# p-value-combination-method-taxonomy

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A taxonomy skill that enumerates and classifies the statistical methods for combining independent and dependent p-values implemented in metacp — from Fisher's, Stouffer's and Edgington's methods to Cauchy Combination Test, effective-number-of-tests Bonferroni variants and the empirical Brown's method family — and situates them against competing packages such as poolr and metap.

## When to use

Use when an analysis requires choosing or cataloguing a method to combine p-values or z-scores (independent or correlated, e.g. GWAS SNPs), or when comparing method coverage across p-value combination packages.

## When NOT to use

- The input statistics are not p-values or z-scores (e.g. raw effect sizes or F-statistics) — metacp requires the 3rd argument to specify p-values/z-scores
- Only a single p-value per variable needs adjusting (plain multiple-testing correction with no combination across columns/tests) — a simple Bonferroni/FDR step suffices without the combination taxonomy
- The correlation structure of the test statistics is hard to decipher (e.g. some multi-omics studies), making the dependent-test methods' assumptions untenable

## Inputs

- Canonical list of implemented combination methods from the article's methods table and README (logit, meanp, fisher, stouffer, wstouffer, invchi, binomial, cct, minp, mcm, cmc, bonferroni, adbonferroni, cn, lj, gao, galwey, ebm, kost, yang, corstouffer, wcorstouffer)
- Text file of n rows and k columns containing p-values or z-scores (e.g. SNP_p_values.txt)
- Optional correlation matrix text file for dependent p-values
- Optional weight matrix text file for weighted methods such as wstouffer

## Outputs

- Method taxonomy with one entry per method describing its statistical transformation (e.g. t-distribution on summed logits, inverse Cauchy CDF, eigenvalue-based effective number of tests)
- Binary methods_coverage_matrix.csv with a header row of package names (metacp, poolr, metap) and one 0/1 row per method
- One combined p-value per input row per requested method, written to the user-specified output text file

## How to apply

Enumerate the method families from the article's methods table and README: independent-test methods (logit, meanp, fisher, stouffer, wstouffer, invchi, binomial, bonferroni), dependent-test methods for correlated p-values (cct, minp, mcm, cmc, corstouffer, wcorstouffer), effective-number-of-tests Bonferroni variants driven by eigenvalues of the correlation matrix (adbonferroni, cn, lj, gao, galwey), and the empirical Brown's method family (ebm, kost, yang). For each method record its operating principle, e.g. fisher sums -2*ln(p) against a chi-squared distribution with 2k degrees of freedom, stouffer applies a normal CDF to summed z-scores, logit applies a t-distribution to summed logits, cct transforms each p-value with the tangent function and inverts the Cauchy CDF, and cn/lj/gao adjust for the effective number of tests using eigenvalues of the user-supplied correlation matrix. Classify each method as suited to independent versus dependent inputs, since the article notes that not all methods are suitable for the same analysis and the optimal choice for dependent tests is highly problem dependent. When benchmarking coverage, build a binary method-by-package matrix over metacp, poolr and metap, marking 1/0 per implemented method; metacp should mark 1 for every method in its implemented-methods table.

## Related tools

- **metacp** (Implements the full method taxonomy for combining independent and dependent p-values/z-scores; invoked from the command line with input/output files, p-values/z-scores flag, Yes/No meta-analysis flag, correlation and weight matrices, and one or more method keywords) — https://github.com/pbagos/metacp
- **Python** (Runtime for metacp.py; the CLI is invoked as 'python metacp.py' with the documented argument order)
- **STATA** (Alternative metacp implementation; see the metacp.hlp file within STATA for usage)
- **poolr** (Comparison package in the method-coverage matrix; implements a subset of the p-value combination methods)
- **metap** (Comparison package in the method-coverage matrix; implements a subset of the p-value combination methods)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values Yes correlation_matrix.txt weight_matrix.txt fisher meanp bonferroni wstouffer
```

## Evaluation signals

- methods_coverage_matrix.csv exists with a header row of package names and every method row carries a binary 0/1 value for each package column
- The metacp column marks value 1 for all methods listed in the article's implemented-methods table (logit through wcorstouffer)
- Each method entry correctly attributes its statistical transform (e.g. fisher → chi-squared with 2k df; logit → t-distribution; cct → inverse Cauchy CDF; cn/lj/gao → eigenvalues of the correlation matrix)
- Dispatch of method keywords matches the README's argument semantics: correlation matrix used for dependent methods, weight matrix for weighted methods such as wstouffer
- Running metacp with a requested method produces exactly one combined p-value per input row in the output text file

## Limitations

- The various methods implemented are not all suitable for the same analysis; for combination of dependent tests the optimal choice is highly problem dependent
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher, limiting the dependent-test methods
- If the correlation matrix argument is omitted, metacp calculates a default correlation matrix from the observations, which may not match the true dependence structure

## Evidence
