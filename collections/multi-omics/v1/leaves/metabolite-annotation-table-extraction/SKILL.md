---
name: metabolite-annotation-table-extraction
description: Use when a paper's supplementary table reports metabolite identity confirmations
  by analytical standards (measured m/z, CAS number, mass error in ppm, retention-time
  error) and these annotations must be extracted, validated, and exported.
when_to_use_negative:
- The input is raw LC-MS data requiring peak picking or annotation de novo — this
  skill only reproduces annotations already published in a standard-confirmation table
- The task concerns Scoary2's mGWAS association outputs (genes/traits, p values) rather
  than the paper's metabolome annotation metadata
- The published table lacks the fields needed for validation (CAS number, mass error
  in ppm, retention-time error), so reproduced values cannot be checked against the
  reference
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pandas
  role: Used to programmatically read the supplementary confirmation table and extract
    the measured m/z, CAS number, mass error, and retention-time error per metabolite
    before writing the output CSV
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:385d45e0bfdbcb4f22652b70b9a0217861038710168a01160c77792d2c79ca1a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: thin
yes_rate: 0.0
n_claims: 2
---

# metabolite-annotation-table-extraction

## Summary

Reproduces published LC-MS metabolite identity-confirmation tables (e.g. Table 1 of the Scoary2 paper, DOI 10.1186/s13059-024-03233-7) by programmatically extracting measured m/z, CAS numbers, mass errors (ppm), and retention-time errors for each metabolite confirmed by pure-standard injection.

## When to use

Use when a paper's supplementary table reports metabolite identity confirmations by analytical standards (measured m/z, CAS number, mass error in ppm, retention-time error) and these annotations must be extracted, validated, and exported.

## When NOT to use

- The input is raw LC-MS data requiring peak picking or annotation de novo — this skill only reproduces annotations already published in a standard-confirmation table
- The task concerns Scoary2's mGWAS association outputs (genes/traits, p values) rather than the paper's metabolome annotation metadata
- The published table lacks the fields needed for validation (CAS number, mass error in ppm, retention-time error), so reproduced values cannot be checked against the reference

## Inputs

- Supplementary Table 1 (LC-MS standard-confirmation table) from the Scoary2 paper (DOI 10.1186/s13059-024-03233-7)
- List of target metabolite names: decanoylcarnitine, octanoylcarnitine, hexanoylcarnitine, carnitine, acetylcarnitine
- Reported mass-accuracy threshold (<1 ppm) for standard-confirmation entries

## Outputs

- table1_acylcarnitine_identity_confirmation.csv with columns metabolite, measured_mz, cas_number, mass_error_ppm, rt_error
- Per-metabolite flags for mass errors deviating from the <1 ppm threshold

## How to apply

Retrieve the supplementary table (here supplementary Table 1 of the Scoary2 paper) containing the identity confirmation of carnitine species detected in the yogurt metabolome. Use pandas to read the table and extract, for each of the five carnitine-related metabolites (decanoylcarnitine, octanoylcarnitine, hexanoylcarnitine, carnitine, acetylcarnitine), the measured m/z, CAS number, mass error (ppm), and retention-time error. Check every mass error against the reported <1 ppm accuracy threshold and flag any entry that deviates — e.g. octanoylcarnitine at 2.02 ppm exceeds the threshold even though decanoylcarnitine and hexanoylcarnitine are <1 ppm. Write the five records to a CSV (table1_acylcarnitine_identity_confirmation.csv) with columns metabolite, measured_mz, cas_number, mass_error_ppm, rt_error, then diff each reproduced value against the published Table 1 entry.

## Related tools

- **pandas** (Used to programmatically read the supplementary confirmation table and extract the measured m/z, CAS number, mass error, and retention-time error per metabolite before writing the output CSV)

## Examples

```
import pandas as pd; df = pd.read_excel('scoary2_supplementary_table1.xlsx'); df[df.metabolite.isin(['decanoylcarnitine','octanoylcarnitine','hexanoylcarnitine','carnitine','acetylcarnitine'])].to_csv('table1_acylcarnitine_identity_confirmation.csv', index=False)
```

## Evaluation signals

- Each reproduced value matches the published Table 1 entry (output_matches_reference): e.g. decanoylcarnitine m/z 316.24764 / CAS 3992-45-8 / RT error 0.67%, octanoylcarnitine m/z 288.21635 / CAS 25243-95-2 / RT error 0.47%, hexanoylcarnitine m/z 260.18515 / CAS 22671-29-0 / RT error 2.13%
- All reported mass errors are checked against the <1 ppm threshold and any deviation (octanoylcarnitine, 2.02 ppm) is explicitly flagged rather than silently passed
- All five metabolite names plus the m/z, CAS, and error fields are present in the output file
- Output CSV schema is exactly metabolite, measured_mz, cas_number, mass_error_ppm, rt_error with five records

## Limitations

- Only decanoylcarnitine, octanoylcarnitine, and hexanoylcarnitine are confirmed by standard injection in Table 1; carnitine and acetylcarnitine entries must be extracted from the same table but their confirmation evidence differs
- Mass errors can exceed the <1 ppm threshold (octanoylcarnitine, 2.02 ppm), so the threshold check must be a validation step, not an assumption
- Discussion-section limitations of the paper are not available in the extracted document, so additional caveats about the LC-MS confirmation workflow could not be traced

## Evidence

- [supplementary] Table 1 confirms decanoylcarnitine (measured m/z 316.24764, CAS 3992-45-8, mass error <1 ppm, retention-time error 0.67%): [evidence span withheld — anchor status: not_found]
- [abstract] we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism: "we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism"
- [other] extract the measured m/z, CAS number, mass error (ppm), and retention-time error from the published table using pandas: [evidence span withheld — anchor status: not_found]
