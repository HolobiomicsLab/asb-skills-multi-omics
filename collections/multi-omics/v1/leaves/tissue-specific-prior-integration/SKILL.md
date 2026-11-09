---
name: tissue-specific-prior-integration
description: Use when recovering missing proteins from a DDA-PASEF or similar proteomics
  expression matrix and the sample's tissue/cell line (e.g. lung/A549, HeLa) is known,
  so complex-based inference can be conditioned on tissue-specific complex confidence.
when_to_use_negative:
- The tissue or cell line of the sample is unknown or not among the 27 supported tissues
  — the prior scores would be misleading; the README suggests defaulting to 'lung'
  only if unsure, but results should be interpreted cautiously.
- The analysis requires protein abundance, modification status, or subcellular localization
  — Protrec2 operates on a binary presence/absence basis and does not incorporate
  these.
- Recovering proteins from novel or condition-specific complexes not represented in
  the curated complex repository (CORUM/Complex Portal/Reactome/STRING), which remains
  incomplete.
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0128
tools:
- name: Protrec2
  role: Runs the Bayesian recovery that consumes the tissue-specific complex prior
    as one of three logit-averaged evidence layers; prior selected via --tissue
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Execution environment for main.py and loading of the expression matrix and
    tissue complex CSVs
- name: pip
  role: Installs Protrec2 dependencies via requirements.txt before running the tissue-prior
    workflow
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:16ad3e764cf64e8e1c7b072f07162ea2a04ef3cfd1cfd49d379a7517f5f21361
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/miaomiao6606/Protrec2
  repo_url: https://github.com/miaomiao6606/Protrec2
  tools:
  - PROTREC
  - Protrec2
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# tissue-specific-prior-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrates built-in tissue-specific protein complex annotations (updated_tissue_complex_*.csv) as a prior evidence layer in Protrec2's Bayesian inference, so that recovery of missing proteins is weighted by tissue relevance rather than generic CORUM-only complex support.

## When to use

Use when recovering missing proteins from a DDA-PASEF or similar proteomics expression matrix and the sample's tissue/cell line (e.g. lung/A549, HeLa) is known, so complex-based inference can be conditioned on tissue-specific complex confidence.

## When NOT to use

- The tissue or cell line of the sample is unknown or not among the 27 supported tissues — the prior scores would be misleading; the README suggests defaulting to 'lung' only if unsure, but results should be interpreted cautiously.
- The analysis requires protein abundance, modification status, or subcellular localization — Protrec2 operates on a binary presence/absence basis and does not incorporate these.
- Recovering proteins from novel or condition-specific complexes not represented in the curated complex repository (CORUM/Complex Portal/Reactome/STRING), which remains incomplete.

## Inputs

- Expression matrix CSV (rows: UniProt IDs or gene symbols, columns: samples, non-negative numeric values, e.g. example/example_expression.csv)
- Built-in tissue-specific complex prior: updated_tissue_complex_*.csv files (tissue parameter, e.g. 'lung')
- Tissue label for the sample/cell line (e.g. lung, brain, breast)
- Parameters: --fdr 0.01, --threshold 5

## Outputs

- Per-sample CSV files ({output_dir}/PROTREC2_result_{sample_name}.csv) with columns Protein,Probability (probabilities in [0,1])
- Tissue-conditioned recovery of missing proteins whose confidence meets the cutoff (e.g. 0.95)

## How to apply

Select the tissue-specific complex prior matching the sample's tissue from the built-in updated_tissue_complex_*.csv files (27 tissues available, e.g. 'lung' for A549) and pass it via the --tissue parameter of main.py. The prior assigns tissue relevance scores (1.0 explicit tissue support, 0.5 ambiguous, 0.0 explicitly non-relevant) to complexes built from CORUM (size >= 5), Complex Portal, Reactome, and STRING-expanded interactions (score > 780; 3,885 complexes total), combined with GO-based functional coherence. This tissue-specific complex confidence is then averaged in logit space with the prior detection probability (1 - FDR for observed proteins, 0.1 for unobserved; --fdr 0.01) and cocomplex structural support (complexes passing --threshold 5) using equal weights alpha = beta = gamma = 1/3. The rationale is that PROTREC's reliance on CORUM alone (only 694 human complexes of size >= 5) and its ignorance of tissue expression variation limit sensitivity; conditioning on tissue priors recovers phenotypically relevant differential proteins.

## Related tools

- **Protrec2** (Runs the Bayesian recovery that consumes the tissue-specific complex prior as one of three logit-averaged evidence layers; prior selected via --tissue) — https://github.com/miaomiao6606/Protrec2
- **Python** (Execution environment for main.py and loading of the expression matrix and tissue complex CSVs)
- **pip** (Installs Protrec2 dependencies via requirements.txt before running the tissue-prior workflow)

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- The chosen tissue name is accepted by main.py (--tissue lung for A549; check --help lists the 27 available tissues) and the corresponding updated_tissue_complex_*.csv prior loads without error.
- Output CSVs parse with exactly the columns 'Protein' and 'Probability', contain probabilities within [0,1], and include only proteins meeting the 0.95 confidence threshold.
- Benchmark reproduction: with the lung prior on the A549 benchmark, lower-bound recovery rate reaches ~90.8% (vs PROTREC 81.8%, FCS 82.8%, HE 80.1%, GSEA 73.8%) and upper-bound ~98.4%.
- Recovery improves monotonically as the probability cutoff increases from 0.50 to 0.95, consistent with the reported threshold sensitivity analysis.
- Validated-protein counts match published benchmarks, e.g. 650 proteins validated in A549 and 453 in HeLa upper-bound evaluations against diaPASEF references.

## Limitations

- Tissue-specific annotations were derived from population-level data and might not fully reflect interpatient heterogeneity.
- The protein complex repository remains incomplete for novel or condition-specific complexes.
- Protrec2 operates on a binary presence/absence basis and does not incorporate protein abundance, modification status, or subcellular localization.
- Validation performance is constrained by proteomics dataset coverage, so reported recovery rates represent lower bounds on true performance.

## Evidence
