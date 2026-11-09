---
name: mapman-annotation-gene-set-curation
description: Use when you need a curated, functionally coherent seed gene set (e.g.
  genes sharing a MAPMAN 'Jasmonate' term) to validate a multiplex network with RWR_CV,
  and you must write it as a two-column set/gene file with matched negative/loosely-connected
  control genes.
when_to_use_negative:
- The genes of interest do not share a single expert-curated annotation term (MAPMAN/GO),
  so cross-validation would not test functional connectivity
- The target network is not a multiplex built by RWR_make_multiplex; RWR_CV requires
  an mpo multiplex object
- You need de novo gene discovery rather than validation of an already-defined gene
  set — use RWR_LOE seed-ranking instead
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0102
tools:
- name: RWRtoolkit
  role: Provides RWR_CV (k-fold cross-validation of the curated gold set on the multiplex)
    and RWR_make_multiplex to load the Arabidopsis multiplex object
  repo: https://github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Underlying package whose RWR-on-multiplex functionality RWRtoolkit extends;
    supplies the random-walk machinery behind RWR_CV
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: MAPMAN
  role: Source of expert-curated annotation terms used to define the functionally
    coherent jasmonate gene set
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:78ebcc55d4989a48d36e7ca7dd8a15a25a2b03c2f214ae589d6f9d2566f5f715
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - MAPMAN
  - R
  - RWRtoolkit
  - RandomWalkRestartMH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mapman-annotation-gene-set-curation

## Summary

Curate an expert-annotated gold-standard gene set from MAPMAN terms (e.g. 27 Arabidopsis 'Jasmonate' genes plus deliberately added non-jasmonate controls) in the two-column set/gene format required by RWRtoolkit, so that RWR_CV can test whether the set is topologically connected in a multiplex network.

## When to use

Use when you need a curated, functionally coherent seed gene set (e.g. genes sharing a MAPMAN 'Jasmonate' term) to validate a multiplex network with RWR_CV, and you must write it as a two-column set/gene file with matched negative/loosely-connected control genes.

## When NOT to use

- The genes of interest do not share a single expert-curated annotation term (MAPMAN/GO), so cross-validation would not test functional connectivity
- The target network is not a multiplex built by RWR_make_multiplex; RWR_CV requires an mpo multiplex object
- You need de novo gene discovery rather than validation of an already-defined gene set — use RWR_LOE seed-ranking instead

## Inputs

- MAPMAN annotation table for Arabidopsis (genes sharing a MAPMAN term, e.g. 'Jasmonate')
- Gene identifiers for deliberately added non-jasmonate control genes (AT1G08380, AT4G33360, AT5G23010)
- Preassembled Arabidopsis thaliana multiplex network RData object (e.g. GO_AT_d0.5_v01, delta = 0.5) loaded via RWR_make_multiplex

## Outputs

- Two-column set/gene gold-set file listing the 27 MAPMAN jasmonate genes plus 3 non-jasmonate genes
- RWR_CV output files: full ranks, mean ranks, metrics, and summary, including per-gene mean ranks and AUROC

## How to apply

Select genes sharing a single MAPMAN term (here the MAPMAN 'Jasmonate' signalling term), giving 27 jasmonate genes, and add intentionally unrelated control genes (AT1G08380, AT4G33360, AT5G23010) to the same set for contrast. Write the 30 genes into a two-column set/gene file, the input format RWRtoolkit's geneset_path expects. Load the GO_AT_d0.5_v01 Arabidopsis multiplex (delta = 0.5) via RWRtoolkit::RWR_make_multiplex, then run RWRtoolkit::RWR_CV with method='kfold', k=5 and restart value 0.7, writing the four outputs (full ranks, mean ranks, metrics, summary) to a results directory. Judge curation quality by parsing per-gene mean ranks: a well-curated functionally connected set places most members in the top ranks while controls fall far lower, with AUROC well above the 0.50 random baseline.

## Related tools

- **RWRtoolkit** (Provides RWR_CV (k-fold cross-validation of the curated gold set on the multiplex) and RWR_make_multiplex to load the Arabidopsis multiplex object) — https://github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Underlying package whose RWR-on-multiplex functionality RWRtoolkit extends; supplies the random-walk machinery behind RWR_CV) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **MAPMAN** (Source of expert-curated annotation terms used to define the functionally coherent jasmonate gene set)

## Examples

```
RWRtoolkit::RWR_CV(geneset_path="jasmonate_goldset.txt", data_path="GO_AT_d0.5_v01.Rdata", method="kfold", k=5, r=0.7, outdir="results/")
```

## Evaluation signals

- 23 of the 27 jasmonate genes fall within the top 100 mean ranks in the 5-fold RWR_CV ranking
- The remaining jasmonate genes appear at ranks 128/246/1517/1947, while the three non-jasmonate genes rank near 2600/9425/9707
- AUROC from the RWR_CV metrics file is well above the 0.50 random baseline, consistent with a functionally predictive multiplex
- The gold-set file parses correctly as a two-column set/gene file with all 30 genes and no delimiter errors
- All four expected RWR_CV outputs (full ranks, mean ranks, metrics, summary) are generated in the results directory

## Limitations

- MAPMAN sets are expert-curated, so coverage is limited to terms experts have annotated; jasmonate members can still rank poorly (up to rank 1947), so not every gene in a curated term is topologically connected
- The version number of RWRtoolkit/RandomWalkRestartMH used is not stated, complicating exact reproduction
- RWR_CV evaluation depends on the multiplex composition; results are specific to the 9-layer comprehensive Arabidopsis multiplex with delta = 0.5 and restart 0.7

## Evidence

- [methods] MAPMAN gene sets based on expert curation: "These gene sets were based on shared MAPMAN annotations [28] that are curated by experts"
- [discussion] Jasmonate term used to demonstrate RWR_CV power: "Using a set of genes sharing the jasmonate signalling term from the MAPMAN database, we see a small example of the power of RWR_CV’s LOO method."
- [discussion] Cross-validation shows functional groups have strong topological similarity: "The example cross-validation experiment conducted on our 9-layer comprehensive Arabidopsis multiplex demonstrates that functionally related groups of genes have strong topological similarity"
- [readme] RWRtoolkit provides RWR on multiplex networks: "RWRtoolkit enables easy use of RandomWalk with Restart on multiplex networks. These functions are an extension to the [RandomWalkRestartMH](https://github.com/alberto-valdeolivas/RandomWalkRestartMH)"
