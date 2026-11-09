---
name: network-parameter-benchmarking
description: Use when a mechanistic network reconstruction step has many tunable technical
  parameters (PKN source, max path lengths, ILP solver settings) and you must screen
  combinations, rank resulting networks by a combined score, and benchmark them against
  a gold-standard protein set.
when_to_use_negative:
- Step 1 (protein activity inference via PhosphoScore/VIPER) or Step 3 (ProxPath phenotype
  connection) tuning is the goal — this skill only benchmarks Step 2 network construction
- Data types outside transcriptomics/proteomics/phosphoproteomics are involved, e.g.
  metabolomics (SignalingProfiler 2.0 does not include metabolomic data)
- No gold-standard protein set exists to score network quality, so parameter combinations
  cannot be ranked
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3957
tools:
- name: SignalingProfiler 2.0
  role: R pipeline providing choose_PKN, preprocess_PKN, two_layer_naive_network,
    prepare_carnival_input, run_carnival_and_create_graph, union_of_graphs, and expand_and_map_edges
    used to build and validate each candidate network
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: CARNIVAL
  role: ILP optimization that retains sign-coherent interactions to derive the context-specific
    network; run as two shots (receptors to kinases/phosphatases, then to transcription
    factors) with solvers compared (inverseCARNIVAL vs standard CARNIVAL)
  repo: https://saezlab.github.io/CARNIVAL/
- name: cplex
  role: ILP solver configured with mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192
    for the benchmarking runs
- name: Gurobi
  role: Alternative ILP solver for the CARNIVAL optimization in the parameter screen
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:e3e7b512dbd0fb5553f45d0e0846012b9e4ecfc04f35fdb4a374609803856b91
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - CARNIVAL
  - Gurobi
  - SignalingProfiler 2.0
  - cplex
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3957
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# network-parameter-benchmarking

## Summary

Systematic screening of Step 2 network-construction parameter combinations in SignalingProfiler 2.0 (PKN choices, max path lengths, CARNIVAL solver settings) and ranking of the resulting context-specific networks against a perturbation-response gold standard (e.g. metformin proteins) using precision/recall.

## When to use

Use when a mechanistic network reconstruction step has many tunable technical parameters (PKN source, max path lengths, ILP solver settings) and you must screen combinations, rank resulting networks by a combined score, and benchmark them against a gold-standard protein set.

## When NOT to use

- Step 1 (protein activity inference via PhosphoScore/VIPER) or Step 3 (ProxPath phenotype connection) tuning is the goal — this skill only benchmarks Step 2 network construction
- Data types outside transcriptomics/proteomics/phosphoproteomics are involved, e.g. metabolomics (SignalingProfiler 2.0 does not include metabolomic data)
- No gold-standard protein set exists to score network quality, so parameter combinations cannot be ranked

## Inputs

- Transcriptomics, proteomics, and phosphoproteomics quantification tables (case vs control)
- Built-in human prior knowledge network (choose_PKN, direct=TRUE)
- User-defined receptor/perturbed-molecule list with signed activity (e.g. MTOR = -1, AMPK = 1)
- Gold-standard protein set for the perturbation of interest (e.g. metformin proteins)
- Candidate grid of Step 2 technical parameter combinations

## Outputs

- two_layers_naive.rds / two_layers_naive.sif (two-layer naive network)
- optimized_network.sif and optimized_object.rds (merged CARNIVAL solution via union_of_graphs)
- optimized_network_validated.sif / optimized_object_validated.rds (edges annotated with matched phosphosites, fold-changes, is_quantified, is_significant)
- Ranked network list with combined score and aggregate precision/recall metrics (e.g. 2989 of 3328 networks retained, precision 0.75 / recall 0.35)

## How to apply

Load the built-in human prior knowledge network with choose_PKN(organism='human', with_atlas=FALSE, direct=TRUE), restrict it to analytes quantified in the transcriptomics, proteomics, and phosphoproteomics tables via preprocess_PKN, then build the two-layer naive network with two_layer_naive_network using max_length_1=3 (receptors to kinases/phosphatases/other proteins) and max_length_2=4 (receptors to transcription factors). Prepare CARNIVAL inputs with the receptor list (e.g. list('MTOR' = -1, 'AMPK' = 1)) and run the two-shots optimization with default_CARNIVAL_options for the cplex solver configured with mipGAP=0.05, timelimit=3600, and cplexMemoryLimit=8192, merging the two solutions with union_of_graphs. Map phosphoproteomics onto signaling edges with expand_and_map_edges to annotate each edge with matched phosphosites and is_quantified/is_significant flags. Repeat across the candidate parameter grid (e.g. 3328 combinations), rank the resulting networks by the combined score, and validate aggregate performance against the gold standard (retention rate, average precision/recall, and behavior of inverseCARNIVAL vs standard CARNIVAL).

## Related tools

- **SignalingProfiler 2.0** (R pipeline providing choose_PKN, preprocess_PKN, two_layer_naive_network, prepare_carnival_input, run_carnival_and_create_graph, union_of_graphs, and expand_and_map_edges used to build and validate each candidate network) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **CARNIVAL** (ILP optimization that retains sign-coherent interactions to derive the context-specific network; run as two shots (receptors to kinases/phosphatases, then to transcription factors) with solvers compared (inverseCARNIVAL vs standard CARNIVAL)) — https://saezlab.github.io/CARNIVAL/
- **cplex** (ILP solver configured with mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192 for the benchmarking runs)
- **Gurobi** (Alternative ILP solver for the CARNIVAL optimization in the parameter screen)

## Examples

```
carnival_options = default_CARNIVAL_options(solver); naive <- two_layer_naive_network(pkn, max_length_1 = 3, max_length_2 = 4)
```

## Evaluation signals

- Retention rate across the parameter grid matches the reported benchmark: 2989 networks retained of 3328 candidates
- Aggregate performance against the gold standard reaches the reported values (average precision 0.75, recall 0.35)
- The representative selected network reproduces the reported 99-node/219-edge size and recapitulates mTOR inhibition and AMPK activation
- Network edges are supported by quantified phosphosites (is_quantified/is_significant flags populated after expand_and_map_edges)
- Parameter settings selected on the training data show comparable performance on validation datasets, indicating limited overfitting

## Limitations

- Prior knowledge coverage is incomplete: regulon and causal interaction resources cover <50% (~9,000 proteins) of the UniProt-SwissProt proteome, capping recall
- The ILP solver path must be manually specified by the user, and PsP Regulatory_sites and Kinase_Substrates files must be manually downloaded if PsP information is included
- inverseCARNIVAL vs standard CARNIVAL behavior differs across parameter settings and must be checked during validation

## Evidence

- [intro] the CARNIVAL optimization strategy19 that retains only the sign-coherent interactions between proteins (context-specific network): "the CARNIVAL optimization strategy19 that retains only the sign-coherent interactions between proteins (context-specific network)"
- [intro] A user-defined set of perturbed molecules/receptors (e.g., targets of a treatment or mutated genes in a disease) is connected to the inferred proteins using a prior knowledge network (PKN) exploiting: "A user-defined set of perturbed molecules/receptors (e.g., targets of a treatment or mutated genes in a disease) is connected to the inferred proteins using a prior knowledge network (PKN)"
- [methods] carnival_options = default_CARNIVAL_options(solver): "carnival_options = default_CARNIVAL_options(solver)"
- [discussion] a benchmarking approach consisting of a parameter tuning phase on a training dataset30 and a subsequent validation phase on murine20 and human39 datasets: "a benchmarking approach consisting of a parameter tuning phase on a training dataset30 and a subsequent validation phase on murine20 and human39 datasets"
- [readme] SignalingProfiler is dependent on Saez lab algorithm CARNIVAL for performing the signalling pathway optimization.: [evidence span withheld — anchor status: not_found]
