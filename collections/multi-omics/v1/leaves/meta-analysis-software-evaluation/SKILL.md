---
name: meta-analysis-software-evaluation
description: Use when choosing or benchmarking software for combining independent
  or dependent p-values and you need to verify which combination methods each candidate
  package (e.g. metacp, poolr, metap) actually implements.
when_to_use_negative:
- The goal is simply to run a p-value combination on a dataset (use the chosen tool
  directly rather than building a coverage matrix)
- The comparison targets effect-size meta-analysis (pooling estimates/SEs) rather
  than combination of p-values or z-scores
- No documentation or published comparison table exists for the candidate packages,
  so binary implementation flags cannot be verified
tools:
- name: metacp
  role: Tool under evaluation; documented method list supplies the row set and the
    reference column expected to be all 1s
  repo: https://github.com/pbagos/metacp
- name: poolr
  role: Comparison package whose implemented methods populate a column of the coverage
    matrix
- name: metap
  role: Comparison package whose implemented methods populate a column of the coverage
    matrix
- name: Python
  role: Language used to run metacp and to assemble/validate the CSV coverage matrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:225283a60f66ff13f31b5d4a69a3763a9fb2f67b232cc7c2371ef29fa9749e91
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# meta-analysis-software-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Systematically compares which p-value combination methods (Fisher, Stouffer, logit, meanp, invchi, binomial, cct, minp, mcm, cmc, bonferroni, EBM, and correlated-test variants) are implemented by competing software packages (metacp, poolr, metap), producing a binary method-by-package coverage matrix to justify tool selection.

## When to use

Use when choosing or benchmarking software for combining independent or dependent p-values and you need to verify which combination methods each candidate package (e.g. metacp, poolr, metap) actually implements.

## When NOT to use

- The goal is simply to run a p-value combination on a dataset (use the chosen tool directly rather than building a coverage matrix)
- The comparison targets effect-size meta-analysis (pooling estimates/SEs) rather than combination of p-values or z-scores
- No documentation or published comparison table exists for the candidate packages, so binary implementation flags cannot be verified

## Inputs

- Article's method-coverage comparison table (Table 2) enumerating combination methods per package
- Repository README / documentation listing each tool's available methods (e.g. metacp's logit, meanp, fisher, stouffer, wstouffer, invchi, binomial, cct, minp, mcm, cmc, bonferroni, adbonferroni, cn, lj, gao, galwey, ebm, kost, yang, corstouffer, wcorstouffer)
- Input text file of n rows and k columns containing p-values or z-scores (when exercising the tools being compared)

## Outputs

- methods_coverage_matrix.csv — matrix with methods as rows, package names (metacp, poolr, metap, others) as header columns, and binary 0/1 implementation flags per cell

## How to apply

First extract the canonical list of implemented combination methods from the article's methods enumeration (logit, meanp, fisher, stouffer, wstouffer, invchi, binomial, cct, minp, mcm, cmc, bonferroni, adbonferroni, cn, lj, gao, galwey, ebm, kost, yang, corstouffer, wcorstouffer). Next identify the comparison packages named in the article's Table 2 (metacp, poolr, metap, and Robust-type tools) using the article text and repository documentation. For each method-package pair, assign a binary entry (1 if implemented, 0 if not), cross-checking the article's comparison table against the tool's documented method list. Assemble rows (methods) and columns (packages) into a single matrix and write it as methods_coverage_matrix.csv with a header row of package names.

## Related tools

- **metacp** (Tool under evaluation; documented method list supplies the row set and the reference column expected to be all 1s) — https://github.com/pbagos/metacp
- **poolr** (Comparison package whose implemented methods populate a column of the coverage matrix)
- **metap** (Comparison package whose implemented methods populate a column of the coverage matrix)
- **Python** (Language used to run metacp and to assemble/validate the CSV coverage matrix)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values Yes correlation_matrix.txt weight_matrix.txt fisher meanp bonferroni wstouffer
```

## Evaluation signals

- methods_coverage_matrix.csv exists and every method row carries a binary 0/1 value for each package column (no missing cells)
- The metacp column marks all methods listed in the article's implemented-methods table as present (value 1)
- The set of method rows matches the article's enumeration exactly (no extra or missing methods such as omitting cmc, yang, or wcorstouffer)
- Header row contains exactly the package names covered by the article's comparison table (metacp, poolr, metap, and others named there)
- Cell values are consistent between the article's Table 2 and each package's own documentation (no contradictions)

## Limitations

- The various methods implemented are not all suitable for the same analysis, so coverage counts alone do not indicate methodological appropriateness
- For combination of dependent tests the optimal choice is highly problem dependent, limiting the usefulness of a pure presence/absence comparison
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher, complicating evaluation of dependent-test methods
- No changelog was found for the repository, so version-specific implementation details cannot be traced

## Evidence
