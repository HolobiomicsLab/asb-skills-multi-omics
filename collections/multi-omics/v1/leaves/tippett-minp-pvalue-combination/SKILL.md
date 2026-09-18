---
name: tippett-minp-pvalue-combination
description: Use when combining k dependent p-values (e.g., multiple tests per SNP/gene
  row in a GWAS text file) and a minimum-based combination is desired, alone or as
  a component of the MCM/CMC hybrid methods.
when_to_use_negative:
- When the input consists of z-scores rather than p-values without first converting
  them, since the method operates on p-values
- When the analysis requires methods accounting for the effective number of tests
  via eigenvalues of the correlation matrix (e.g., Cheverud-Nyholt, Li-Ji, Gao, Galwey),
  which handle correlation structure differently
- When only independent p-values are present and Fisher's, Stouffer's, or Edgington's
  methods are preferred for that setting
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: metacp
  role: Reference implementation providing the minp (Tippett's minimum-p), mcm (MinP-Cauchy-MinP),
    and cmc (Cauchy-MinP-Cauchy) methods for combining dependent p-values via a command-line
    interface
  repo: github.com/pbagos/metacp
- name: Python
  role: Language in which the metacp module (metacp.py) is invoked and in which a
    reimplementation of the minp/mcm/cmc methods can be written
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:152f420ae68cd593fb5523ebf0194f3d2c30dac1616dbb0eebb1cb0c0f1eee51
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
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# tippett-minp-pvalue-combination

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies Tippett's minimum-p (MinP) method, as implemented in the metacp package, to combine a vector of dependent p-values into a single combined p-value adjusted for the number of tests, without numerical integration.

## When to use

Use when combining k dependent p-values (e.g., multiple tests per SNP/gene row in a GWAS text file) and a minimum-based combination is desired, alone or as a component of the MCM/CMC hybrid methods.

## When NOT to use

- When the input consists of z-scores rather than p-values without first converting them, since the method operates on p-values
- When the analysis requires methods accounting for the effective number of tests via eigenvalues of the correlation matrix (e.g., Cheverud-Nyholt, Li-Ji, Gao, Galwey), which handle correlation structure differently
- When only independent p-values are present and Fisher's, Stouffer's, or Edgington's methods are preferred for that setting

## Inputs

- Text file of n rows and k columns (plus headers) containing p-values (or z-scores) to be combined
- Optional correlation matrix text file for dependent p-values
- Method selection string (e.g., 'minp', 'mcm', 'cmc')

## Outputs

- Output text file with combined p-values per row (one combined p-value per method per row)
- In-memory vector of combined p-values in [0,1]

## How to apply

Parse the input text file of n rows and k columns (with headers) into numeric vectors of k p-values per row. Implement Tippett's MinP for dependent p-values as p_c = 1 - (1 - min(p_1,...,p_k))^k, i.e., take the minimum of the individual p-values and adjust it for the number of tests. Optionally feed the MinP p-value into the hybrid constructions MCM = 2*min(p_CCT, p_MinP, 0.5) and CMC = CCT(p_CCT, p_MinP), where CCT transforms each p-value with the tangent function and evaluates the upper tail of the standard Cauchy(0,1) distribution. Expose the method in a Python module mirroring the metacp interface, writing combined p-values to a user-provided output text file. Rationale: metacp computes these combined p-values without relying on the complex and time-consuming numerical integrations required by tools like GWAR.

## Related tools

- **metacp** (Reference implementation providing the minp (Tippett's minimum-p), mcm (MinP-Cauchy-MinP), and cmc (Cauchy-MinP-Cauchy) methods for combining dependent p-values via a command-line interface) — github.com/pbagos/metacp
- **Python** (Language in which the metacp module (metacp.py) is invoked and in which a reimplementation of the minp/mcm/cmc methods can be written)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values Yes correlation_matrix.txt weight_matrix.txt minp mcm cmc
```

## Evaluation signals

- Verify each combined p-value lies in [0,1]
- Check agreement with published reference p-values for Tippett's MinP on the same input vectors
- Reproduce the R^2 agreement range 0.9818-1.000 that the article reports for metacp's CCT/MinP/MCM/CMC against GWAR (MAX, MIN2) on 67 SNPs from a type-2 diabetes GWAS
- Confirm the combined p-value for minp equals 1 - (1 - min(p_1,...,p_k))^k on hand-computable examples
- Check that MCM output equals 2*min(p_CCT, p_MinP, 0.5) for a given row

## Limitations

- The various methods implemented are not all suitable for the same analysis; the optimal choice for dependent tests is highly problem dependent
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher
- If a correlation matrix file is omitted, the program calculates a default correlation matrix from the observations according to the specifics of the requested method, which may not match the true dependency structure

## Evidence

- [other] metacp provides the CCT, CMC, MCM and minP methods for combining dependent p-values, described as the most suitable methods for such an analysis: [evidence span withheld — anchor status: not_found]
