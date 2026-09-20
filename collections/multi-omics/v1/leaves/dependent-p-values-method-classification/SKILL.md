---
name: dependent-p-values-method-classification
description: Use when choosing or documenting which p-value combination method fits
  a set of independent or correlated test statistics (e.g. SNPs from a GWAS, gene/probe
  sets from transcriptomics) and whether correlation or weight matrices must be supplied.
when_to_use_negative:
- The p-values are independent and the analysis goal is comparing packages rather
  than choosing a dependent-test method — the independent-only class (fisher, stouffer,
  meanp, etc.) suffices and no correlation matrix is needed.
- The correlation structure of the test statistics is hard to decipher (e.g. some
  multi-omics studies) — the article notes the optimal choice of dependent-test method
  is highly problem dependent and some methods require a reliable correlation matrix.
- The input is effect sizes or raw summary statistics rather than p-values or z-scores
  — metacp requires the 3rd argument to specify p-values or z-scores and cannot combine
  other statistic types directly.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0622
tools:
- name: metacp
  role: Implements all 22 documented combination methods for independent and dependent
    p-values; invoked per-method on the command line
  repo: https://github.com/pbagos/metacp
- name: poolr
  role: Comparison package with a smaller subset of combination methods, used as a
    benchmark for method coverage
- name: metap
  role: Comparison package with a smaller subset of combination methods, used as a
    benchmark for method coverage
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:6e305d40dfd8cda1d113d89424268e6d149e54499cef0cba5329b0bd0a2cccf1
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
  - metap
  - poolr
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# dependent-p-values-method-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify p-value combination methods by whether they handle independent p-values (e.g. Fisher, Stouffer, Edgington/meanp), dependent p-values via correlation/effective-number-of-tests corrections (e.g. Brown/CCT, minp, MCM, CMC, EBM, Kost, Yang), and weighted or correlated Stouffer variants, using metacp's documented method inventory.

## When to use

Use when choosing or documenting which p-value combination method fits a set of independent or correlated test statistics (e.g. SNPs from a GWAS, gene/probe sets from transcriptomics) and whether correlation or weight matrices must be supplied.

## When NOT to use

- The p-values are independent and the analysis goal is comparing packages rather than choosing a dependent-test method — the independent-only class (fisher, stouffer, meanp, etc.) suffices and no correlation matrix is needed.
- The correlation structure of the test statistics is hard to decipher (e.g. some multi-omics studies) — the article notes the optimal choice of dependent-test method is highly problem dependent and some methods require a reliable correlation matrix.
- The input is effect sizes or raw summary statistics rather than p-values or z-scores — metacp requires the 3rd argument to specify p-values or z-scores and cannot combine other statistic types directly.

## Inputs

- Text file of n rows and k columns containing p-values or z-scores (with headers), e.g. SNP_p_values.txt
- Optional correlation matrix text file (for dependent p-values)
- Optional weight matrix text file (for methods that allow weighting)
- Method names to invoke: logit, meanp, fisher, stouffer, wstouffer, invchi, binomial, cct, minp, mcm, cmc, bonferroni, adbonferroni, cn, lj, gao, galwey, ebm, kost, yang, corstouffer, wcorstouffer

## Outputs

- Output text file with combined p-values per variable, e.g. output.txt
- Classification of each method into independent, dependent (correlation-adjusted), weighted, or correlated-Stouffer classes

## How to apply

Start from metacp's documented method list and split it into classes: independent-only methods (logit, meanp, fisher, stouffer, wstouffer, invchi, binomial, bonferroni); dependent-test methods that need a correlation matrix or estimate one from observations (cct, minp, mcm, cmc, adbonferroni, cn, lj, gao, galwey, ebm, kost, yang); and correlated/weighted Stouffer variants (corstouffer, wcorstouffer). For each candidate method, check the required inputs: methods like wstouffer need a weight matrix, methods like cn, lj, gao, galwey adjust for the effective number of tests via eigenvalues of the correlation matrix, and ebm/kost/yang use covariance or polynomial fits of the correlation structure. Run metacp with the 7-argument command form (input file, output file, p-values/z-scores, Yes/No meta-analysis flag, correlation matrix file, weight matrix file, method names) and record which class each method belongs to. When the correlation matrix is omitted, metacp computes one by default from the observations according to the requested method, so note this behavior per method. Validate the classification against the article's claim that metacp implements the largest collection of combination methods compared to poolr and metap.

## Related tools

- **metacp** (Implements all 22 documented combination methods for independent and dependent p-values; invoked per-method on the command line) — https://github.com/pbagos/metacp
- **poolr** (Comparison package with a smaller subset of combination methods, used as a benchmark for method coverage)
- **metap** (Comparison package with a smaller subset of combination methods, used as a benchmark for method coverage)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values Yes correlation_matrix.txt weight_matrix.txt fisher meanp bonferroni wstouffer
```

## Evaluation signals

- Every method named in the metacp README (logit through wcorstouffer, 22 methods) is assigned to exactly one class and marked available in metacp.
- Methods requiring dependence correction (cn, lj, gao, galwey, ebm, kost, yang, corstouffer, wcorstouffer) all reference correlation/eigenvalue/covariance logic in their descriptions.
- Running metacp without a correlation matrix file succeeds for dependent methods, since the program 'calculates by default a correlation matrix from the observations'.
- Combined p-values for all requested methods appear in output.txt, one per variable row of the input file.
- Coverage comparison against poolr and metap shows metacp implementing the largest collection, consistent with the article's finding.

## Limitations

- The various methods implemented are not all suitable for the same analysis; for dependent tests the optimal choice is highly problem dependent.
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher, complicating method choice.
- Not all methods allow weighting; a weight matrix is only used 'for methods that allow weighting'.

## Evidence
