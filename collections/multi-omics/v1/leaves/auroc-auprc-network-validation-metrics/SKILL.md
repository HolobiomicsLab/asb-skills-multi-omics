---
name: auroc-auprc-network-validation-metrics
description: Use when a multiplex network (e.g. GO_AT_d0.5_v01 built with delta =
  0.5) must be validated by showing that a curated gene set such as 27 MAPMAN 'Jasmonate'
  genes ranks topologically higher than negative controls in RWR_CV.
when_to_use_negative:
- When the goal is gene-function prediction or ranking of novel candidates rather
  than validating network quality — use RWR_LOE instead of RWR_CV
- When the gene set is not curated (e.g. not expert-annotated as in MAPMAN), since
  cross-validation recovery metrics are uninterpretable for arbitrary seeds
- When the network is a single aggregated/summarized graph rather than a multiplex,
  since layer-specific topological information — the basis of the validation — is
  lost
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RWRtoolkit
  role: Provides RWR_CV (k-fold and LOO cross-validation of gene set interconnectivity)
    and RWR_make_multiplex for building the mpo multiplex object
  repo: https://github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Underlying engine implementing RWR on the multiplex supra-adjacency structure
    that RWR_CV wraps
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: R
  role: Environment in which RWR_CV is run and output rank/metric files are parsed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:757da2110d5d606e5632a441478b06f0203e4411a6c0c029a1ceafac514e53b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - R
  - RWRtoolkit
  - RandomWalkRestartMH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# auroc-auprc-network-validation-metrics

## Summary

Evaluates the functional predictiveness of a multiplex biological network by running Random Walk with Restart cross-validation (RWR_CV) on curated gene sets and interpreting per-gene mean ranks plus AUROC/AUPRC against the 0.50 random baseline, as demonstrated on the 9-layer Arabidopsis multiplex with MAPMAN jasmonate genes.

## When to use

Use when a multiplex network (e.g. GO_AT_d0.5_v01 built with delta = 0.5) must be validated by showing that a curated gene set such as 27 MAPMAN 'Jasmonate' genes ranks topologically higher than negative controls in RWR_CV.

## When NOT to use

- When the goal is gene-function prediction or ranking of novel candidates rather than validating network quality — use RWR_LOE instead of RWR_CV
- When the gene set is not curated (e.g. not expert-annotated as in MAPMAN), since cross-validation recovery metrics are uninterpretable for arbitrary seeds
- When the network is a single aggregated/summarized graph rather than a multiplex, since layer-specific topological information — the basis of the validation — is lost

## Inputs

- Multiplex network RData object (e.g. GO_AT_d0.5_v01 Arabidopsis multiplex with delta = 0.5) loadable via RWRtoolkit::RWR_make_multiplex
- Two-column set/gene gold-set file (30 genes: 27 MAPMAN 'Jasmonate' genes plus non-jasmonate controls AT1G08380, AT4G33360, AT5G23010)
- Cross-validation parameters: method = 'kfold', k = 5, restart = 0.7

## Outputs

- Four RWR_CV output files per run: full ranks, mean ranks, metrics, and summary
- Per-gene mean-rank table split by jasmonate vs non-jasmonate genes
- AUROC/AUPRC metrics compared against the 0.50 random baseline

## How to apply

Load the multiplex (e.g. GO_AT_d0.5_v01, delta = 0.5) into an mpo object with RWRtoolkit::RWR_make_multiplex, then write the seed gene set in a two-column set/gene file combining the 27 MAPMAN jasmonate genes with 3 non-jasmonate negative controls (AT1G08380, AT4G33360, AT5G23010). Run RWRtoolkit::RWR_CV with method = 'kfold', k = 5, and restart = 0.7, which generates four output files (full ranks, mean ranks, metrics, summary). Parse the per-gene mean ranks and compare the within-set jasmonate genes against the out-of-set non-jasmonate controls; a functionally predictive multiplex yields an AUROC well above the 0.50 random baseline. The rationale is that genes sharing a functional annotation (jasmonate signalling in MAPMAN) should have strong topological similarity across the network layers, so cross-validation recovery of held-out seeds measures whether the multiplex encodes meaningful biology rather than noise.

## Related tools

- **RWRtoolkit** (Provides RWR_CV (k-fold and LOO cross-validation of gene set interconnectivity) and RWR_make_multiplex for building the mpo multiplex object) — https://github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Underlying engine implementing RWR on the multiplex supra-adjacency structure that RWR_CV wraps) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **R** (Environment in which RWR_CV is run and output rank/metric files are parsed)

## Examples

```
Rscript run_cv.R --data GO_AT_d0.5_v01.Rdata --seed_geneset jasmonate_gold30.txt --method kfold --k 5 --restart 0.7 -o ./cv_results
```

## Evaluation signals

- 23 of the 27 jasmonate genes fall within the top 100 mean ranks in the 5-fold RWR_CV ranking
- Remaining jasmonate genes appear at ranks 128/246/1517/1947 while non-jasmonate controls rank near 2600/9425/9707 — a large separation between positives and negatives
- AUROC is well above the 0.50 random baseline, consistent with a functionally predictive multiplex
- All four expected output files (full ranks, mean ranks, metrics, summary) are generated and parseable
- In the LOO variant, all 30 individual leave-one-out tests complete (one gene held out each time)

## Limitations

- RWR_CV requires carefully curated gene sets; results are only as meaningful as the annotation source (e.g. MAPMAN expert curation)
- Some jasmonate genes rank far below the top 100 (up to rank 1947), so within-set connectivity is strong but not uniform
- A prior limitation in RandomWalkRestartMH incorrectly weighted seeds equally across layers via tau, so restart/tau behaviour differs between the base package and RWRtoolkit
- The paper notes no version number stated for RWRtoolkit or RandomWalkRestartMH, complicating exact reproducibility

## Evidence

- [discussion] Cross-validation on curated gene sets demonstrates network predictiveness: "The example cross-validation experiment conducted on our 9-layer comprehensive Arabidopsis multiplex demonstrates that functionally related groups of genes have strong topological similarity"
- [methods] RWR_CV is the cross-validation function used: "we employed Random Walk with Restart Cross-Validation (RWR_CV) using a series of carefully curated gene sets"
- [discussion] Jasmonate MAPMAN gene set powers the LOO demonstration: "Using a set of genes sharing the jasmonate signalling term from the MAPMAN database, we see a small example of the power of RWR_CV’s LOO method."
- [abstract] RWRtoolkit is used for RWR-based multiplex analysis: "We introduce RWRtoolkit, a multiplex generation, exploration, and statistical package built for R and command-line users."
- [readme] CLI invocation pattern for RWRtoolkit scripts: "Rscript <LIBPATHS_DIRECTORY>/RWRtoolkit/scripts/run_loe.R 
    --data            <LIBPATHS_DIRECTORY>/RWRtoolkit/example_data/string_interactions.Rdata \"
