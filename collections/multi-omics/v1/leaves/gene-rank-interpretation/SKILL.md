---
name: gene-rank-interpretation
description: Use when a seed-gene set (e.g. jasmonate genes AT2G44810 and AT1G17420)
  has been run through RWR_LOE or RWR_CV on a multiplex and you must judge rank plausibility,
  functional enrichment, or extract/visualize the top-ranked neighborhood.
when_to_use_negative:
- The input is an expression or GWAS summary-statistics table and the goal is statistical
  ranking rather than topological connectivity — RWR ranks genes only by network proximity
  to seeds.
- Network layers have not yet been assembled into a multiplex object; RWR_make_multiplex
  must be run first before any ranking interpretation.
- You need to evaluate network construction quality or layer similarity rather than
  gene ranks — RWR_netstats (including pairwise between-layer Jaccard similarity)
  is the appropriate tool.
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: RWRtoolkit
  role: Provides RWR_LOE (seed-relative scoring and ranking of all multiplex genes,
    cyto top-k subnetwork export) and RWR_CV (k-fold/LOO cross-validation mean ranks)
  repo: https://github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Underlying engine implementing RWR on multiplex networks whose ranking output
    RWRtoolkit extends
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: Cytoscape
  role: Receives the top-50 ranked subnetwork via RCy3 for visual inspection with
    edges colored by layer
- name: KBase
  role: Web-application route for running RWR_LOE and RWR_CV and visualizing subnetworks
    reproducibly
merged_aliases:
- rank-based-gene-prioritization-interpretation
merged_alias_records:
- alias: rank-based-gene-prioritization-interpretation
  slug: rank-based-gene-prioritization-interpretation
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:36aa3165020dfc661b9c2d223155c64c5614480873f86b679dbc10d7e762d0a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - Cytoscape
  - KBase
  - R
  - RWRtoolkit
  - RandomWalkRestartMH
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# gene-rank-interpretation

## Summary

Interpret ranked gene outputs from Random Walk with Restart (RWR) on a multiplex network: RWR_LOE scores and ranks every multiplex gene relative to a seed set, and RWR_CV cross-validation mean ranks reveal whether a curated gene set is topologically more connected than a background set. Ranks are validated against functional coherence (e.g. MAPMAN jasmonate genes) and visualized as top-k subnetworks.

## When to use

Use when a seed-gene set (e.g. jasmonate genes AT2G44810 and AT1G17420) has been run through RWR_LOE or RWR_CV on a multiplex and you must judge rank plausibility, functional enrichment, or extract/visualize the top-ranked neighborhood.

## When NOT to use

- The input is an expression or GWAS summary-statistics table and the goal is statistical ranking rather than topological connectivity — RWR ranks genes only by network proximity to seeds.
- Network layers have not yet been assembled into a multiplex object; RWR_make_multiplex must be run first before any ranking interpretation.
- You need to evaluate network construction quality or layer similarity rather than gene ranks — RWR_netstats (including pairwise between-layer Jaccard similarity) is the appropriate tool.

## Inputs

- Two-column seed-geneset file (set/gene format), e.g. jasmonate seeds AT2G44810 and AT1G17420
- Preassembled multiplex network RData object (e.g. Comprehensive_Network_AT_d0.5_v02.RData, 9 layers, delta = 0.5)
- Gold-set file for cross-validation (30 genes: 27 MAPMAN 'Jasmonate' genes + AT1G08380, AT4G33360, AT5G23010)
- RWR_LOE/RWR_CV output tables of full ranks and mean ranks

## Outputs

- Ranked gene output table with random walk scores and ranks for every multiplex gene
- Per-gene mean ranks across cross-validation folds (full ranks, mean ranks, metrics, summary files)
- Top-50 subnetwork exported to Cytoscape via RCy3 with edges colored by layer name

## How to apply

Load a preassembled multiplex (e.g. the 9-layer comprehensive Arabidopsis thaliana network built with delta = 0.5) via RWRtoolkit::RWR_make_multiplex, then either run RWR_LOE with a two-column seed-geneset file and restart value 0.7 to obtain random-walk scores and ranks for every gene in the multiplex, or run RWR_CV with method = 'kfold', k = 5, restart 0.7 to obtain per-gene mean ranks across folds. Interpret ranks against functional expectations: in the article's 30-gene gold set (27 MAPMAN 'Jasmonate' genes plus non-jasmonate genes AT1G08380, AT4G33360, AT5G23010), 23 of 27 jasmonate genes fell within the top 100 mean ranks while non-jasmonate genes ranked near 2600/9425/9707, indicating topological connectivity tracks shared function. For context extraction, set cyto = 50 to isolate the subnetwork of the 50 highest-ranked nodes surrounding the seeds and export it to Cytoscape via RCy3 with edges colored by layer name, so each high-ranking interaction can be traced back to its evidence layer (coexpression, PPI, regulation, etc.).

## Related tools

- **RWRtoolkit** (Provides RWR_LOE (seed-relative scoring and ranking of all multiplex genes, cyto top-k subnetwork export) and RWR_CV (k-fold/LOO cross-validation mean ranks)) — https://github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Underlying engine implementing RWR on multiplex networks whose ranking output RWRtoolkit extends) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **Cytoscape** (Receives the top-50 ranked subnetwork via RCy3 for visual inspection with edges colored by layer)
- **KBase** (Web-application route for running RWR_LOE and RWR_CV and visualizing subnetworks reproducibly)

## Examples

```
Rscript <LIBPATHS_DIRECTORY>/RWRtoolkit/scripts/run_loe.R --data <LIBPATHS_DIRECTORY>/RWRtoolkit/example_data/string_interactions.Rdata --seed_geneset <LIBPATHS_DIRECTORY>/RWRtoolkit/example_data/geneset1.txt --tau "1.0,1.0" -o ./outdir
```

## Evaluation signals

- Ranked output file exists and contains scores/ranks for all genes in the multiplex.
- Exported subnetwork contains exactly 50 nodes when cyto = 50.
- In the 5-fold RWR_CV gold-set test, 23 of 27 MAPMAN jasmonate genes fall within the top 100 mean ranks; remaining jasmonate genes appear at ranks 128/246/1517/1947.
- The three non-jasmonate control genes (AT1G08380, AT4G33360, AT5G23010) rank near 2600/9425/9707, far below jasmonate genes.
- AUROC is well above the 0.50 random baseline, and top-ranked genes are functionally related (jasmonate-associated) to the seeds.

## Limitations

- Tau seed-weighting was corrected in RWRtoolkit; in RandomWalkRestartMH, tau incorrectly weighted each seed equally across all layers, leading to the conclusion that tau had minimal effect on output rankings.
- Aggregating layers into a single network loses the unique topological information carried by each layer, so ranked context should be interpreted per evidence layer.
- No version number is stated for RWRtoolkit or RandomWalkRestartMH, and R >= 4.1.0 with devtools is required for installation.

## Evidence

- [methods] RWR_LOE with a seed set file containing the seed genes of AT2G44810 and AT1G17420: "We used the RWR_LOE function with a seed set file containing the seed genes of AT2G44810 and AT1G17420."
- [discussion] RWR scores topological connectivity from multiple lines of evidence: "RWRtoolkit uses RWR to explore and score topological connectivity from multiple lines of evidence between seed genes and other genes."
- [discussion] MAPMAN jasmonate set demonstrates RWR_CV LOO power: "Using a set of genes sharing the jasmonate signalling term from the MAPMAN database, we see a small example of the power of RWR_CV’s LOO method."
- [readme] README: RWRtoolkit enables RWR on multiplex networks: "RWRtoolkit enables easy use of RandomWalk with Restart on multiplex networks. These functions are an extension to the [RandomWalkRestartMH](https://github.com/alberto-valdeolivas/RandomWalkRestartMH)"
- [abstract] Abstract: functions for topological context around sets of interest: "A range of functions can be used to find topological distances between biological entities, determine relationships within sets of interest, search for topological context around sets of interest,"
