---
name: phosphosite-mapping-on-network-edges
description: Use when a two-shots CARNIVAL-optimized signaling network (union_of_graphs
  output, SIF + RDS) must be supported by measured phosphoproteomics — i.e., each
  retained signaling edge needs its matched phosphosite, fold-change and significance
  annotation before downstream phenoscore or benchmarking.
when_to_use_negative:
- The dataset is metabolomics-only or lacks a phosphoproteomics table — SignalingProfiler
  2.0 does not include metabolomic data and there are no phosphosites to map onto
  edges
- The network is still the naïve two-layer graph (two_layers_naive.sif) and has not
  yet been optimized by two-shots CARNIVAL — edges are not yet sign-coherent or context-specific
- Only protein-level or transcript-level abundances exist with no site-level phosphorylation
  quantification, so no edge can be supported by a matched phosphosite
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_2258
tools:
- name: SignalingProfiler 2.0
  role: Provides expand_and_map_edges, the routine that annotates each optimized network
    edge with matched phosphosites, fold-changes and is_quantified/is_significant
    flags
  repo: github.com/SaccoPerfettoLab/SignalingProfiler
- name: CARNIVAL
  role: Upstream ILP optimization producing the sign-coherent context-specific network
    (optimized_network.sif / optimized_object.rds) that the phosphosite mapping validates
  repo: saezlab.github.io/CARNIVAL
- name: PhosphoSitePlus
  role: Source of kinase-substrate/regulatory-site relations underlying the PKN edges
    onto which phosphoproteomics is mapped
provenance:
  source_task_ids:
  - task_003
  - task_007
  - task_005
  - task_001
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:9530dab108b190d4512b8eca29df71ee9c67f452dcb99ee86b660a527f2c82cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CARNIVAL
  - Gurobi
  - PhosphoSitePlus
  - SIGNOR
  - Ser/Thr Kinome Atlas
  - SignalingProfiler 2.0
  - VIPER
  - cplex
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# phosphosite-mapping-on-network-edges

## Summary

Annotate SignalingProfiler 2.0's optimized context-specific network edges with matched phosphosites and phosphoproteomic evidence (fold-changes, is_quantified and is_significant flags) so the mechanistic model is validated by quantified phosphorylation events rather than prior knowledge alone.

## When to use

Use when a two-shots CARNIVAL-optimized signaling network (union_of_graphs output, SIF + RDS) must be supported by measured phosphoproteomics — i.e., each retained signaling edge needs its matched phosphosite, fold-change and significance annotation before downstream phenoscore or benchmarking.

## When NOT to use

- The dataset is metabolomics-only or lacks a phosphoproteomics table — SignalingProfiler 2.0 does not include metabolomic data and there are no phosphosites to map onto edges
- The network is still the naïve two-layer graph (two_layers_naive.sif) and has not yet been optimized by two-shots CARNIVAL — edges are not yet sign-coherent or context-specific
- Only protein-level or transcript-level abundances exist with no site-level phosphorylation quantification, so no edge can be supported by a matched phosphosite

## Inputs

- Optimized network SIF file (optimized_network.sif) and CARNIVAL result object (optimized_object.rds) from union_of_graphs
- Deduplicated phosphoproteomics table (one occurrence per phosphosite) with fold-changes and significance per site, from transcriptomics/proteomics/phosphoproteomics preprocessing
- Prior knowledge network restricted to quantified analytes via preprocess_PKN (human or mouse, e.g. PKN_human_dir)

## Outputs

- optimized_network_validated.sif — edge-annotated SIF with matched phosphosites, fold-changes, is_quantified and is_significant flags per edge
- optimized_object_validated.rds — serialized RDS of the validated network object
- Phosphorylation-edge coverage statistics (fraction of is_quantified / is_significant edges) used for robustness benchmarking

## How to apply

After the two-layer naïve network is optimized with two-shots CARNIVAL (cplex solver, mipGAP=0.05, timelimit=3600) and the two graphs merged via union_of_graphs into optimized_network.sif / optimized_object.rds, run expand_and_map_edges to intersect the receptor→kinase/phosphatase/TF edges with the deduplicated phosphoproteomics table (one occurrence per site). Each edge is annotated with its matched phosphosites, log2 fold-changes, and binary is_quantified / is_significant flags, writing optimized_network_validated.sif and optimized_object_validated.rds. The rationale is that a sign-coherent CARNIVAL edge is only mechanistically credible for phosphorylation-driven signaling if the target phosphosite was actually quantified (and ideally significant) in the experiment. The same routine is re-applied across all Step 2 parameter combinations and under PKN randomization (BiRewire shuffling at 25/50/100%), where phosphorylation-edge coverage of is_quantified / is_significant edges serves as the degradation metric. In the murine ac220 validation, this annotation supports computing the fraction of gold-standard phosphorylation events recapitulated in the network against the 33-protein / 7-phenotype gold standard.

## Related tools

- **SignalingProfiler 2.0** (Provides expand_and_map_edges, the routine that annotates each optimized network edge with matched phosphosites, fold-changes and is_quantified/is_significant flags) — github.com/SaccoPerfettoLab/SignalingProfiler
- **CARNIVAL** (Upstream ILP optimization producing the sign-coherent context-specific network (optimized_network.sif / optimized_object.rds) that the phosphosite mapping validates) — saezlab.github.io/CARNIVAL
- **PhosphoSitePlus** (Source of kinase-substrate/regulatory-site relations underlying the PKN edges onto which phosphoproteomics is mapped)

## Examples

```
optimized_validated <- expand_and_map_edges(network = optimized_object, phos = phosphoproteomics) # writes optimized_network_validated.sif and optimized_object_validated.rds
```

## Evaluation signals

- Output files optimized_network_validated.sif and optimized_object_validated.rds exist and the SIF edges carry non-empty phosphosite, fold-change, is_quantified and is_significant annotations
- Representative metformin network (99 nodes / 219 edges) recapitulates mTOR inhibition and AMPK activation signatures with edges supported by quantified phosphosites
- Phosphorylation-edge coverage (fraction of is_quantified / is_significant edges) is computed and, under BiRewire PKN shuffling at 25/50/100%, shows the expected reduced coverage relative to the unshuffled PKN
- In the murine ac220 validation, the fraction of gold-standard phosphorylation events recapitulated in the network is comparable to the training-dataset benchmark, and metrics are recorded in murine_ac220_benchmark_results.csv

## Limitations

- Phosphorylation-edge coverage degrades when the prior knowledge network is randomized (BiRewire edge shuffling at 25–100%), so mapped-edge support is only as good as the PKN's edge specificity
- PhosphoSitePlus Regulatory_sites and Kinase_Substrates files must be manually downloaded to include PsP information in the PKN
- Regulon and causal-interaction resources cover <50% (~9,000 proteins) of the UniProt-SwissProt proteome, limiting which edges can ever receive phosphosite support

## Evidence

- [methods] Map phosphoproteomics data onto the signaling edges with expand_and_map_edges to annotate each edge with matched phosphosites, fold-changes, is_quantified and is_significant flags: [evidence span withheld — anchor status: not_found]
- [results] phosphorylation-edge coverage (is_quantified / is_significant edges) for PKN shuffling: [evidence span withheld — anchor status: not_found]
- [results] the representative network recapitulates the expected perturbation signatures with edges supported by quantified phosphosites: [evidence span withheld — anchor status: not_found]
- [discussion] sole tool capable of estimating the activity of proteins, aside from kinases and phosphatases, from the phosphoproteomic data: "is the sole tool capable of estimating the activity of proteins, aside from kinases and phosphatases, from the phosphoproteomic data"
