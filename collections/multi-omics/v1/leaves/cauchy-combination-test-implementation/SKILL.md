---
name: cauchy-combination-test-implementation
description: Use when combining k dependent p-values per row (e.g. GWAS SNPs from
  a text file of n rows x k columns) and numerical-integration methods like GWAR are
  too slow, or when the correlation structure is unknown.
when_to_use_negative:
- The p-values to combine are fully independent and a classical method (Fisher's,
  Stouffer's, Edgington's) is explicitly preferred — CCT and its hybrids target dependent
  p-values
- The analysis already produces combined p-values from a method requiring no numerical
  integrations and only a subset of these robust methods is needed
- Input contains z-scores without first confirming the statistic type to the tool
  (the 3rd CLI argument must be 'p-values' or 'z-scores'), risking misinterpretation
  of the tangent/normal transforms
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0622
tools:
- name: metacp
  role: Reference Python/STATA package whose cct, minp, mcm and cmc methods are reimplemented;
    computes combined p-values without complex and time-consuming numerical integrations
  repo: https://github.com/pbagos/metacp
- name: Python
  role: Implementation language for the module accepting p-value vectors and writing
    combined p-values to an output text file, mirroring the metacp interface
- name: GWAR
  role: 'Comparison baseline for validation: metacp''s CCT/MinP/MCM/CMC were checked
    against GWAR (MAX, MIN2) on 67 SNPs from a type-2 diabetes GWAS'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:dd4ba616567de503a1d9f80b4b4349c96d3c77aae2e17a31c4630d17c90618dc
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cauchy-combination-test-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implement metacp's robust correlation-free combination methods for dependent p-values: the Cauchy Combination Test (CCT), Tippett's MinP, the hybrid MinP-Cauchy-MinP (MCM) and Cauchy-MinP-Cauchy (CMC) constructions, and the Harmonic Mean p-value (HMP), all computed without complex numerical integrations.

## When to use

Use when combining k dependent p-values per row (e.g. GWAS SNPs from a text file of n rows x k columns) and numerical-integration methods like GWAR are too slow, or when the correlation structure is unknown.

## When NOT to use

- The p-values to combine are fully independent and a classical method (Fisher's, Stouffer's, Edgington's) is explicitly preferred — CCT and its hybrids target dependent p-values
- The analysis already produces combined p-values from a method requiring no numerical integrations and only a subset of these robust methods is needed
- Input contains z-scores without first confirming the statistic type to the tool (the 3rd CLI argument must be 'p-values' or 'z-scores'), risking misinterpretation of the tangent/normal transforms

## Inputs

- Text file of n rows and k columns of p-values (or z-scores) to be combined, e.g. SNP_p_values.txt
- Optional text file with the correlation matrix for dependent p-values
- Optional text file with the weight matrix (for methods that allow weighting)

## Outputs

- Combined p-values for each row, one per requested method (cct, minp, mcm, cmc, hmp), written to a user-provided output text file such as output.txt

## How to apply

Parse the input text file of n rows and k columns into numeric vectors of k p-values to be combined per row. For CCT, transform each p-value with the tangent function, sum the transformed values, and evaluate p_CCT = P[C(0,1) >= t], the upper tail of the standard Cauchy(0,1) distribution (inverse Cauchy CDF). For MinP, apply Tippett's adjustment p_c = 1 - (1 - min(p_1,...,p_k))^k. Form the hybrids as MCM = 2*min(p_CCT, p_MinP, 0.5) and CMC = CCT applied to (p_CCT, p_MinP); implement HMP as p_c = k / sum_i(1/p_i). Expose all five methods (cct, minp, mcm, cmc, hmp) mirroring the metacp CLI interface that writes combined p-values to a user-provided output text file; if no correlation matrix file is given, metacp calculates one by default from the observations according to the specifics of the requested method.

## Related tools

- **metacp** (Reference Python/STATA package whose cct, minp, mcm and cmc methods are reimplemented; computes combined p-values without complex and time-consuming numerical integrations) — https://github.com/pbagos/metacp
- **Python** (Implementation language for the module accepting p-value vectors and writing combined p-values to an output text file, mirroring the metacp interface)
- **GWAR** (Comparison baseline for validation: metacp's CCT/MinP/MCM/CMC were checked against GWAR (MAX, MIN2) on 67 SNPs from a type-2 diabetes GWAS)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values No cct minp mcm cmc
```

## Evaluation signals

- Every combined p-value lies in [0,1] for all rows of the input file
- p_CCT is computed as P[C(0,1) >= t] where t is the sum of tangent-transformed p-values; verify against the inverse Cauchy CDF on known vectors
- MinP equals 1 - (1 - min(p_1,...,p_k))^k; MCM equals 2*min(p_CCT, p_MinP, 0.5) and CMC equals CCT applied to (p_CCT, p_MinP)
- R^2 agreement with GWAR's MAX/MIN2 output on the 67 type-2 diabetes GWAS SNPs falls within the reported 0.9818 to 1.000 range
- Reproduces published reference p-values for CCT, MinP, MCM, CMC without performing numerical integrations

## Limitations

- The various methods implemented are not all suitable for the same analysis; for combination of dependent tests the optimal choice is highly problem dependent
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher
- MCM is capped at 0.5 (MCM = 2*min(p_CCT, p_MinP, 0.5)), so values above the cap are not reachable; hybrid behavior differs from plain CCT or MinP

## Evidence

- [other] metacp provides the CCT, CMC, MCM and minP methods for combining dependent p-values: [evidence span withheld — anchor status: not_found]
- [results] computes combined p-values without numerical integrations: "The advantage of metacp lies in its ability to compute combined p-values without relying on the complex and time-consuming numerical integrations required by GWAR."
- [results] R2 agreement range on 67 SNPs: "we show the R2 values that range from 0.9818 to 1.000, demonstrating consistent agreement across different approaches"
- [readme] README description of the cct method: "cct: This method combines p-values using a Cauchy distribution. It transforms each p-value using the tangent function, sums these transformed values"
- [readme] default correlation matrix behavior: "If this argument is omitted the program calculates by default a correlation matrix from the observations according to the specifics of the requested method."
