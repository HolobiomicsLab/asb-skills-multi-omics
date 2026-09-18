---
name: retention-time-error-assessment
description: Use when metabolite identities detected in an LC-MS metabolome (e.g.
  carnitine species) must be confirmed by pure-standard injection and reported as
  a table of m/z, CAS numbers, and mass/retention-time errors.
when_to_use_negative:
- The input is already a validated identity-confirmation table — re-deriving errors
  would be redundant
- The metabolite of interest has no commercially available pure analytical standard,
  so no standard-injection comparison can be made
- The goal is pan-genome association (Scoary2's core analysis) rather than metabolite
  identity confirmation
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Scoary2
  role: Source study whose yogurt-metabolome workflow produced the LC-MS standard-confirmation
    table (Table 1) being reproduced
  repo: https://github.com/MrTomRod/scoary-2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:0a2f35921a087b05bf42403efa2bf6b86a6600b2dc06af6f1607d3a06bf8b7ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - Scoary2
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# retention-time-error-assessment

## Summary

Confirms the identity of detected metabolite features (here carnitine-related species in a yogurt metabolome) by comparing LC-MS measurements of pure analytical standards against detected peaks, recording measured m/z, CAS number, mass error in ppm, and retention-time error as a percentage.

## When to use

Use when metabolite identities detected in an LC-MS metabolome (e.g. carnitine species) must be confirmed by pure-standard injection and reported as a table of m/z, CAS numbers, and mass/retention-time errors.

## When NOT to use

- The input is already a validated identity-confirmation table — re-deriving errors would be redundant
- The metabolite of interest has no commercially available pure analytical standard, so no standard-injection comparison can be made
- The goal is pan-genome association (Scoary2's core analysis) rather than metabolite identity confirmation

## Inputs

- Supplementary Table 1 (identity confirmation of carnitine species by LC-MS of pure analytical standard solutions)
- LC-MS measured m/z values, CAS numbers, mass errors (ppm), and retention-time errors for decanoylcarnitine, octanoylcarnitine, hexanoylcarnitine, carnitine, and acetylcarnitine

## Outputs

- table1_acylcarnitine_identity_confirmation.csv with columns metabolite, measured_mz, cas_number, mass_error_ppm, rt_error
- Per-metabolite flag indicating whether the mass error meets the <1 ppm threshold

## How to apply

Retrieve the published supplementary Table 1 that records the identity confirmation of carnitine species (decanoylcarnitine, octanoylcarnitine, hexanoylcarnitine, carnitine, acetylcarnitine) from the Scoary2 yogurt metabolome study. For each metabolite, extract the measured m/z (e.g. 316.24764 for decanoylcarnitine, 288.21635 for octanoylcarnitine, 260.18515 for hexanoylcarnitine), the CAS number, the mass error in ppm, and the retention-time error percentage using pandas. Check each mass error against the reported accuracy threshold of <1 ppm (octanoylcarnitine is a known exception at 2.02 ppm) and flag deviations. Retention-time errors reported by the standard injections are small percentages (0.67% for decanoylcarnitine, 0.47% for octanoylcarnitine, 2.13% for hexanoylcarnitine), so values in this range indicate the standard and the detected feature co-elute. Write the records to a CSV with columns metabolite, measured_mz, cas_number, mass_error_ppm, rt_error and validate each reproduced value against the published entry.

## Related tools

- **Scoary2** (Source study whose yogurt-metabolome workflow produced the LC-MS standard-confirmation table (Table 1) being reproduced) — https://github.com/MrTomRod/scoary-2

## Examples

```
import pandas as pd; df = pd.read_excel('table1_acylcarnitine_identity_confirmation.xlsx'); df.to_csv('table1_acylcarnitine_identity_confirmation.csv', columns=['metabolite','measured_mz','cas_number','mass_error_ppm','rt_error'], index=False)
```

## Evaluation signals

- Each reproduced measured m/z, CAS number, mass error, and retention-time error matches the published Table 1 entry (output_matches_reference)
- Mass errors are checked against the <1 ppm accuracy threshold; octanoylcarnitine's 2.02 ppm is the documented deviation to expect
- Retention-time errors fall in the small-percent range reported by the paper (0.47–2.13%)
- All five metabolite names (decanoylcarnitine, octanoylcarnitine, hexanoylcarnitine, carnitine, acetylcarnitine) plus m/z, CAS, and error fields are present in the output CSV
- Output CSV schema is exactly metabolite, measured_mz, cas_number, mass_error_ppm, rt_error

## Limitations

- One standard-confirmed metabolite (octanoylcarnitine, 2.02 ppm) exceeds the <1 ppm accuracy threshold, so a strict threshold check will flag a valid entry
- The discussion-section text of the source document is not available, so rationale beyond the table values and workflow steps cannot be extracted
- Confirmation is only possible for carnitine-related metabolites with pure analytical standards; other detected features cannot be validated this way

## Evidence

- [supplementary] Table 1 confirms decanoylcarnitine (measured m/z 316.24764, CAS 3992-45-8, mass error <1 ppm, retention-time error 0.67%): [evidence span withheld — anchor status: not_found]
- [supplementary] For each of decanoylcarnitine, octanoylcarnitine, hexanoylcarnitine, carnitine, and acetylcarnitine, extract the measured m/z, CAS number, mass error (ppm), and retention-time error from the published table using pandas.: [evidence span withheld — anchor status: not_found]
- [abstract] we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism: "we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism"
