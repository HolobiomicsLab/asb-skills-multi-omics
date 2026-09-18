---
name: package-feature-comparison-matrix-construction
description: Use when you need to quantify and compare the feature/method coverage
  of overlapping software packages against a canonical list of methods enumerated
  in an article's methods table or tool documentation.
when_to_use_negative:
- When the goal is to run or benchmark the combination methods on actual p-value data
  rather than compare package feature coverage
- When the packages under comparison do not overlap in function (e.g. an alignment
  tool vs a p-value combiner), making a shared method axis meaningless
- When a package's documentation is insufficient to determine which methods it implements,
  since binary entries would be guesses
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: metacp
  role: Subject package whose documented method list (logit through wcorstouffer)
    defines the rows and the reference column of the coverage matrix
  repo: github.com/pbagos/metacp
- name: poolr
  role: Comparison package column; evaluated for which p-value combination methods
    it implements
- name: metap
  role: Comparison package column; evaluated for which p-value combination methods
    it implements
- name: Python
  role: Used to assemble the method-package matrix and write methods_coverage_matrix.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:93097c368b56ea03f9a26aa9d571cadbb2b617c08fa500e16f43a6501a006c4b
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
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# package-feature-comparison-matrix-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a binary method-coverage matrix comparing the p-value combination methods (Fisher, Stouffer, logit, CCT, MinP, MCM, CMC, EBM, etc.) implemented across competing software packages such as metacp, poolr, and metap, to establish which tool offers the widest methodological choice.

## When to use

Use when you need to quantify and compare the feature/method coverage of overlapping software packages against a canonical list of methods enumerated in an article's methods table or tool documentation.

## When NOT to use

- When the goal is to run or benchmark the combination methods on actual p-value data rather than compare package feature coverage
- When the packages under comparison do not overlap in function (e.g. an alignment tool vs a p-value combiner), making a shared method axis meaningless
- When a package's documentation is insufficient to determine which methods it implements, since binary entries would be guesses

## Inputs

- Article methods table or README enumerating implemented methods (e.g. metacp's 22 named combination methods)
- Article text identifying comparison packages (poolr, metap)
- Documentation/READMEs of each comparison package

## Outputs

- methods_coverage_matrix.csv: rows = combination methods, columns = packages, cells = binary 1/0 implementation flags with a header row of package names

## How to apply

First extract the canonical method list from the source's implemented-methods documentation (for metacp this yields logit, meanp, fisher, stouffer, wstouffer, invchi, binomial, cct, minp, mcm, cmc, bonferroni, adbonferroni, cn, lj, gao, galwey, ebm, kost, yang, corstouffer, wcorstouffer). Next, identify the comparison packages named in the article text (poolr, metap) and consult each package's documentation to fill in its column. For each method-package pair, enter a binary 1/0 value marking whether the package implements that method. Assemble the rows (methods) and columns (packages) into a single matrix, e.g. methods_coverage_matrix.csv with a header row of package names. Validation checks: every method row carries a binary value per package, and the metacp column marks all documented methods as present, supporting the article's finding that metacp implements the largest collection of combination methods.

## Related tools

- **metacp** (Subject package whose documented method list (logit through wcorstouffer) defines the rows and the reference column of the coverage matrix) — github.com/pbagos/metacp
- **poolr** (Comparison package column; evaluated for which p-value combination methods it implements)
- **metap** (Comparison package column; evaluated for which p-value combination methods it implements)
- **Python** (Used to assemble the method-package matrix and write methods_coverage_matrix.csv)

## Examples

```
methods = ['logit','meanp','fisher','stouffer','wstouffer','invchi','binomial','cct','minp','mcm','cmc','bonferroni','adbonferroni','cn','lj','gao','galwey','ebm','kost','yang','corstouffer','wcorstouffer']; import csv; csv.writer(open('methods_coverage_matrix.csv','w')).writerow(['method','metacp','poolr','metap'])
```

## Evaluation signals

- methods_coverage_matrix.csv exists and every method row carries a binary 0/1 value for each package column
- The metacp column is 1 for all methods listed in the article's/README's implemented-methods table (logit, meanp, fisher, stouffer, wstouffer, invchi, binomial, cct, minp, mcm, cmc, bonferroni, adbonferroni, cn, lj, gao, galwey, ebm, kost, yang, corstouffer, wcorstouffer)
- Header row exactly matches the package set named in the article (metacp, poolr, metap, and any others covered)
- No non-binary (missing/NA) entries remain without explicit justification
- Coverage counts are consistent with the article's claim that metacp implements the largest collection of methods

## Limitations

- Not all implemented methods are suitable for the same analysis — coverage breadth does not imply methodological appropriateness for a given correlation structure
- Coverage entries depend on package documentation quality; a method may be implemented but undocumented, leading to false 0s
- The comparison reflects the state of the article and repository (BMC Bioinformatics 2025; no changelog found for the repository), so newer package versions may alter coverage

## Evidence

- [readme] README enumerates the full method list that populates the matrix rows: [evidence span withheld — anchor status: not_found]
