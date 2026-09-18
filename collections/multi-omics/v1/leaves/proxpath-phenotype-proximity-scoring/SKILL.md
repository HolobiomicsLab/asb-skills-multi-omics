---
name: proxpath-phenotype-proximity-scoring
description: Use when a validated context-specific signaling network (nodes/edges
  from Step 2) must be linked to cellular phenotypes, and phenotype activation/deactivation
  signs need to be inferred from upstream protein activities.
when_to_use_negative:
- The analysis requires integration of metabolomic data — SignalingProfiler 2.0 does
  not include metabolomic data (COSMOS does)
- Input is raw omics data rather than a Step 2-optimized context-specific network
  and regulator activities — run Steps 1-2 first
- Phenotypes of interest are not annotated in SIGNOR, since ProxPath estimates regulatory
  impact on phenotypes annotated in SIGNOR
edam_operation: http://edamontology.org/operation_0259
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ProxPath
  role: Graph-based algorithm that estimates the regulatory impact of proteins on
    SIGNOR-annotated phenotypes and selects significantly close phenotypes (zscore_threshold,
    stat)
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: SignalingProfiler 2.0
  role: 'Pipeline hosting Step 3: computes the PhenoScore from upstream regulator
    activities and maps up to 200 cellular phenotypes onto the final model'
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: CARNIVAL
  role: ILP optimization used by optimize_pheno_network to re-optimize the final model
    on phenotypic activity with sign-coherent interactions
  repo: https://saezlab.github.io/CARNIVAL/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:b212c72cfeadb38def2f452f1af30edc2fc4144d86be7d7b5df68dc3e312c562
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - CARNIVAL
  - ProxPath
  - SignalingProfiler 2.0
  edam_operation: http://edamontology.org/operation_0259
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# proxpath-phenotype-proximity-scoring

## Summary

Step 3 of SignalingProfiler 2.0: connect a CARNIVAL-optimized context-specific network to SIGNOR-annotated cellular phenotypes using the ProxPath graph-proximity algorithm, then score phenotype activity as a weighted PhenoScore over upstream regulator activities.

## When to use

Use when a validated context-specific signaling network (nodes/edges from Step 2) must be linked to cellular phenotypes, and phenotype activation/deactivation signs need to be inferred from upstream protein activities.

## When NOT to use

- The analysis requires integration of metabolomic data — SignalingProfiler 2.0 does not include metabolomic data (COSMOS does)
- Input is raw omics data rather than a Step 2-optimized context-specific network and regulator activities — run Steps 1-2 first
- Phenotypes of interest are not annotated in SIGNOR, since ProxPath estimates regulatory impact on phenotypes annotated in SIGNOR

## Inputs

- Step 2 validated network object (nodes_df, edges_df, igraph_network, sp_graph)
- ProxPath protein-phenotype distance table preprocessed against the quantified proteome and phosphoproteome
- Inferred regulator (kinase/TF/protein) activity estimates from Step 1
- Phenotypic gold standard table for benchmarking PhenoScore predictions

## Outputs

- Phenotype-optimized network (e.g. pheno_opt1.sif and pheno_opt1.rds; reference model: 109 nodes, 298 edges)
- Inferred phenotype activity table (down = phenoscore < 0, up otherwise)
- Per-parameter-condition PhenoScore profiles from the 96-condition benchmark

## How to apply

Take the Step 2 network (nodes_df, edges_df, igraph_network) and a ProxPath protein-phenotype distance table preprocessed against the quantified proteome and phosphoproteome. Select significantly close phenotypes using path_length 2-4, stat='mean', and zscore_threshold=-1.96. Apply the randomization filter by sampling n_random=1000 random protein lists of equal size and discarding phenotypes whose activating/inhibitory path counts are matched by random lists at pvalue_threshold=0.05. Compute the PhenoScore as the weighted average of regulator activities under each condition's switches (remove_cascade=TRUE to keep only the most downstream regulator in cascades via the sp_graph, node_idx TRUE/FALSE, use_carnival_activity TRUE/FALSE), then re-optimize the phenotype-activity network with CARNIVAL (cplex solver, mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192) to obtain the final phenotype-optimized model.

## Related tools

- **ProxPath** (Graph-based algorithm that estimates the regulatory impact of proteins on SIGNOR-annotated phenotypes and selects significantly close phenotypes (zscore_threshold, stat)) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **SignalingProfiler 2.0** (Pipeline hosting Step 3: computes the PhenoScore from upstream regulator activities and maps up to 200 cellular phenotypes onto the final model) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **CARNIVAL** (ILP optimization used by optimize_pheno_network to re-optimize the final model on phenotypic activity with sign-coherent interactions) — https://saezlab.github.io/CARNIVAL/

## Examples

```
carnival_options = default_CARNIVAL_options(solver)
```

## Evaluation signals

- The final phenotype-optimized network must match the published model size of 109 nodes and 298 edges (output_matches_reference)
- PhenoScore signs must recapitulate the phenotypic gold standard: apoptosis/autophagy up and proliferation/biosynthesis down
- Randomization filter must be satisfied: phenotypes not significantly closer than n_random=1000 size-matched random protein lists (pvalue_threshold=0.05) are discarded
- Phenotype activity table follows the sign convention: down = phenoscore < 0, up otherwise
- Optimization must complete under default_CARNIVAL_options with cplex, mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192

## Limitations

- Prior knowledge coverage is incomplete: regulon and causal interaction resources cover <50% (about 9,000 proteins) of the UniProt-SwissProt proteome, limiting phenotype proximity scoring
- ProxPath depends on phenotypes annotated in SIGNOR; phenotypes outside SIGNOR cannot be scored
- Parameter choices (path_length 2-4, zscore_threshold, stat, remove_cascade, node_idx, use_carnival_activity) span 96 benchmarked conditions and must be tuned against a phenotypic gold standard; the ILP solver path must be manually specified by the user

## Evidence

- [intro] Step 3 connects optimized network to phenotypes via ProxPath: "Step 3 The context-specific network is connected to cellular phenotypes using the ProxPath algorithm24 and the phenotype activity is obtained by integrating upstream protein activities."
- [intro] ProxPath purpose: "we recently developed ProxPath24, a graph-based tool designed to estimate the regulatory impact of proteins on phenotypes annotated in SIGNOR17"
- [methods] Parameter selection from ProxPath table: "select from the ProxPath table the significantly close proteins next to your phenotypes of interest (key parameters are `zscore_threshold`, `stat`)"
- [methods] Randomization filter: "we sample a random protein list with the same number of proteins and compute the number of activating and inhibitory paths over the phenotype of interest"
- [methods] Cascade removal rule: "if two independent regulators are connected in the network (`sp_graph`), consider the most downstream (`remove_cascade = TRUE`)"
- [discussion] Phenotype coverage of final model: "map up to 200 cellular phenotypes onto the final model (Supplementary Data 13)"
