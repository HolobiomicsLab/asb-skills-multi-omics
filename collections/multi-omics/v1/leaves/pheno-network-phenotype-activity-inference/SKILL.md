---
name: pheno-network-phenotype-activity-inference
description: Use when a validated context-specific network (nodes_df, edges_df, igraph)
  must be linked to cellular phenotypes and phenotype activities inferred from upstream
  protein activities, as in benchmarking 96 PhenoScore/ProxPath parameter conditions
  against a phenotypic gold standard.
when_to_use_negative:
- The input is raw multi-omics data and no Step 2 network has been built yet — Step
  3 requires a CARNIVAL-optimized context-specific network object as its starting
  point.
- The goal involves metabolomic data integration — SignalingProfiler 2.0, unlike COSMOS,
  does not include metabolomic data.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3523
tools:
- name: SignalingProfiler 2.0
  role: 'Pipeline hosting Step 3: connects the context-specific network to phenotypes,
    computes PhenoScore, and re-optimizes the final phenotype-level model'
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: ProxPath
  role: Graph-based tool estimating the regulatory impact of proteins on SIGNOR-annotated
    phenotypes; supplies the distance table and path-count statistics
- name: CARNIVAL
  role: ILP optimization (cplex solver) used by optimize_pheno_network to produce
    the sign-coherent phenotype-optimized network
  repo: https://saezlab.github.io/CARNIVAL/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:a70ae26760a25dd7114d6c58c85a5252c9f7ec87a1b237b4d37bbdc308baf8d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - CARNIVAL
  - ProxPath
  - SignalingProfiler 2.0
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# pheno-network-phenotype-activity-inference

## Summary

SignalingProfiler 2.0 Step 3 skill: connects a CARNIVAL-optimized context-specific signaling network to SIGNOR-annotated cellular phenotypes using ProxPath, infers phenotype activity (PhenoScore) from upstream regulator activities, and re-optimizes a final phenotype-level network (e.g., the 109-node/298-edge metformin model).

## When to use

Use when a validated context-specific network (nodes_df, edges_df, igraph) must be linked to cellular phenotypes and phenotype activities inferred from upstream protein activities, as in benchmarking 96 PhenoScore/ProxPath parameter conditions against a phenotypic gold standard.

## When NOT to use

- The input is raw multi-omics data and no Step 2 network has been built yet — Step 3 requires a CARNIVAL-optimized context-specific network object as its starting point.
- The goal involves metabolomic data integration — SignalingProfiler 2.0, unlike COSMOS, does not include metabolomic data.

## Inputs

- Step 2 validated network object (nodes_df, edges_df, igraph_network, e.g. toy_opt_network)
- ProxPath protein–phenotype distance table preprocessed against the quantified proteome and phosphoproteome
- Phenotypic gold standard (e.g., 10 phenotypes with expected up/down signs, e.g. metformin: apoptosis/autophagy up, proliferation/biosynthesis down)
- PhenoScore/ProxPath parameter grid (96 conditions)

## Outputs

- Phenotype-optimized network (109 nodes, 298 edges) written to pheno_opt1.sif and pheno_opt1.rds
- Inferred phenotype activity table (down = phenoscore < 0, up otherwise)
- Per-condition PhenoScore profiles used to rank the 96 parameter settings against the gold standard

## How to apply

Load the Step 2 validated network (nodes_df, edges_df, igraph_network) and the ProxPath protein–phenotype distance table preprocessed against the quantified proteome and phosphoproteome. Select significantly close phenotypes per condition using path_length (2–4), stat='mean', and zscore_threshold=-1.96, then apply the randomization filter: sample n_random=1000 random protein lists of equal size and discard phenotypes whose activating/inhibitory path counts are matched by random lists at pvalue_threshold=0.05. Compute the PhenoScore as the weighted average of upstream regulator activities under each condition's switches (remove_cascade=TRUE, node_idx=TRUE/FALSE, use_carnival_activity=TRUE/FALSE), using the sp_graph to keep only the most downstream regulator when two regulators are connected in a cascade. Score each of the 96 parameter conditions against the phenotypic gold standard (e.g., 10 phenotypes) and select the top-performing combination. Finally, re-optimize the network on phenotypic activity with optimize_pheno_network via CARNIVAL (default_CARNIVAL_options with cplex solver, mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192) and write the phenotype-optimized model to pheno_opt1.sif and pheno_opt1.rds.

## Related tools

- **SignalingProfiler 2.0** (Pipeline hosting Step 3: connects the context-specific network to phenotypes, computes PhenoScore, and re-optimizes the final phenotype-level model) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **ProxPath** (Graph-based tool estimating the regulatory impact of proteins on SIGNOR-annotated phenotypes; supplies the distance table and path-count statistics)
- **CARNIVAL** (ILP optimization (cplex solver) used by optimize_pheno_network to produce the sign-coherent phenotype-optimized network) — https://saezlab.github.io/CARNIVAL/

## Examples

```
carnival_options = default_CARNIVAL_options(solver)  # cplex, mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192; then optimize_pheno_network() -> pheno_opt1.sif / pheno_opt1.rds
```

## Evaluation signals

- The reproduced phenotype-optimized network matches the reported final model size (109 nodes, 298 edges).
- The PhenoScore profile recapitulates the expected gold-standard signs: apoptosis/autophagy up and proliferation/biosynthesis down.
- output_matches_reference against the published phenotype-optimized model (pheno_opt1.sif / pheno_opt1.rds).
- Phenotype activity table signs follow the rule: down = phenoscore < 0, up otherwise.
- All phenotypes surviving selection have randomization-filter p-values below pvalue_threshold=0.05 with n_random=1000.

## Limitations

- Prior knowledge coverage is incomplete: regulon and causal interaction resources cover <50% (~9,000 proteins) of the UniProt-SwissProt proteome, limiting which phenotypes can be connected.
- The ILP solver path (cplex) must be manually specified by the user when configuring default_CARNIVAL_options.
- The exact parameter combination (path_length, node_idx, use_carnival_activity) must be selected empirically from the 96-condition benchmark against a gold standard; performance depends on gold-standard quality.

## Evidence

- [intro] Step 3 connects the context-specific network to cellular phenotypes via ProxPath and infers activity: "Step 3 The context-specific network is connected to cellular phenotypes using the ProxPath algorithm24 and the phenotype activity is obtained by integrating upstream protein activities."
- [methods] ProxPath selection parameters: "select from the ProxPath table the significantly close proteins next to your phenotypes of interest (key parameters are `zscore_threshold`, `stat`)"
- [methods] Randomization filter rationale: "we sample a random protein list with the same number of proteins and compute the number of activating and inhibitory paths over the phenotype of interest"
- [methods] remove_cascade decision rule: "if two independent regulators are connected in the network (`sp_graph`), consider the most downstream (`remove_cascade = TRUE`)"
- [readme] README installation command: "devtools::install_github('https://github.com/SaccoPerfettoLab/SignalingProfiler/')"
