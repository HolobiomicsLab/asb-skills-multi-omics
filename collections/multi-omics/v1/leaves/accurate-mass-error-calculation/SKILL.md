---
name: accurate-mass-error-calculation
description: Use when reproducing or auditing metabolite identity confirmations from
  LC-MS analysis of pure analytical standard solutions, specifically the carnitine
  species (carnitine, acetylcarnitine, hexanoylcarnitine, octanoylcarnitine, decanoylcarnitine)
  in the yogurt metabolome study.
when_to_use_negative:
- The metabolites of interest were annotated by database matching only and were not
  confirmed by LC-MS injection of pure analytical standards — no m/z/CAS/error confirmation
  records exist.
- The input is a raw metabolomics feature table (e.g. m/z–retention-time features
  without standard confirmation), where feature annotation rather than standard-confirmation
  reproduction is needed.
- The paper under analysis contains no supplementary identity-confirmation table (e.g.
  the Scoary2 main text contains only abstract-style and Methods sections without
  the full LC-MS details).
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3292
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:55aac88fc8ad43df0849cdbc8b2033ed7c461bb251b1aa992110eecc59fbe07a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools: []
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# accurate-mass-error-calculation

## Summary

Extract and validate measured m/z, CAS numbers, mass errors (ppm), and retention-time errors from the Scoary2 paper's supplementary LC-MS standard-confirmation table for five carnitine-related metabolites, checking each mass error against the reported <1 ppm accuracy threshold.

## When to use

Use when reproducing or auditing metabolite identity confirmations from LC-MS analysis of pure analytical standard solutions, specifically the carnitine species (carnitine, acetylcarnitine, hexanoylcarnitine, octanoylcarnitine, decanoylcarnitine) in the yogurt metabolome study.

## When NOT to use

- The metabolites of interest were annotated by database matching only and were not confirmed by LC-MS injection of pure analytical standards — no m/z/CAS/error confirmation records exist.
- The input is a raw metabolomics feature table (e.g. m/z–retention-time features without standard confirmation), where feature annotation rather than standard-confirmation reproduction is needed.
- The paper under analysis contains no supplementary identity-confirmation table (e.g. the Scoary2 main text contains only abstract-style and Methods sections without the full LC-MS details).

## Inputs

- Supplementary Table 1 of the Scoary2 paper (LC-MS standard-confirmation table for carnitine-related metabolites)
- LC-MS analysis results of pure analytical standard solutions (measured m/z, CAS numbers, mass and retention-time errors)

## Outputs

- table1_acylcarnitine_identity_confirmation.csv with columns: metabolite, measured_mz, cas_number, mass_error_ppm, rt_error
- Flagged list of entries whose mass error deviates from the <1 ppm threshold

## How to apply

Retrieve the supplementary Table 1 of the Scoary2 paper (DOI 10.1186/s13059-024-03233-7), which reports identity confirmation by standard injection for decanoylcarnitine, octanoylcarnitine, hexanoylcarnitine, carnitine, and acetylcarnitine. Using pandas, extract for each metabolite the measured m/z, CAS number, mass error in ppm, and retention-time error, e.g. decanoylcarnitine at measured m/z 316.24764 (CAS 3992-45-8, mass error <1 ppm, RT error 0.67%). Validate each mass error against the reported <1 ppm accuracy threshold and flag deviations such as octanoylcarnitine (2.02 ppm). Write the five records to table1_acylcarnitine_identity_confirmation.csv with columns metabolite, measured_mz, cas_number, mass_error_ppm, rt_error, and confirm every reproduced value matches the published table entry.

## Examples

```
pd.read_excel('scoary2_table1.xlsx')[['metabolite','measured_mz','cas_number','mass_error_ppm','rt_error']].to_csv('table1_acylcarnitine_identity_confirmation.csv', index=False)
```

## Evaluation signals

- Each reproduced value (measured_mz, cas_number, mass_error_ppm, rt_error) matches the published Table 1 entry (output_matches_reference).
- All reported mass errors are checked against the <1 ppm accuracy threshold and octanoylcarnitine (2.02 ppm) is flagged as the deviating entry.
- The output CSV contains all five metabolite names: decanoylcarnitine, octanoylcarnitine, hexanoylcarnitine, carnitine, acetylcarnitine.
- The CSV schema has exactly the columns metabolite, measured_mz, cas_number, mass_error_ppm, rt_error with no missing m/z, CAS, or error fields.
- Spot-check example values: decanoylcarnitine m/z 316.24764 / CAS 3992-45-8 / RT error 0.67%; hexanoylcarnitine m/z 260.18515 / CAS 22671-29-0 / RT error 2.13%.

## Limitations

- Only three of the five carnitine metabolites (decanoylcarnitine, octanoylcarnitine, hexanoylcarnitine) had full confirmation values surfaced in the source card; carnitine and acetylcarnitine records must be sourced directly from the published Table 1.
- Not all mass errors satisfy the <1 ppm threshold — octanoylcarnitine reports 2.02 ppm, so threshold checks must flag rather than assume universal compliance.
- Retention-time errors are reported as percentages rather than absolute time deviations, limiting direct comparison with studies reporting RT deviation in minutes.

## Evidence

- [supplementary] Table 1 confirms decanoylcarnitine (measured m/z 316.24764, CAS 3992-45-8, mass error <1 ppm, retention-time error 0.67%): [evidence span withheld — anchor status: not_found]
- [methods] Check each mass error against the reported <1 ppm accuracy threshold and flag any entry that deviates: [evidence span withheld — anchor status: not_found]
- [abstract] we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism: "we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism"
