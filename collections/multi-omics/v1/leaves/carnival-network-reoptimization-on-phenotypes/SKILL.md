---
name: carnival-network-reoptimization-on-phenotypes
description: Use when a validated Step 2 context-specific network and a scored phenotype
  activity table are available and the final mechanistic model must be re-optimized
  so its edges explain phenotype-level (not protein-level) activity changes.
when_to_use_negative:
- The context-specific network from Step 2 has not yet been validated, or phenotype
  activities have not been inferred/scored via PhenoScore/PhenoScore benchmarking
  — CARNIVAL re-optimization on phenotypes presupposes these Step 2/Step 3 outputs.
- The analysis requires metabolomic data integration, which SignalingProfiler 2.0
  (and hence this phenotype re-optimization) does not support compared with COSMOS.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: SignalingProfiler 2.0
  role: R pipeline providing optimize_pheno_network, default_CARNIVAL_options, and
    the Step 2/Step 3 objects consumed by the re-optimization
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler
- name: CARNIVAL
  role: ILP optimization strategy that retains only sign-coherent interactions between
    proteins when re-optimizing the network on phenotype activities
  repo: https://saezlab.github.io/CARNIVAL/
- name: ProxPath
  role: Graph-based tool whose protein-phenotype distance table supplies the phenotype
    activities used as re-optimization targets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:1da15188b0a302e35b2ea655e000e5c114ecafe4309f8b0ee02c6be0be058faa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - CARNIVAL
  - ProxPath
  - SignalingProfiler 2.0
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# CARNIVAL network re-optimization on phenotypes

## Summary

Re-runs the CARNIVAL ILP optimization in SignalingProfiler 2.0 Step 3 using inferred phenotype activities (PhenoScores) rather than protein activities as objective targets, producing a final sign-coherent, phenotype-optimized context-specific network such as the 109-node/298-edge metformin model.

## When to use

Use when a validated Step 2 context-specific network and a scored phenotype activity table are available and the final mechanistic model must be re-optimized so its edges explain phenotype-level (not protein-level) activity changes.

## When NOT to use

- The context-specific network from Step 2 has not yet been validated, or phenotype activities have not been inferred/scored via PhenoScore/PhenoScore benchmarking — CARNIVAL re-optimization on phenotypes presupposes these Step 2/Step 3 outputs.
- The analysis requires metabolomic data integration, which SignalingProfiler 2.0 (and hence this phenotype re-optimization) does not support compared with COSMOS.

## Inputs

- Step 2 validated context-specific network object (nodes_df, edges_df, igraph_network, e.g. from toy_opt_network)
- Phenotype activity table from Step 3 PhenoScore/ProxPath inference (down = PhenoScore < 0, up otherwise)
- Selected optimal PhenoScore/ProxPath parameter combination (path_length 2-4, stat 'mean', zscore_threshold -1.96, n_random=1000, pvalue_threshold=0.05)
- CARNIVAL installation with a working ILP solver (cplex) and manually specified solver path

## Outputs

- Phenotype-optimized network in SIF format (e.g. pheno_opt1.sif; expected 109 nodes and 298 edges for the metformin model)
- Phenotype-optimized model as R object (e.g. pheno_opt1.rds)
- Inferred phenotype activity table integrated into the final model

## How to apply

After Step 3 has benchmarked the 96 PhenoScore/ProxPath parameter conditions against the phenotypic gold standard and selected the top-performing combination, load the Step 2 validated network (nodes_df, edges_df, igraph_network) and the phenotype activity table (signs defined as PhenoScore < 0 = down, otherwise up). Re-optimize with optimize_pheno_network, which formulates the CARNIVAL mixed-integer linear problem so that the retained edges are sign-coherent and consistent with the target phenotype activities, using default_CARNIVAL_options with the cplex solver, mipGAP=0.05, timelimit=3600 s, and cplexMemoryLimit=8192; the ILP solver path must be manually specified. The rationale is that the Step 2 network explains protein activity changes, whereas this re-optimization yields a final hierarchical model whose signaling axes directly recapitulate perturbed phenotypes (e.g., apoptosis/autophagy up, proliferation/biosynthesis down for metformin).

## Related tools

- **SignalingProfiler 2.0** (R pipeline providing optimize_pheno_network, default_CARNIVAL_options, and the Step 2/Step 3 objects consumed by the re-optimization) — https://github.com/SaccoPerfettoLab/SignalingProfiler
- **CARNIVAL** (ILP optimization strategy that retains only sign-coherent interactions between proteins when re-optimizing the network on phenotype activities) — https://saezlab.github.io/CARNIVAL/
- **ProxPath** (Graph-based tool whose protein-phenotype distance table supplies the phenotype activities used as re-optimization targets)

## Examples

```
carnival_options = default_CARNIVAL_options(solver = 'cplex', mipGAP = 0.05, timelimit = 3600, cplexMemoryLimit = 8192); pheno_opt <- optimize_pheno_network(opt_network, pheno_activities, carnival_options = carnival_options)
```

## Evaluation signals

- The reproduced phenotype-optimized network matches the reported final model size (109 nodes, 298 edges) via output_matches_reference against the published phenotype-optimized model.
- The PhenoScore profile of the final model shows the expected signs: apoptosis and autophagy up, proliferation and biosynthesis down (metformin gold standard).
- All retained edges are sign-coherent with the input phenotype activity signs, as enforced by the CARNIVAL optimization strategy.
- CARNIVAL run completes within the configured limits (timelimit=3600 s, cplexMemoryLimit=8192) at the mipGAP=0.05 tolerance.
- Output files pheno_opt1.sif and pheno_opt1.rds are written and parse as a valid SIF/R network object.

## Limitations

- The ILP solver path must be manually specified by the user before default_CARNIVAL_options can be used.
- Prior knowledge coverage is incomplete: regulon and causal interaction resources cover <50% (about 9,000 proteins) of the UniProt-SwissProt proteome, constraining which nodes can be retained.
- SignalingProfiler 2.0 does not include metabolomic data, so metabolite-driven phenotype changes cannot be represented in the re-optimized network.

## Evidence

- [intro] CARNIVAL optimization: "the CARNIVAL optimization strategy19 that retains only the sign-coherent interactions between proteins (context-specific network)"
- [methods] default_CARNIVAL_options call: "carnival_options = default_CARNIVAL_options(solver)"
- [discussion] phenotype integration in final model: "to estimate the activation level of hallmark phenotypes and integrate them in the final model"
- [readme] README CARNIVAL prerequisite: [evidence span withheld — anchor status: not_found]
