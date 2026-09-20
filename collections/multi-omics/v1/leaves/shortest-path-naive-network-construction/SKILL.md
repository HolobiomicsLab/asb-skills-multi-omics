---
name: shortest-path-naive-network-construction
description: Use when multi-omics perturbation data (transcriptomics, proteomics,
  phosphoproteomics) must be connected to known receptors/perturbed molecules to yield
  a context-specific mechanistic signaling network.
when_to_use_negative:
- The goal is phenotypic interpretation or metabolite integration — SignalingProfiler
  2.0 does not include metabolomic data (use COSMOS instead)
- The network nodes and edge directions are already resolved and only pathway-level
  phenotype scoring is needed (HiPathia-style circuit decomposition is the wrong fit
  for building the network itself)
- No prior-knowledge causal resources cover the analytes of interest — regulons and
  causal interaction databases cover <50% (~9,000 proteins) of the UniProt-SwissProt
  proteome
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3577
tools:
- name: SignalingProfiler 2.0
  role: R pipeline providing choose_PKN, preprocess_PKN, two_layer_naive_network,
    prepare_carnival_input, run_carnival_and_create_graph, union_of_graphs, and expand_and_map_edges
    for network construction
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: CARNIVAL
  role: ILP optimization strategy that retains only sign-coherent interactions between
    proteins to yield the context-specific network
  repo: https://saezlab.github.io/CARNIVAL/
- name: cplex
  role: ILP solver used with mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192; solver
    path must be manually specified
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:e811a56a3c7c863f4229b8e21adfa12aa143ed2d90d08c12c6582f62dc8ff985
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - CARNIVAL
  - Gurobi
  - SignalingProfiler 2.0
  - cplex
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Shortest-path naïve network construction

## Summary

Builds a two-layer naïve prior-knowledge network connecting user-defined perturbed receptors (e.g., MTOR, AMPK) to kinases/phosphatases and transcription factors via shortest-path search on a restricted SIGNOR/PsP PKN, then prunes it with CARNIVAL ILP optimization and validates edges against matched phosphosite fold-changes.

## When to use

Use when multi-omics perturbation data (transcriptomics, proteomics, phosphoproteomics) must be connected to known receptors/perturbed molecules to yield a context-specific mechanistic signaling network.

## When NOT to use

- The goal is phenotypic interpretation or metabolite integration — SignalingProfiler 2.0 does not include metabolomic data (use COSMOS instead)
- The network nodes and edge directions are already resolved and only pathway-level phenotype scoring is needed (HiPathia-style circuit decomposition is the wrong fit for building the network itself)
- No prior-knowledge causal resources cover the analytes of interest — regulons and causal interaction databases cover <50% (~9,000 proteins) of the UniProt-SwissProt proteome

## Inputs

- Human prior knowledge network (SIGNOR, PhosphoSitePlus; optional Ser/Thr Kinome Atlas) restricted to quantified analytes
- Transcriptomics, proteomics, and phosphoproteomics abundance tables (case vs control)
- Receptor/perturbed-molecule list with signed directions (e.g., list('MTOR' = -1, 'AMPK' = 1))
- CARNIVAL solver path (cplex or gurobi) and solver options

## Outputs

- two_layers_naive.rds and two_layers_naive.sif (two-layer naïve network)
- optimized_network.sif and optimized_object.rds (merged CARNIVAL solution graph)
- optimized_network_validated.sif and optimized_object_validated.rds (edges annotated with matched phosphosites, fold-changes, is_quantified/is_significant flags)
- Ranked network list with aggregate gold-standard precision/recall (benchmarking: 0.75 precision / 0.35 recall)

## How to apply

Load the human prior knowledge network with choose_PKN(organism='human', with_atlas=FALSE, direct=TRUE) and restrict it to analytes quantified in the transcriptomics, proteomics, and phosphoproteomics tables using preprocess_PKN. Build the two-layer naïve network with two_layer_naive_network using max_length_1=3 (receptors to kinases/phosphatases/other proteins) and max_length_2=4 (receptors to transcription factors), saving two_layers_naive.rds and two_layers_naive.sif. Prepare CARNIVAL inputs with prepare_carnival_input (receptor_list = list('MTOR' = -1, 'AMPK' = 1)) and run the two-shots optimization with run_carnival_and_create_graph — first from receptors to kinases/phosphatases/other, then from those proteins to transcription factors — merging solutions with union_of_graphs into optimized_network.sif. Configure the solver via default_CARNIVAL_options (cplex: mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192); the ILP solver path must be manually specified. Map phosphoproteomics onto signaling edges with expand_and_map_edges to annotate matched phosphosites, fold-changes, and is_quantified/is_significant flags. When benchmarking, screen candidate parameter combinations across Step 2 and rank networks by a combined score against a gold standard.

## Related tools

- **SignalingProfiler 2.0** (R pipeline providing choose_PKN, preprocess_PKN, two_layer_naive_network, prepare_carnival_input, run_carnival_and_create_graph, union_of_graphs, and expand_and_map_edges for network construction) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **CARNIVAL** (ILP optimization strategy that retains only sign-coherent interactions between proteins to yield the context-specific network) — https://saezlab.github.io/CARNIVAL/
- **cplex** (ILP solver used with mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192; solver path must be manually specified)

## Examples

```
pkn <- choose_PKN(organism='human', with_atlas=FALSE, direct=TRUE); naive <- two_layer_naive_network(pkn, max_length_1=3, max_length_2=4)
```

## Evaluation signals

- The naïve network files two_layers_naive.rds and two_layers_naive.sif exist and respect max_length_1=3 / max_length_2=4 shortest-path constraints
- The merged CARNIVAL solution reproduces expected perturbation signatures (e.g., a representative 99-node/219-edge network recapitulating mTOR inhibition and AMPK activation)
- Validation edges in optimized_network_validated.sif carry matched phosphosite annotations with is_quantified and is_significant flags
- Benchmarking over the 3328 candidate Step 2 parameter combinations retains ~2989 networks with aggregate precision ≈ 0.75 and recall ≈ 0.35 against the metformin protein gold standard
- All retained edges are sign-coherent as enforced by the CARNIVAL optimization

## Limitations

- Regulon and causal interaction databases are incomplete, offering information for <50% (about 9,000 proteins) of the UniProt-SwissProt proteome
- The ILP solver path must be manually specified by the user; PsP Regulatory_sites and Kinase_Substrates files must be manually downloaded from the PsP database
- If two independent regulators are connected in the network, the most downstream must be considered (remove_cascade = TRUE) to avoid upstream-cascade artifacts

## Evidence

- [intro] Step 2 connects perturbed molecules to inferred proteins via PKN shortest-path and CARNIVAL: "A user-defined set of perturbed molecules/receptors (e.g., targets of a treatment or mutated genes in a disease) is connected to the inferred proteins using a prior knowledge network (PKN) exploiting"
- [readme] CARNIVAL dependency for signaling pathway optimization: [evidence span withheld — anchor status: not_found]
- [methods] Solver options for the Step 2 optimization: "carnival_options = default_CARNIVAL_options(solver)"
- [results] Aggregate benchmark performance of Step 2 networks against the metformin gold standard: [evidence span withheld — anchor status: not_found]
- [methods] PKN construction options from article methods: [evidence span withheld — anchor status: not_found]
