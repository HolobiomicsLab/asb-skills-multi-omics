---
name: software-method-coverage-extraction
description: 'Reference for building a software feature-coverage comparison: when
  the goal is to enumerate which statistical methods (e.g. Fisher, Stouffer, CCT,
  EBM) each competing package implements and output a 0/1 coverage matrix (CSV).'
when_to_use_negative:
- The goal is to actually run p-value combination on data (use metacp directly via
  python metacp.py) rather than document which packages implement which methods
- The source article provides no explicit comparison table or package list, so package-level
  0/1 entries could not be grounded and would have to be invented
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: metacp
  role: Source software whose implemented method list (logit, meanp, fisher, stouffer,
    wstouffer, invchi, binomial, cct, minp, mcm, cmc, bonferroni, adbonferroni, cn,
    lj, gao, galwey, ebm, kost, yang, corstouffer, wcorstouffer) populates the coverage
    matrix rows
  repo: https://github.com/pbagos/metacp
- name: poolr
  role: Comparison package column in the coverage matrix; article states metacp implements
    a larger method collection than poolr
- name: metap
  role: Comparison package column in the coverage matrix; article states metacp implements
    a larger method collection than metap
- name: Python
  role: Environment in which metacp runs (python metacp.py) and in which the coverage
    CSV can be assembled and validated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:746bdcf182fbc692a2ce7dc221fcaa71823714fac5328a889b74f32195462916
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# software-method-coverage-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts the canonical list of p-value combination methods implemented by a tool (metacp) and compiles a binary method-by-package coverage matrix comparing metacp against poolr and metap, mirroring the article's comparison table (Table 2).

## When to use

Reference for building a software feature-coverage comparison: when the goal is to enumerate which statistical methods (e.g. Fisher, Stouffer, CCT, EBM) each competing package implements and output a 0/1 coverage matrix (CSV).

## When NOT to use

- The goal is to actually run p-value combination on data (use metacp directly via python metacp.py) rather than document which packages implement which methods
- The source article provides no explicit comparison table or package list, so package-level 0/1 entries could not be grounded and would have to be invented

## Inputs

- Article methods section listing implemented combination methods (e.g. fisher, stouffer, cct, ebm)
- Repository README enumerating metacp's available methods and CLI usage
- Article comparison table (Table 2) of method coverage across metacp, poolr, metap and other packages

## Outputs

- methods_coverage_matrix.csv with a header row of package names and one binary 0/1 row per combination method

## How to apply

First extract the canonical list of implemented combination methods from the article's methods section and README: logit, meanp, fisher, stouffer, wstouffer, invchi, binomial, cct, minp, mcm, cmc, bonferroni, adbonferroni, cn, lj, gao, galwey, ebm, kost, yang, corstouffer, and wcorstouffer. Next identify the comparison packages from the article text (metacp, poolr, metap) and, for each method-package pair, assign a binary 1/0 value indicating whether the package implements that method, consulting the article's comparison table. Assemble the methods as rows and package names as header columns into methods_coverage_matrix.csv. Validate that every row carries a binary value for each package and that the metacp column marks all methods listed in the README's available-methods list as 1, reflecting the finding that metacp implements the largest collection of methods.

## Related tools

- **metacp** (Source software whose implemented method list (logit, meanp, fisher, stouffer, wstouffer, invchi, binomial, cct, minp, mcm, cmc, bonferroni, adbonferroni, cn, lj, gao, galwey, ebm, kost, yang, corstouffer, wcorstouffer) populates the coverage matrix rows) — https://github.com/pbagos/metacp
- **poolr** (Comparison package column in the coverage matrix; article states metacp implements a larger method collection than poolr)
- **metap** (Comparison package column in the coverage matrix; article states metacp implements a larger method collection than metap)
- **Python** (Environment in which metacp runs (python metacp.py) and in which the coverage CSV can be assembled and validated)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values Yes correlation_matrix.txt weight_matrix.txt fisher meanp bonferroni wstouffer
```

## Evaluation signals

- methods_coverage_matrix.csv exists and every method row carries a binary 0/1 value for each package column
- The metacp column equals 1 for all methods enumerated in the README's available-methods list (22 methods from logit through wcorstouffer)
- Header row contains exactly the comparison package names drawn from the article (e.g. metacp, poolr, metap)
- Method row labels match the article/README terminology verbatim (e.g. invchi, wstouffer, corstouffer, not paraphrased names)
- Cross-check: poolr and metap columns contain at least one 0 for methods the article reports as missing, consistent with metacp implementing the largest collection

## Limitations

- Coverage entries for non-metacp packages depend entirely on the article's comparison table; if a package's implementation status is not stated there, it cannot be filled honestly
- The article notes that 'the various methods implemented are not all suitable for the same analysis' and that for dependent tests 'the optimal choice is also highly' problem dependent, so coverage breadth does not imply suitability for a given analysis
- In some multi-omics studies the correlation structure of the test statistics is hard to decipher, limiting the applicability of correlation-adjusting methods such as ebm, kost, yang, cn, lj, gao, and galwey

## Evidence
