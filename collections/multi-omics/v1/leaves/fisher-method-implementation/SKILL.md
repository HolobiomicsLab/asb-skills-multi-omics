---
name: fisher-method-implementation
description: Use when you have a text file of n rows x k columns of independent p-values
  (one variable per row, e.g. SNPs or genes across tests/traits) and need a combined
  per-row p-value via Fisher's method.
when_to_use_negative:
- The p-values to be combined are dependent (correlated) and no correction or correlation
  matrix is supplied — use dependent-p-value methods such as Brown's method or the
  Cauchy Combination Test instead
- The input file contains z-scores but you invoke Fisher as if the values were p-values
  without specifying 'z-scores' in the CLI argument
- You need to adjust for the effective number of tests under correlation (Cheverud-Nyholt,
  Li-Ji, Galwey, Gao) rather than standard Fisher combination
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: metacp
  role: Python/STATA command-line tool that implements Fisher's method (via 'fisher')
    among many other p-value combination methods; takes an n-rows x k-columns text
    file of p-values/z-scores and writes combined p-values to an output file
  repo: github.com/pbagos/metacp
- name: Python
  role: Runtime for the metacp.py script invoked from the terminal to run Fisher's
    combination
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:d3e59ff1c705f5a5f9165b6ad584b03bc7075ff5243ca71189dcff20148b639e
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# fisher-method-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implementation of Fisher's method for combining independent p-values, as provided by the metacp package (Python and STATA): the log-transformed p-values of each variable are summed and the combined p-value is obtained from a chi-squared distribution with 2k degrees of freedom.

## When to use

Use when you have a text file of n rows x k columns of independent p-values (one variable per row, e.g. SNPs or genes across tests/traits) and need a combined per-row p-value via Fisher's method.

## When NOT to use

- The p-values to be combined are dependent (correlated) and no correction or correlation matrix is supplied — use dependent-p-value methods such as Brown's method or the Cauchy Combination Test instead
- The input file contains z-scores but you invoke Fisher as if the values were p-values without specifying 'z-scores' in the CLI argument
- You need to adjust for the effective number of tests under correlation (Cheverud-Nyholt, Li-Ji, Galwey, Gao) rather than standard Fisher combination

## Inputs

- Text file of n rows and k columns, plus headers, containing p-values (or z-scores) to be combined; rows are variables such as SNPs/genes/probes, columns are tests/traits/SNPs
- CLI arguments: input file, output file, statistic type (p-values/z-scores), Yes/No meta-analysis flag, optional correlation matrix file, optional weight matrix file, and method name 'fisher'

## Outputs

- Text output file (e.g. output.txt) containing the Fisher combined p-value for each variable (row) in the input file

## How to apply

Load the input file where each row holds the statistics for one variable (SNP/gene/probe) across k columns; metacp also accepts z-scores and can perform a prior meta-analysis if the same SNP/gene appears in multiple rows (Yes/No flag). Compute the test statistic as the sum of the log-transformed p-values and apply a chi-squared distribution with 2k degrees of freedom, i.e. pc = 1 - F(-2*sum(ln p_i), 2k). Call metacp via the command line, passing the input file, output file, the statistic type (p-values/z-scores), the meta-analysis flag, optional correlation and weight matrix files, and 'fisher' as the method. Write the combined p-values to the user-specified output text file. Validate the implementation against closed-form reference values (e.g. scipy.stats chi2 survival function), requiring agreement within tolerance such as absolute error < 1e-8.

## Related tools

- **metacp** (Python/STATA command-line tool that implements Fisher's method (via 'fisher') among many other p-value combination methods; takes an n-rows x k-columns text file of p-values/z-scores and writes combined p-values to an output file) — github.com/pbagos/metacp
- **Python** (Runtime for the metacp.py script invoked from the terminal to run Fisher's combination)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values No fisher meanp bonferroni
```

## Evaluation signals

- For a small reference vector of independent p-values, the output matches the closed-form value pc = 1 - F(-2*sum(ln p_i), 2k) computed with a chi-squared CDF (e.g. scipy.stats chi2) with absolute error < 1e-8
- The output file contains one combined p-value per input row, with row identifiers preserved from the input text file
- Fisher runs on the loaded input file without error and all requested methods (e.g. fisher, meanp, bonferroni, wstouffer) produce values in [0, 1]
- With 'No' meta-analysis and no correlation matrix supplied, the program still completes, computing a default correlation matrix from the observations
- Equivalent independent-p-value methods available in metacp (Stouffer, invchi, logit) give mutually consistent orderings on the same rows

## Limitations

- The various implemented methods are not all suitable for the same analysis; Fisher's method assumes independence of the combined p-values
- If the correlation matrix argument is omitted, metacp calculates a default correlation matrix from the observations, which may not match the true dependence structure
- The optimal choice for combining dependent tests is highly problem dependent, and in some multi-omics studies the correlation structure of the test statistics is hard to decipher

## Evidence
