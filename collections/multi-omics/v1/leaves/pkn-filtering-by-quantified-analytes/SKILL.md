---
name: pkn-filtering-by-quantified-analytes
description: 'Use when starting SignalingProfiler 2.0 Step 2 with multi-omics tables
  and a full prior knowledge network: the PKN must first be subset to analytes measured
  in transcriptomics, proteomics, and phosphoproteomics before receptor-to-TF path
  construction.'
when_to_use_negative:
- The goal includes metabolomic data integration — SignalingProfiler 2.0 does not
  include metabolomic data, so metabolite nodes cannot be retained by this filtering
- 'The analyte of interest is known to be relevant but unquantified: regulon and causal
  interaction resources cover <50% (~9,000 proteins) of the UniProt-SwissProt proteome,
  so filtering may drop biologically important, evidence-poor nodes'
- Input is a single omics modality only — the two-layer receptor-to-TF construction
  expects matched transcriptomics, proteomics, and phosphoproteomics support
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0203
tools:
- name: SignalingProfiler 2.0
  role: Provides choose_PKN and preprocess_PKN to load and restrict the prior knowledge
    network to quantified analytes before two-layer network construction
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: CARNIVAL
  role: Downstream ILP optimizer that consumes the filtered PKN to retain only sign-coherent
    interactions in the context-specific network
  repo: https://saezlab.github.io/CARNIVAL/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:38a43f4240cb3bec9010b862205bbe60be18d2b3484b48607d671a78a894cd2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - CARNIVAL
  - Gurobi
  - SignalingProfiler 2.0
  - cplex
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# PKN filtering by quantified analytes

## Summary

Restricts the human prior knowledge causal network (e.g., SIGNOR/PsP-derived) to only those nodes (proteins, kinases, phosphatases, transcription factors) that are actually quantified in the input transcriptomics, proteomics, and phosphoproteomics tables before Step 2 network construction, so CARNIVAL optimization operates on an evidence-supported subnetwork.

## When to use

Use when starting SignalingProfiler 2.0 Step 2 with multi-omics tables and a full prior knowledge network: the PKN must first be subset to analytes measured in transcriptomics, proteomics, and phosphoproteomics before receptor-to-TF path construction.

## When NOT to use

- The goal includes metabolomic data integration — SignalingProfiler 2.0 does not include metabolomic data, so metabolite nodes cannot be retained by this filtering
- The analyte of interest is known to be relevant but unquantified: regulon and causal interaction resources cover <50% (~9,000 proteins) of the UniProt-SwissProt proteome, so filtering may drop biologically important, evidence-poor nodes
- Input is a single omics modality only — the two-layer receptor-to-TF construction expects matched transcriptomics, proteomics, and phosphoproteomics support

## Inputs

- Built-in human prior knowledge causal network from SIGNOR and PhosphoSitePlus (optionally Ser/Thr Kinome Atlas), loaded via choose_PKN
- Case-vs-control transcriptomics quantification table
- Case-vs-control proteomics quantification table
- Case-vs-control phosphoproteomics quantification table with phosphosite identifiers

## Outputs

- Filtered PKN restricted to analytes quantified in the transcriptomics, proteomics, and phosphoproteomics tables
- Two-layer naïve network (two_layers_naive.rds and two_layers_naive.sif) built on the filtered PKN

## How to apply

Load the built-in human prior knowledge network with choose_PKN(organism='human', with_atlas=FALSE, direct=TRUE) (optionally including kinase-substrate relations from the Ser/Thr Kinome Atlas via with_atlas). Then apply preprocess_PKN to restrict the PKN to analytes quantified in the transcriptomics, proteomics, and phosphoproteomics tables, intersecting node identifiers across omics layers. The rationale is that CARNIVAL's ILP optimization should only reason over nodes with measured activity or abundance, preventing unquantified intermediates from dominating the inferred context-specific network. The filtered PKN then feeds two_layer_naive_network (receptors to kinases/phosphatases/other proteins with max_length_1=3, and to transcription factors with max_length_2=4) and downstream prepare_carnival_input / run_carnival_and_create_graph steps.

## Related tools

- **SignalingProfiler 2.0** (Provides choose_PKN and preprocess_PKN to load and restrict the prior knowledge network to quantified analytes before two-layer network construction) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **CARNIVAL** (Downstream ILP optimizer that consumes the filtered PKN to retain only sign-coherent interactions in the context-specific network) — https://saezlab.github.io/CARNIVAL/

## Examples

```
pkn <- choose_PKN(organism = 'human', with_atlas = FALSE, direct = TRUE); pkn <- preprocess_PKN(pkn, transcriptomics, proteomics, phosphoproteomics)
```

## Evaluation signals

- Every node in the filtered PKN is present as a quantified analyte in at least one of the transcriptomics, proteomics, or phosphoproteomics tables
- Across the full Step 2 parameter screen, the retained networks reproduce the reported aggregate performance: 2989 of 3328 candidate networks retained, average precision 0.75 and recall 0.75/0.35 against the metformin protein gold standard
- The representative optimized network recapitulates the expected perturbation signatures (mTOR inhibition, AMPK activation) with a 99-node/219-edge structure and edges supported by quantified phosphosites (is_quantified/is_significant flags from expand_and_map_edges)
- max_length_1=3 (receptors to kinases/phosphatases/other) and max_length_2=4 (receptors to TFs) are respected in the two-layer naïve network built on the filtered PKN

## Limitations

- Regulon databases and causal interaction resources are incomplete and offer information for <50% (about 9,000 proteins) of the UniProt-SwissProt proteome, so filtering by quantified analytes compounds prior-knowledge sparsity
- If PsP information is included, the Regulatory_sites and Kinase_Substrates files must be manually downloaded from the PsP database before PKN loading
- The ILP solver path (cplex, mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192) must be manually specified by the user

## Evidence

- [methods] Load the built-in human prior knowledge network with choose_PKN(organism='human', with_atlas=FALSE, direct=TRUE) and restrict it to analytes quantified in the transcriptomics, proteomics, and phosphoproteomics tables using preprocess_PKN.: [evidence span withheld — anchor status: not_found]
- [methods] create_PKN(database = c('SIGNOR', 'PsP'), direct = TRUE, organism = 'human': [evidence span withheld — anchor status: not_found]
- [intro] Step 2: connect user-defined perturbed molecules/receptors to inferred proteins via a prior knowledge network using a shortest-path algorithm and the CARNIVAL optimization strategy: [evidence span withheld — anchor status: not_found]
- [readme] SignalingProfiler is a R based pipeline that integrates transcriptomics, proteomics and phosphoproteomics data among a case and a control to build a signaling network: [evidence span withheld — anchor status: not_found]
- [intro] the CARNIVAL optimization strategy19 that retains only the sign-coherent interactions between proteins (context-specific network): "retains only the sign-coherent interactions between proteins (context-specific network)"
