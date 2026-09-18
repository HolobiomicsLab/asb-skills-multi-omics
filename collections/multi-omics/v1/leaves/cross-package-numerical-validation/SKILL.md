---
name: cross-package-numerical-validation
description: Use when a new or reimplemented statistical method (e.g., p-value combination
  in metacp) must be verified against an established reference package before trust
  is placed in its numerical output.
when_to_use_negative:
- The methods of interest are only implemented in one package (e.g., metacp's cct,
  mcm, cmc, logit, binomial without equivalents elsewhere), so no cross-package reference
  exists
- The comparison targets methods with stochastic or approximation-dependent components
  (e.g., GWAR's numerical integrations), where exact numerical identity is not the
  expected agreement criterion
- The input is not a common p-value matrix shared across tools — comparing results
  on different inputs cannot establish implementation consistency
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: metacp
  role: Primary tool under validation; run via CLI to produce combined p-values for
    each shared method
  repo: github.com/pbagos/metacp
- name: poolr
  role: Reference R package; executes the equivalent shared methods (fisher, stouffer,
    weighted Stouffer, inverse chi-squared, Galwey, Kost) on the same input p-values
- name: Robust tests package
  role: Second reference implementation; executes the same shared methods for a three-way
    numerical comparison
- name: Python
  role: Runtime environment for metacp and for consolidating outputs into comparison.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:46445c864c0baf91abf051dc09a013380087d9d4601cc990285a94d21b41ca5c
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
  - Robust tests package
  - metacp
  - poolr
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cross-package-numerical-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A validation procedure that runs the same p-value combination methods (fisher, stouffer, wstouffer, invchi, galwey, kost) through independent implementations (metacp, poolr, Robust tests package) on a common input and checks that combined p-values are numerically identical across packages.

## When to use

Use when a new or reimplemented statistical method (e.g., p-value combination in metacp) must be verified against an established reference package before trust is placed in its numerical output.

## When NOT to use

- The methods of interest are only implemented in one package (e.g., metacp's cct, mcm, cmc, logit, binomial without equivalents elsewhere), so no cross-package reference exists
- The comparison targets methods with stochastic or approximation-dependent components (e.g., GWAR's numerical integrations), where exact numerical identity is not the expected agreement criterion
- The input is not a common p-value matrix shared across tools — comparing results on different inputs cannot establish implementation consistency

## Inputs

- SNP_p_values.txt — text file of p-values (67 SNPs from a GWAS identifying susceptibility loci for type-2 diabetes), rows = SNPs, columns = tests/traits
- List of shared combination methods implemented in all packages being compared (e.g., fisher, stouffer, wstouffer, invchi, galwey, kost)

## Outputs

- metacp_output.txt — combined p-values per SNP per method from metacp
- poolr_output.txt — combined p-values from the poolr R package
- robust_output.txt — combined p-values from the Robust tests package
- comparison.csv — side-by-side table, one row per SNP, paired columns per package-method combination, used to confirm numerical identity

## How to apply

Load one common input file (e.g., SNP_p_values.txt with p-values for 67 SNPs) and run each tool with the identical shared methods. For metacp, call the CLI as 'python metacp.py SNP_p_values.txt metacp_output.txt p-values No fisher stouffer wstouffer invchi galwey kost'. Run the equivalent methods in the poolr R package and the Robust tests package on the same p-values, saving each package's output to a separate file. Consolidate all combined p-values into a single side-by-side comparison table (comparison.csv) with one row per SNP and paired columns per package-method combination. Then verify, per shared method, that the combined p-values are numerically identical across all three packages; exact agreement is the pass criterion, since the same mathematical method should yield identical results regardless of implementation.

## Related tools

- **metacp** (Primary tool under validation; run via CLI to produce combined p-values for each shared method) — github.com/pbagos/metacp
- **poolr** (Reference R package; executes the equivalent shared methods (fisher, stouffer, weighted Stouffer, inverse chi-squared, Galwey, Kost) on the same input p-values)
- **Robust tests package** (Second reference implementation; executes the same shared methods for a three-way numerical comparison)
- **Python** (Runtime environment for metacp and for consolidating outputs into comparison.csv)

## Examples

```
python metacp.py SNP_p_values.txt metacp_output.txt p-values No fisher stouffer wstouffer invchi galwey kost
```

## Evaluation signals

- For every shared method, combined p-values in comparison.csv are numerically identical (exact match) across metacp, poolr, and the Robust tests package for all 67 SNPs
- Each package's output file contains one combined p-value per SNP per requested method, matching the row count of the input p-value file
- The comparison table pairs each method consistently across packages (no column misalignment between method names)
- The comparison reproduces the published finding: metacp results are identical to poolr and Robust tests results for fisher, stouffer, wstouffer, invchi, galwey, and kost

## Limitations

- Cross-package identity can only be checked for methods that all packages implement; metacp implements the largest collection of methods, so many (e.g., CCT, MCM, CMC) lack equivalents in poolr or metap for this check
- For dependent-p-value methods the optimal choice is highly problem dependent, and in some multi-omics studies the correlation structure of the test statistics is hard to decipher, complicating the choice of methods to cross-validate
- The article's 'Missing signals' note that no changelog was found for the metacp repository and no DOI/title/authors/journal information appears in the GitHub-sourced document, so package version provenance for the validation is incomplete

## Evidence

- [other] finding: For every shared combination method, the combined p-values computed by metacp are identical to those computed by the other packages (poolr and the Robust tests package): [evidence span withheld — anchor status: not_found]
- [other] workflow step: execute metacp CLI on the shared input: [evidence span withheld — anchor status: not_found]
