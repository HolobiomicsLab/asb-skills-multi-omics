---
name: arabidopsis-gene-identifier-handling
description: Use when seed or target gene sets for Arabidopsis multiplex network analysis
  are specified as AGI locus codes (ATxG#####) and must be formatted into RWRtoolkit
  geneset files or KBase FeatureSets that match multiplex node identifiers.
when_to_use_negative:
- The analysis targets a non-Arabidopsis species, since node identifiers will follow
  that species' gene naming and AGI codes will not resolve
- Input identifiers are not AGI locus codes (e.g. symbols, UniProt IDs) and no cross-reference
  table is available — RWRtoolkit itself does not perform identifier mapping
- Building a custom multiplex from scratch with non-gene entity types (e.g. metabolites),
  where node identifiers are not gene IDs
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: RWRtoolkit
  role: Consumes AGI-based geneset files as --seed_geneset / target set inputs for
    RWR_LOE and RWR_ShortestPaths against the A. thaliana multiplex
  repo: http://github.com/dkainer/RWRtoolkit
- name: KBase
  role: Hosts the 'Build FeatureSet from Genome' workflow that turns the NCBI RefSeq
    Arabidopsis genome into gene sets with AGI identifiers
- name: dkainer/RWRtoolkit-data
  role: Provides the preassembled A. thaliana multiplex RData objects whose layers
    are indexed by AGI gene identifiers
  repo: https://github.com/dkainer/RWRtoolkit-data
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:2289574728aff806ebf217e7b40b4d724ea8496b83bf793beecfd4f841aac721
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KBase
  - R
  - RWR_ShortestPaths
  - RWRtoolkit
  - RandomWalkRestartMH
  - dkainer/RWRtoolkit-data
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Arabidopsis gene identifier handling

## Summary

Handles Arabidopsis thaliana AGI locus identifiers (ATxG#####) when preparing seed/target gene sets for RWRtoolkit multiplex analysis, ensuring gene IDs match the identifiers used across the 9-layer comprehensive A. thaliana multiplex (e.g. AT2G44810, AT1G17420, AT1G48500).

## When to use

Use when seed or target gene sets for Arabidopsis multiplex network analysis are specified as AGI locus codes (ATxG#####) and must be formatted into RWRtoolkit geneset files or KBase FeatureSets that match multiplex node identifiers.

## When NOT to use

- The analysis targets a non-Arabidopsis species, since node identifiers will follow that species' gene naming and AGI codes will not resolve
- Input identifiers are not AGI locus codes (e.g. symbols, UniProt IDs) and no cross-reference table is available — RWRtoolkit itself does not perform identifier mapping
- Building a custom multiplex from scratch with non-gene entity types (e.g. metabolites), where node identifiers are not gene IDs

## Inputs

- Arabidopsis AGI locus identifiers (ATxG##### format, e.g. AT2G44810, AT1G17420, AT1G13280, AT1G48500, AT3G43440)
- RWRtoolkit geneset-format seed set file (e.g. geneset1.txt with seed genes)
- RWRtoolkit geneset-format target set file (e.g. targets for RWR_ShortestPaths)
- Arabidopsis_thaliana genome from NCBI RefSeq Genomes (for KBase FeatureSet construction)

## Outputs

- Seed geneset file in RWRtoolkit format containing validated AGI identifiers
- KBase FeatureSet built from the Arabidopsis RefSeq genome with AGI-consistent gene IDs
- Genes whose identifiers resolve against nodes of Comprehensive_Network_AT_d0.5_v02.RData

## How to apply

Represent each Arabidopsis gene by its AGI locus identifier in the standardized ATxG##### form (e.g. AT2G44810, AT1G17420 as seeds; AT1G48500, AT3G43440 as targets). Write these identifiers into RWRtoolkit geneset-format files (one set per file), relying on the flist/geneset delimiter conventions (comma, tab, pipe, or semicolon) documented in the README. When working through KBase, load the Arabidopsis_thaliana genome via the public database tab using NCBI RefSeq Genomes, then select 'Build FeatureSet from Genome' from the Comparative Genomics dropdown so that gene IDs in the FeatureSet agree with the preassembled multiplex node IDs. Before running RWR_LOE or RWR_ShortestPaths, verify that every identifier in the geneset file resolves to a node in the comprehensive multiplex (Comprehensive_Network_AT_d0.5_v02.RData), since unmatched seeds are silently dropped from the walk. The rationale is that the preassembled multiplex layers (DU, GA, KS, PP, PX, PY, RE, RP, RX) are indexed by these AGI codes, so identifier consistency is a prerequisite for meaningful rankings and shortest paths.

## Related tools

- **RWRtoolkit** (Consumes AGI-based geneset files as --seed_geneset / target set inputs for RWR_LOE and RWR_ShortestPaths against the A. thaliana multiplex) — http://github.com/dkainer/RWRtoolkit
- **KBase** (Hosts the 'Build FeatureSet from Genome' workflow that turns the NCBI RefSeq Arabidopsis genome into gene sets with AGI identifiers)
- **dkainer/RWRtoolkit-data** (Provides the preassembled A. thaliana multiplex RData objects whose layers are indexed by AGI gene identifiers) — https://github.com/dkainer/RWRtoolkit-data

## Examples

```
Rscript <LIBPATHS_DIRECTORY>/RWRtoolkit/scripts/run_loe.R --data <LIBPATHS_DIRECTORY>/RWRtoolkit/example_data/string_interactions.Rdata --seed_geneset <LIBPATHS_DIRECTORY>/RWRtoolkit/example_data/geneset1.txt --tau "1.0,1.0" -o ./outdir
```

## Evaluation signals

- Every gene identifier in the geneset file matches the AGI locus pattern (AT + chromosome digit + G + 5 digits), e.g. AT2G44810
- All seed identifiers resolve to nodes in the comprehensive multiplex object loaded via RWR_make_multiplex; the ranked RWR_LOE output table contains score/rank rows for the seeds and for every multiplex gene
- In KBase, the FeatureSet derived from 'Build FeatureSet from Genome' uses the same NCBI RefSeq-based AGI identifiers as the multiplex layers
- RWR_ShortestPaths returns edge-list records for all source-target pairs (2 sources x 2 targets) only when both gene sets' identifiers resolve in the multiplex; expected result was 4 paths of length 3
- Geneset files parse correctly with any of the documented delimiters (comma, tab, pipe, semicolon) in the flist/geneset reader

## Limitations

- RWRtoolkit provides no identifier mapping or translation functionality; genes not present in any multiplex layer cannot be scored
- The article does not state which genome annotation version ties AGI codes to features, so identifier-version mismatches across network layers are possible
- Seeds must exist in the loaded multiplex object — an AGI code present in the geneset but absent from the network layers will not contribute to rankings

## Evidence

- [methods] Seed set handling in RWR_LOE uses AGI seed genes: "We used the RWR_LOE function with a seed set file containing the seed genes of AT2G44810 and AT1G17420."
- [methods] Arabidopsis genome loaded from NCBI RefSeq in KBase: "The Arabidopsis_thaliana genome was loaded via the public database tab, using the NCBI RefSeq Genomes."
- [methods] KBase FeatureSet construction step: [evidence span withheld — anchor status: not_found]
- [readme] Geneset file delimiter conventions from README: [evidence span withheld — anchor status: not_found]
- [readme] Preassembled multiplex layers indexed by gene identifiers: [evidence span withheld — anchor status: not_found]
