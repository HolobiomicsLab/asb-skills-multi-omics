---
name: arabidopsis-multiplex-network-loading
description: Use when an analysis needs a ready-made, species-specific multiplex network
  for Arabidopsis thaliana — e.g., to run netstats, cross-validation, or LOE — instead
  of constructing one from raw layer files.
when_to_use_negative:
- You need network layers for a species other than Arabidopsis thaliana — the deposit
  is species-specific and RWRtoolkit is otherwise species agnostic, so build a custom
  multiplex with RWR_make_multiplex instead.
- You need weighted edges or a different delta (inter-layer weighting) than 0.5 —
  the preassembled objects are fixed to unweighted edges and delta = 0.5.
- You only need a small subset of evidence types not represented in the preassembled
  layer sets; construct a multiplex from your own flist of layer files.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0092
tools:
- name: RWRtoolkit
  role: R/CLI package that consumes the loaded multiplex object for netstats, CV,
    LOE, and shortest-path analyses
  repo: https://github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Underlying package providing the multiplex-object representation and RWR core
    functionality that RWRtoolkit extends
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: RWRtoolkit-data
  role: Repository hosting the preassembled Arabidopsis multiplex RData objects (comprehensive,
    close-to-phenotype, expression/regulation variants)
  repo: https://github.com/dkainer/RWRtoolkit-data
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:1e1f7d638d9d6907209b70da3900ca1f527d61047963c6f0d8da25261ebbbc1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - R
  - RWRtoolkit
  - RWRtoolkit-data
  - RandomWalkRestartMH
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# arabidopsis-multiplex-network-loading

## Summary

Load the preassembled comprehensive Arabidopsis thaliana multiplex network (9 layers, 26,605 unique nodes, 918,640 edges, delta = 0.5, unweighted edges) from the RWRtoolkit-data deposit as an RData object, so that RWRtoolkit functions (RWR_netstats, RWR_LOE, RWR_CV, etc.) can operate on it directly.

## When to use

Use when an analysis needs a ready-made, species-specific multiplex network for Arabidopsis thaliana — e.g., to run netstats, cross-validation, or LOE — instead of constructing one from raw layer files.

## When NOT to use

- You need network layers for a species other than Arabidopsis thaliana — the deposit is species-specific and RWRtoolkit is otherwise species agnostic, so build a custom multiplex with RWR_make_multiplex instead.
- You need weighted edges or a different delta (inter-layer weighting) than 0.5 — the preassembled objects are fixed to unweighted edges and delta = 0.5.
- You only need a small subset of evidence types not represented in the preassembled layer sets; construct a multiplex from your own flist of layer files.

## Inputs

- Preassembled multiplex RData object Comprehensive_Network_AT_d0.5_v02.RData from dkainer/RWRtoolkit-data
- R environment with R >= 4.1.0 and devtools
- RWRtoolkit R package (extension of RandomWalkRestartMH)

## Outputs

- In-memory R multiplex network object (list of igraph objects) with 9 layers, 26,605 unique nodes, 918,640 edges, delta = 0.5
- Optionally, per-layer statistics and pairwise Jaccard outputs (base_stats.tsv, pairwise_between_mpo_layer_jaccard.tsv) if RWR_netstats is subsequently run

## How to apply

Fetch the preassembled multiplex RData object 'Comprehensive_Network_AT_d0.5_v02.RData' (version 0.2) from the dkainer/RWRtoolkit-data deposit, which contains the 9 unweighted layers with delta = 0.5: CoEvolution-DUO (DU), Coexpression Gene-Atlas (GA), Knockout Similarity (KS), PPI-6merged (PP), PEN-Diversity (PX), Predictive CG Methylation (PY), Regulation-ATRM (RE), Regulation-Plantregmap (RP), and Metabolic-AraCyc (RX). Install RWRtoolkit (requires R >= 4.1.0, e.g. devtools::install_github('dkainer/RWRtoolkit')) and load the object in an R session so the RandomWalkRestartMH-style multiplex list of igraph objects is in memory. Choose between the comprehensive default multiplex, the 3-layer 'Close_To_Phenotype' multiplex (KS, PP, RX), or the Expression/Regulation multiplex depending on the biological question. Validate the loaded object by checking it has 9 layers and that per-layer sizes match the article's Table 2 (e.g., PPI-6merged: 19,191 nodes / 317,787 edges; totals of 26,605 unique nodes and 918,640 edges).

## Related tools

- **RWRtoolkit** (R/CLI package that consumes the loaded multiplex object for netstats, CV, LOE, and shortest-path analyses) — https://github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Underlying package providing the multiplex-object representation and RWR core functionality that RWRtoolkit extends) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **RWRtoolkit-data** (Repository hosting the preassembled Arabidopsis multiplex RData objects (comprehensive, close-to-phenotype, expression/regulation variants)) — https://github.com/dkainer/RWRtoolkit-data

## Examples

```
library(RWRtoolkit); load('Comprehensive_Network_AT_d0.5_v02.RData'); RWRtoolkit::RWR_netstats(multiplex_object, basic_statistics=TRUE)
```

## Evaluation signals

- Loaded object contains exactly 9 layers matching the named layer set (DU, GA, KS, PP, PX, PY, RE, RP, RX)
- Total unique node count is 26,605 and total edge count is 918,640 across the multiplex
- Per-layer sizes match Table 2 values, e.g. PPI-6merged has 19,191 nodes and 317,787 edges
- All layers are unweighted and the multiplex was constructed with delta = 0.5
- Object class is compatible with RandomWalkRestartMH-style multiplex (list of igraph objects) so RWRtoolkit functions accept it

## Limitations

- Preassembled objects are fixed versions (comprehensive network version 0.2) with unweighted edges and delta = 0.5; custom weighting or layer selection requires rebuilding the multiplex
- No version number stated for RWRtoolkit or RandomWalkRestartMH in the article, and no changelog was found, so exact software versions corresponding to the deposit are uncertain
- RWRtoolkit installation requires R >= 4.1.0 and can encounter SSL/certificate issues with Bioconductor and missing system libraries (harfbuzz, fribidi, libgit2) for devtools

## Evidence

- [readme] Comprehensive Network Multiplex (Default Multiplex Network): "Contains the following network layers with unweighted edges and delta value = 0.5"
- [discussion] Preassembled A. thaliana multiplexes available: "we have included RWR_LOE and RWR_CV as applications within KBase and provided preassembled Arabidopsis thaliana multiplex networks."
- [methods] RWR_netstats on comprehensive multiplex: "we used the RWR_netstats function to extract basic statistics using the option basic_statistics"
- [readme] Layer inventory of comprehensive multiplex: "CoEvolution-DUO (DU) version 0.1 (AT-UU-DU-67-AA-01)<br />Coexpression Gene-Atlas (GA) version 0.3 (AT-UU-GA-01-AA-01)"
- [readme] Close-to-phenotype alternative multiplex: [evidence span withheld — anchor status: not_found]
