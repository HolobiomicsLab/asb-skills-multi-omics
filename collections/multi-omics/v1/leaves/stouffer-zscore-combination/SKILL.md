---
name: stouffer-zscore-combination
description: Use when a text file of n rows × k columns holds independent p-values
  (or z-scores) per variable (e.g. SNP/gene rows across tests or traits) and one combined
  p-value per row is needed.
when_to_use_negative:
- Test statistics are dependent/correlated — use metacp's 'corstouffer' or 'wcorstouffer'
  with a correlation matrix file instead
- Only the minimum p-value matters with many tests — Bonferroni ('bonferroni'/'adbonferroni')
  or effective-number-of-tests methods (cn, lj, gao, galwey) may be more appropriate
- The input is raw test statistics rather than p-values or z-scores — metacp expects
  the 3rd argument to declare p-values or z-scores specifically
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0625
tools:
- name: metacp
  role: Python/STATA package whose 'stouffer' and 'wstouffer' methods implement the
    Z-score combination for independent p-values
  repo: https://github.com/pbagos/metacp
- name: Python
  role: Runtime environment; metacp is invoked as 'python metacp.py' from the command
    line
merged_aliases:
- correlated-stouffer-z-score-combination
merged_alias_records:
- alias: correlated-stouffer-z-score-combination
  slug: correlated-stouffer-z-score-combination
  cosine_score: 0.8327
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:b6dc03ecfd9d6eafe3959b25a0d5f0871623771aecebdc38156f409522aac918
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
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# stouffer-zscore-combination

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Combines a row of independent p-values via Stouffer's Z-score method: each p-value is converted to a Z-score, Z-scores are summed, and the sum is transformed through the standard normal distribution, as implemented in metacp's 'stouffer' and weighted 'wstouffer' methods.

## When to use

Use when a text file of n rows × k columns holds independent p-values (or z-scores) per variable (e.g. SNP/gene rows across tests or traits) and one combined p-value per row is needed.

## When NOT to use

- Test statistics are dependent/correlated — use metacp's 'corstouffer' or 'wcorstouffer' with a correlation matrix file instead
- Only the minimum p-value matters with many tests — Bonferroni ('bonferroni'/'adbonferroni') or effective-number-of-tests methods (cn, lj, gao, galwey) may be more appropriate
- The input is raw test statistics rather than p-values or z-scores — metacp expects the 3rd argument to declare p-values or z-scores specifically

## Inputs

- Text file of n rows and k columns plus headers containing p-values or z-scores (one variable per row)
- Weight matrix text file (optional, for the weighted wstouffer variant)
- CLI arguments: input file, output file, 'p-values'/'z-scores' flag, meta-analysis Yes/No flag, method name(s)

## Outputs

- Text output file (e.g. output.txt) with one combined p-value per row/variable

## How to apply

Load the input text file of n rows and k columns (plus headers) containing the p-values or z-scores to be combined, one variable per row. Run metacp with the 3rd argument set to 'p-values' or 'z-scores' as appropriate, the 4th argument 'No' for independent tests, and 'stouffer' as the method argument; metacp converts each p-value to a Z-score via the inverse normal, sums the Z-scores, and computes p_c = 1 - Phi(sum(z_i)/sqrt(k)). For weighted combination supply a weight matrix text file as the 6th argument and use 'wstouffer', which divides the weighted Z-sum by sqrt(sum(w_i^2)). The combined p-values per row are written to the user-specified output file (e.g. output.txt); verify each output lies in [0,1] and matches closed-form reference values (e.g. scipy.stats.norm survival functions) within tolerance.

## Related tools

- **metacp** (Python/STATA package whose 'stouffer' and 'wstouffer' methods implement the Z-score combination for independent p-values) — https://github.com/pbagos/metacp
- **Python** (Runtime environment; metacp is invoked as 'python metacp.py' from the command line)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values No stouffer
```

## Evaluation signals

- Each returned combined p-value lies in [0,1]
- Implemented Stouffer output matches closed-form reference p_c = 1 - Phi(sum(z_i)/sqrt(k)) computed via scipy.stats.norm within tolerance (e.g. absolute error < 1e-8)
- Weighted variant divides the weighted Z-sum by sqrt(sum(w_i^2)) and matches the weighted closed-form value
- Combined p-values are produced for every row of the input file without error
- For dependent tests, outputs agree with reference implementations (paper reports R2 agreement of 0.9818 to 1.000 for related combination approaches)

## Limitations

- The plain 'stouffer' method assumes independence across the k tests; correlated p-values require 'corstouffer'/'wcorstouffer' with a correlation matrix
- The various implemented methods are not all suitable for the same analysis — optimal choice is problem dependent
- Methods that allow weighting require the user to supply a weight matrix text file in the 6th CLI argument

## Evidence
