---
name: ilp-causal-network-optimization
description: Use when multi-omics case-vs-control data (transcriptomics, proteomics,
  phosphoproteomics) yield inferred protein activities plus a user-defined set of
  perturbed molecules/receptors (e.g. EGF, MTOR/AMPK, Flt3) that must be connected
  through a prior knowledge causal network.
when_to_use_negative:
- The dataset includes metabolomics that must be integrated — SignalingProfiler 2.0/CARNIVAL
  does not include metabolomic data (COSMOS would be the alternative)
- No ILP solver (cplex/Gurobi) is installed or its path cannot be manually specified
  — the optimization step cannot run
- You need to infer protein activities first rather than connect already-inferred
  proteins to a receptor — Step 1 footprint-based analysis (VIPER/PhosphoScore) must
  precede this skill
edam_operation: http://edamontology.org/operation_0271
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_2259
tools:
- name: SignalingProfiler 2.0
  role: 'Wraps the whole Step 2 workflow: choose_PKN, preprocess_PKN, two_layer_naive_network,
    prepare_carnival_input, run_carnival_and_create_graph, union_of_graphs, expand_and_map_edges'
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: CARNIVAL
  role: Performs the sign-coherent ILP optimization (two-shots, standard and inverseCARNIVAL
    variants) over the naive two-layer network
  repo: https://saezlab.github.io/CARNIVAL/
- name: cplex
  role: Default ILP solver configured via default_CARNIVAL_options (mipGAP=0.05, timelimit=3600,
    cplexMemoryLimit=8192); Gurobi is an alternative
provenance:
  source_task_ids:
  - task_005
  - task_003
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:0552b28b60cf6b99bf0dbd4d8f43694333faba7056727d59a4f82bf1031e9bb3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - CARNIVAL
  - Gurobi
  - PhosphoSitePlus
  - SIGNOR
  - SignalingProfiler 2.0
  - VIPER
  - cplex
  edam_operation: http://edamontology.org/operation_0271
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# ilp-causal-network-optimization

## Summary

Sign-coherent integer linear programming (CARNIVAL) optimization used in SignalingProfiler 2.0's Step 2 to retain only the interactions of a prior knowledge network that coherently explain measured protein activities, yielding a context-specific mechanistic signaling network.

## When to use

Use when multi-omics case-vs-control data (transcriptomics, proteomics, phosphoproteomics) yield inferred protein activities plus a user-defined set of perturbed molecules/receptors (e.g. EGF, MTOR/AMPK, Flt3) that must be connected through a prior knowledge causal network.

## When NOT to use

- The dataset includes metabolomics that must be integrated — SignalingProfiler 2.0/CARNIVAL does not include metabolomic data (COSMOS would be the alternative)
- No ILP solver (cplex/Gurobi) is installed or its path cannot be manually specified — the optimization step cannot run
- You need to infer protein activities first rather than connect already-inferred proteins to a receptor — Step 1 footprint-based analysis (VIPER/PhosphoScore) must precede this skill

## Inputs

- Case-vs-control transcriptomics, proteomics and deduplicated phosphoproteomics tables (UniProt ID, gene_name, aminoacid, position, sequence window, difference, logpval, significant)
- Inferred protein activities from Step 1 (TFEA/KSEA footprints via VIPER plus PhosphoScore)
- Prior knowledge network (SIGNOR/PsP-derived, e.g. PKN_human_dir or PKN_mouse) restricted via preprocess_PKN
- User-defined signed perturbed receptor/molecule list (e.g. list('MTOR'=-1, 'AMPK'=1))
- ILP solver installation (cplex or Gurobi) with manually specified solver path

## Outputs

- two_layers_naive.rds and two_layers_naive.sif (pre-optimization naive network)
- optimized_network.sif and optimized_object.rds (merged two-shots CARNIVAL solution)
- optimized_network_validated.sif and optimized_object_validated.rds (edges annotated with is_quantified / is_significant phosphosites)
- Context-specific mechanistic network, e.g. 84-node/256-edge EGF HeLa network or 99-node/219-edge metformin network

## How to apply

First restrict the chosen PKN (e.g. PKN_human_dir via choose_PKN(organism='human', with_atlas=FALSE, direct=TRUE)) to analytes quantified in the omics tables with preprocess_PKN, then build a two-layer naive network with two_layer_naive_network (max_length_1=3 from receptors to kinases/phosphatases/other, max_length_2=4 to transcription factors). Prepare the CARNIVAL input with prepare_carnival_input using a signed receptor list (e.g. list('MTOR'=-1, 'AMPK'=1) for metformin) and configure default_CARNIVAL_options for the cplex solver with mipGAP=0.05, timelimit=3600 and cplexMemoryLimit=8192; the ILP solver path must be manually specified by the user. Run the two-shots CARNIVAL optimization with run_carnival_and_create_graph — first from receptor to kinases/phosphatases/other proteins, then from those to transcription factors (passing output nodes of run 1 via convert_output_nodes_in_next_input) — and merge the two solutions with union_of_graphs. The ILP retains only sign-coherent interactions, so the result is a small context-specific network rather than the full shortest-path skeleton. Finally annotate edges with phosphoproteomic evidence using expand_and_map_edges, flagging is_quantified and is_significant phosphosite-supported edges.

## Related tools

- **SignalingProfiler 2.0** (Wraps the whole Step 2 workflow: choose_PKN, preprocess_PKN, two_layer_naive_network, prepare_carnival_input, run_carnival_and_create_graph, union_of_graphs, expand_and_map_edges) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **CARNIVAL** (Performs the sign-coherent ILP optimization (two-shots, standard and inverseCARNIVAL variants) over the naive two-layer network) — https://saezlab.github.io/CARNIVAL/
- **cplex** (Default ILP solver configured via default_CARNIVAL_options (mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192); Gurobi is an alternative)

## Examples

```
carnival_options = default_CARNIVAL_options(solver='cplex')  # mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192
```

## Evaluation signals

- Network retains only sign-coherent interactions consistent with the signed receptor input and Step 1 protein activities
- Against a protein gold standard (e.g. metformin), retained networks reach satisfactory aggregate performance — average precision 0.75 and recall 0.35 across the 2989 networks retained of 3328 candidates
- Representative networks recapitulate expected perturbation signatures (e.g. 99-node/219-edge metformin network reproducing mTOR inhibition and AMPK activation; EGF HeLa network of 84 nodes/256 edges recovering BRAF, RAF1, MAPK1-3, MAPK14, ATF7 and JUN)
- A defined fraction of edges carry phosphoproteomic evidence via expand_and_map_edges (is_quantified / is_significant flags; ~14% phosphorylation-mapped edges in the EGF HeLa case)
- Both two-shots solutions exist and were merged with union_of_graphs into a single optimized_object.rds

## Limitations

- The ILP solver path must be manually specified by the user before optimization can run
- Prior knowledge coverage is incomplete: causal interaction resources cover <50% (~9,000 proteins) of the UniProt-SwissProt proteome, bounding recoverable networks
- InverseCARNIVAL behaves differently from standard CARNIVAL (screened among the 3328 parameter combinations)
- Under heavy perturbation of the prior knowledge (PKN shuffling in robustness tests), phosphorylation-edge coverage of the optimized network is reduced

## Evidence

- [intro] CARNIVAL optimization rationale: "the CARNIVAL optimization strategy19 that retains only the sign-coherent interactions between proteins (context-specific network)"
- [intro] Step 2 PKN connection: "A user-defined set of perturbed molecules/receptors (e.g., targets of a treatment or mutated genes in a disease) is connected to the inferred proteins using a prior knowledge network (PKN) exploiting"
- [methods] solver options configuration: "carnival_options = default_CARNIVAL_options(solver)"
- [methods] solver path requirement: "Then, you have to manually specify the ILP solver path."
- [readme] CARNIVAL dependency: [evidence span withheld — anchor status: not_found]
