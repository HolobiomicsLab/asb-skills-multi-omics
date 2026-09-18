---
name: pvalue-combination-dependent-tests
description: Use when input is a text file of n rows × k columns of p-values (or z-scores)
  whose columns/tests are correlated (e.g. SNPs within a gene, correlated GWAS tests)
  and a combined p-value robust to dependence is required.
when_to_use_negative:
- The p-values to be combined are independent — use Fisher, Stouffer, Edgington or
  the other independent-p methods instead of the dependence-robust CCT/MinP/MCM/CMC
  family
- The correlation structure of the test statistics is hard to decipher or the input
  is z-scores from unrelated omics blocks where a correlation matrix cannot be reliably
  supplied or estimated
- The analysis goal is adjusting for the effective number of tests (Cheverud-Nyholt,
  Li-Ji, Gao, Galwey, adbonferroni) rather than combining p-values under dependence
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0625
tools:
- name: metacp
  role: Primary CLI tool that implements cct, minp, mcm and cmc for dependent p-values
    and writes combined p-values to a user-specified output file
  repo: https://github.com/pbagos/metacp
- name: Python
  role: Runtime for metacp.py and for post-hoc -log10 transformation and R2 comparison
    against reference combined p-values
- name: GWAR
  role: Reference package providing MIN2/MAX combined p-values used to validate metacp's
    dependent-method agreement (R2 on -log10 p-values)
merged_aliases:
- dependent-pvalue-robust-combination
- pvalue-combination-method-execution
- pvalue-combination-statistics
merged_alias_records:
- alias: dependent-pvalue-robust-combination
  slug: dependent-pvalue-robust-combination
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
- alias: pvalue-combination-method-execution
  slug: pvalue-combination-method-execution
  cosine_score: 0.8206
  method: embedding-cosine
  decision: needs_review
- alias: pvalue-combination-statistics
  slug: pvalue-combination-statistics
  cosine_score: 0.8851
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_002
  - task_004
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:5eed266ab6c97652df375ce39f1dd11b45780b33299e1bc2b70d24bc20b53519
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
  - GWAR
  - Python
  - metacp
  - poolr
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# pvalue-combination-dependent-tests

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Combine dependent (correlated) p-values using metacp's robust correlation-free methods — CCT, MinP, MCM (2*min(pCCT, pMinP, 0.5)) and CMC (CCT applied to pCCT and pMinP) — supplying a correlation matrix or letting metacp derive one, without time-consuming numerical integrations.

## When to use

Use when input is a text file of n rows × k columns of p-values (or z-scores) whose columns/tests are correlated (e.g. SNPs within a gene, correlated GWAS tests) and a combined p-value robust to dependence is required.

## When NOT to use

- The p-values to be combined are independent — use Fisher, Stouffer, Edgington or the other independent-p methods instead of the dependence-robust CCT/MinP/MCM/CMC family
- The correlation structure of the test statistics is hard to decipher or the input is z-scores from unrelated omics blocks where a correlation matrix cannot be reliably supplied or estimated
- The analysis goal is adjusting for the effective number of tests (Cheverud-Nyholt, Li-Ji, Gao, Galwey, adbonferroni) rather than combining p-values under dependence

## Inputs

- Text file (n rows × k columns, plus headers) of p-values or z-scores to be combined, e.g. SNP_p_values.txt from the 67-SNP type-2 diabetes GWAS
- Optional correlation matrix text file for dependent p-values (5th CLI argument; default correlation computed from observations if omitted)
- Optional weight matrix text file for methods that allow weighting (6th CLI argument)

## Outputs

- User-specified output text file (e.g. output.txt) with one combined p-value per input row per requested method (cct, minp, mcm, cmc, hmp)
- Optional per-condition agreement table (e.g. gwar_agreement_table.csv) of R2 values on -log10 p-values versus reference methods (GWAR MIN2/MAX)

## How to apply

Load the p-value file and, when available, a correlation matrix file (metacp computes a default correlation matrix from the observations if the 5th argument is omitted). Run metacp as 'python metacp.py <input> <output> p-values Yes correlation_matrix.txt cct minp mcm cmc', so each row receives combined p-values from the Cauchy Combination Test (tangent-transform each p, sum, evaluate the upper tail of Cauchy(0,1)), Tippett's MinP (1-(1-min(p))^k), and the hybrids MCM = 2*min(pCCT, pMinP, 0.5) and CMC = CCT(pCCT, pMinP). Optionally add hmp (k / sum_i(1/p_i)) as another robust option. These methods avoid the complex and time-consuming numerical integrations required by tools like GWAR. To validate, transform combined p-values to -log10 scale and compute R2 against a reference such as GWAR's MIN2/MAX; expected agreement on the 67-SNP type-2 diabetes GWAS data spans R2 = 0.9818 to 1.000. Decide between methods per problem: the article notes the optimal choice for dependent tests is highly problem dependent, and not all implemented methods suit the same analysis.

## Related tools

- **metacp** (Primary CLI tool that implements cct, minp, mcm and cmc for dependent p-values and writes combined p-values to a user-specified output file) — https://github.com/pbagos/metacp
- **Python** (Runtime for metacp.py and for post-hoc -log10 transformation and R2 comparison against reference combined p-values)
- **GWAR** (Reference package providing MIN2/MAX combined p-values used to validate metacp's dependent-method agreement (R2 on -log10 p-values))

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values Yes correlation_matrix.txt cct minp mcm cmc
```

## Evaluation signals

- All combined p-values fall within [0,1] for every row and every requested method
- R2 computed on -log10 combined p-values between metacp (CCT/MinP/MCM/CMC) and GWAR (MIN2/MAX) falls in the 0.9818–1.000 range reported for the 67-SNP type-2 diabetes GWAS
- MCM output equals 2*min(pCCT, pMinP, 0.5) and CMC output equals CCT applied to the pair (pCCT, pMinP) for each row
- pCCT matches the standard Cauchy upper-tail evaluation of the summed tangent-transformed p-values within numerical tolerance
- Output file contains one combined p-value per input row per method, with no missing rows relative to the input file

## Limitations

- Regarding the combination of dependent tests, the optimal choice is highly problem dependent — no single method (cct/minp/mcm/cmc) is universally best
- The various implemented methods are not all suitable for the same analysis; method choice must match the dependence setting
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher, which can undermine dependent-p methods
- If no correlation matrix file is supplied, metacp calculates a default one from the observations, which may not reflect the true dependence structure

## Evidence
