---
name: random-walk-cross-validation-scoring
description: Use when you have a curated gene set (e.g. MAPMAN or GO terms) and a
  multiplex network, and want to test whether the set's members are topologically
  more connected than the rest of the network via RWR cross-validation.
when_to_use_negative:
- The question is gene-to-gene connectivity ranking of specific seeds against all
  genes, not held-out validation — use RWR_LOE instead of RWR_CV
- The input is a single-layer network where multiplex structure (layer-specific topological
  information) is irrelevant
- The gene set is not curated (e.g. no shared MAPMAN/GO annotation), so a high rank
  cannot be interpreted as functional predictiveness
edam_operation: http://edamontology.org/operation_2945
tools:
- name: RWRtoolkit
  role: Provides RWR_CV (k-fold/LOO cross-validation) and RWR_make_multiplex used
    to build the multiplex and score gene set interconnectivity
  repo: https://github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Underlying engine implementing RWR on multiplex networks that RWRtoolkit builds
    on
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: R
  role: Environment in which the RWRtoolkit package functions and command-line scripts
    are run
  repo: https://github.com/dkainer/RWRtoolkit
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:6bb7a1c5dec2cae1f1c886c7ffda3401a470ba0d2f37e2ad14efb28e03be45d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - R
  - RWRtoolkit
  - RandomWalkRestartMH
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# random-walk-cross-validation-scoring

## Summary

Score the topological interconnectivity of a curated gene set within a multiplex network by running Random Walk with Restart cross-validation (RWR_CV, k-fold or LOO) via RWRtoolkit, then check that held-out seed genes rank highly — demonstrating functionally related genes have strong network connectivity.

## When to use

Use when you have a curated gene set (e.g. MAPMAN or GO terms) and a multiplex network, and want to test whether the set's members are topologically more connected than the rest of the network via RWR cross-validation.

## When NOT to use

- The question is gene-to-gene connectivity ranking of specific seeds against all genes, not held-out validation — use RWR_LOE instead of RWR_CV
- The input is a single-layer network where multiplex structure (layer-specific topological information) is irrelevant
- The gene set is not curated (e.g. no shared MAPMAN/GO annotation), so a high rank cannot be interpreted as functional predictiveness

## Inputs

- Multiplex network RData/mpo object (e.g. GO_AT_d0.5_v01 Arabidopsis multiplex, delta=0.5)
- Gold set gene file in two-column set/gene format (curated gene set plus optional negative-control genes)
- flist file defining multiplex layers (file_path, name, group per row)

## Outputs

- Full per-gene ranks file from RWR_CV
- Mean ranks file with per-gene mean rank across CV folds
- Metrics file (e.g. AUROC per gene/set)
- Summary file of cross-validation results

## How to apply

Build the multiplex network object first with RWRtoolkit::RWR_make_multiplex from an flist defining each network layer (file_path, name, group triplets). Write the gold set in two-column set/gene format, optionally adding negative-control genes (e.g. non-jasmonate genes AT1G08380, AT4G33360, AT5G23010 alongside 27 MAPMAN Jasmonate genes). Run RWRtoolkit::RWR_CV with either method='kfold' (e.g. k=5, restart r=0.7) or leave-one-out (30 genes yield 30 individual RWR tests, leaving one gene out each time). Parse the four generated output files (full ranks, mean ranks, metrics, summary) and extract per-gene mean ranks for the seed genes. Validate that most seed genes fall in the top 100 mean ranks (e.g. 23 of 27 jasmonate genes), with negative controls ranked far lower (e.g. ~2600/9425/9707), and that AUROC sits well above the 0.50 random baseline.

## Related tools

- **RWRtoolkit** (Provides RWR_CV (k-fold/LOO cross-validation) and RWR_make_multiplex used to build the multiplex and score gene set interconnectivity) — https://github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Underlying engine implementing RWR on multiplex networks that RWRtoolkit builds on) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **R** (Environment in which the RWRtoolkit package functions and command-line scripts are run) — https://github.com/dkainer/RWRtoolkit

## Examples

```
RWRtoolkit::RWR_CV(geneset_path="goldset_30genes.txt", flist="flist.txt", method="kfold", k=5, r=0.7, outdir="./results")
```

## Evaluation signals

- 23 of 27 jasmonate gold-set genes fall within the top 100 mean ranks under 5-fold RWR_CV
- Remaining jasmonate genes appear at mean ranks 128, 246, 1517, and 1947
- Non-jasmonate negative-control genes (AT1G08380, AT4G33360, AT5G23010) rank near 2600, 9425, and 9707
- AUROC for the gene set is well above the 0.50 random baseline
- Four output files (full ranks, mean ranks, metrics, summary) are generated and parseable per RWR_CV run

## Limitations

- RandomWalkRestartMH seeds were previously incorrectly weighted equally across all layers (tau had minimal effect on output rankings), so tau settings must be interpreted with care in that base implementation
- Results depend on the multiplex construction (e.g. delta=0.5, layer composition), so cross-validation scores are not transferable across differently built multiplexes
- No version number stated for RWRtoolkit or RandomWalkRestartMH, complicating reproducibility of exact ranks

## Evidence

- [methods] we employed Random Walk with Restart Cross-Validation (RWR_CV) using a series of carefully curated gene sets: "we employed Random Walk with Restart Cross-Validation (RWR_CV) using a series of carefully curated gene sets"
- [discussion] LOO per-gene test count: "Given that there are 30 genes within the set, 30 individual tests of RWR were run, leaving only one gene out each time."
- [discussion] CV demonstrates topological similarity: "The example cross-validation experiment conducted on our 9-layer comprehensive Arabidopsis multiplex demonstrates that functionally related groups of genes have strong topological similarity"
- [readme] make multiplex first: [evidence span withheld — anchor status: not_found]
- [discussion] MAPMAN jasmonate example: "Using a set of genes sharing the jasmonate signalling term from the MAPMAN database, we see a small example of the power of RWR_CV’s LOO method."
