---
name: benchmark-precision-recall-evaluation-against-gold-standard
description: Use when you have run SignalingProfiler 2.0 on an independent validation
  dataset (e.g. murine AML ac220 FLT3-TKI multi-omics) and need to quantify, against
  a curated gold standard, whether default parameters generalize or overfit.
when_to_use_negative:
- The dataset is metabolomic or epigenetic/acetylomic/ubiquitylomic — SignalingProfiler
  2.0 does not include metabolomic data and these regulation types cannot be integrated
  into the signaling model, so its benchmark outputs would be meaningless
- No independently curated gold standard (validated protein activities, phosphorylation
  events, and phenotypes) exists for the dataset — precision, recall, and phosphorylation-recapitulation
  fractions cannot be computed
- You are parameter tuning on the training dataset itself rather than testing generalization
  — the skill's purpose is to detect overfitting on held-out validation data (murine
  ref. 20, human ref. 39)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SignalingProfiler 2.0
  role: Pipeline whose default-parameter outputs (protein activities, network nodes/edges,
    phenotype scores) are benchmarked against the gold standard
  repo: github.com/SaccoPerfettoLab/SignalingProfiler
- name: VIPER
  role: Footprint-based inference of TF/protein activities whose predictions are scored
    for precision/recall against the gold-standard protein set
- name: CARNIVAL
  role: ILP optimization (two-shots, unioned) producing the context-specific network
    whose nodes/edges are evaluated for gold-standard recapitulation
  repo: saezlab.github.io/CARNIVAL
- name: ProxPath
  role: Connects the context-specific network to the 7 gold-standard phenotypes used
    in the benchmark comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:6986a541437c1ec667a9a7449604e0dc8ba8fdb482f68f413bec10f1170a2dbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - CARNIVAL
  - PhosphoSitePlus
  - ProxPath
  - SIGNOR
  - SignalingProfiler 2.0
  - VIPER
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# Benchmark precision/recall evaluation against gold standard

## Summary

Quantifies how faithfully a SignalingProfiler 2.0-derived context-specific network recapitulates an independently curated gold standard (e.g. the 33-protein / 7-phenotype murine AML ac220 reference) by computing precision, recall, and the fraction of gold-standard phosphorylation events reproduced. It was used to demonstrate that training-derived default parameters generalize without overfitting to validation datasets.

## When to use

Use when you have run SignalingProfiler 2.0 on an independent validation dataset (e.g. murine AML ac220 FLT3-TKI multi-omics) and need to quantify, against a curated gold standard, whether default parameters generalize or overfit.

## When NOT to use

- The dataset is metabolomic or epigenetic/acetylomic/ubiquitylomic — SignalingProfiler 2.0 does not include metabolomic data and these regulation types cannot be integrated into the signaling model, so its benchmark outputs would be meaningless
- No independently curated gold standard (validated protein activities, phosphorylation events, and phenotypes) exists for the dataset — precision, recall, and phosphorylation-recapitulation fractions cannot be computed
- You are parameter tuning on the training dataset itself rather than testing generalization — the skill's purpose is to detect overfitting on held-out validation data (murine ref. 20, human ref. 39)

## Inputs

- SignalingProfiler 2.0 outputs on the validation dataset: inferred TF and kinase/phosphatase activities (VIPER footprint-based and KSEA/PhosphoScore), context-specific CARNIVAL network with phosphoproteomics-annotated edges (expand_and_map_edges), and phenotype scores for the 7 gold-standard phenotypes
- Curated gold standard for the validation dataset (33 proteins / 7 phenotypes for the murine ac220 case, ref. 20), including gold-standard phosphorylation events
- Deduplicated murine AML ac220 transcriptomics, proteomics, and phosphoproteomics tables (one occurrence per phosphosite)

## Outputs

- Precision and recall of predicted protein activities and network nodes/edges against the gold standard
- Fraction of gold-standard phosphorylation events recapitulated in the predicted network
- Benchmark results table (e.g. murine_ac220_benchmark_results.csv) with metrics under default vs. reference parameters

## How to apply

Run the full SignalingProfiler 2.0 pipeline with default (training-derived) options — run_footprint_based_analysis (TFEA reg_minsize=10; KSEA reg_minsize=5, integrated_regulons=TRUE), phosphoscore_computation, a two-layer naïve network (max_length_1=3, max_length_2=4, connect_all=TRUE) rooted at the perturbed receptor (Flt3), two-shots CARNIVAL (solver='cplex', mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192) unioned via union_of_graphs, and phenoscore_computation (path_length=3, stat='mean', zscore_threshold=-1.96, n_random=1000, pvalue_threshold=0.05, remove_cascade=TRUE). Then compare the predicted protein activities, network nodes/edges, and phenotype scores against the 33-protein / 7-phenotype murine gold standard, computing precision, recall, and the fraction of gold-standard phosphorylation events recapitulated in the network. Judgment criterion: metrics from default parameters must be comparable to the reference benchmark performance reported for the validation dataset (as in Fig. 6A of the article), demonstrating broad applicability and limited overfitting; record all values (e.g. in murine_ac220_benchmark_results.csv).

## Related tools

- **SignalingProfiler 2.0** (Pipeline whose default-parameter outputs (protein activities, network nodes/edges, phenotype scores) are benchmarked against the gold standard) — github.com/SaccoPerfettoLab/SignalingProfiler
- **VIPER** (Footprint-based inference of TF/protein activities whose predictions are scored for precision/recall against the gold-standard protein set)
- **CARNIVAL** (ILP optimization (two-shots, unioned) producing the context-specific network whose nodes/edges are evaluated for gold-standard recapitulation) — saezlab.github.io/CARNIVAL
- **ProxPath** (Connects the context-specific network to the 7 gold-standard phenotypes used in the benchmark comparison)

## Examples

```
tfea_regulons <- create_tfea_regulons(resources = c('SIGNOR', 'Dorothea'), organism = 'mouse'); ksea_regulons <- create_ksea_regulons(resources = c('SIGNOR', 'Omnipath'), organism = 'mouse')
```

## Evaluation signals

- Precision and recall of predicted protein activities against the 33-protein murine gold standard fall in the range reported for the reference benchmark of the validation dataset (Fig. 6A)
- The fraction of gold-standard phosphorylation events recapitulated in the predicted network is comparable to the reference benchmark value
- Metrics obtained with default (training-derived) parameters are comparable to those from the tuned benchmark, demonstrating limited overfitting
- All metric values are recorded in murine_ac220_benchmark_results.csv for reproducibility
- Phenotype scores for all 7 gold-standard phenotypes are produced with the specified phenoscore_computation settings (path_length=3, zscore_threshold=-1.96, n_random=1000)

## Limitations

- Regulon databases and causal interaction resources cover <50% (~9,000 proteins) of the UniProt-SwissProt proteome, limiting the recall ceiling of any benchmark comparison
- PsP Regulatory_sites and Kinase_Substrates files must be manually downloaded, and the ILP solver path manually specified, before the benchmarked pipeline can run
- Generalization was only demonstrated for the specific validation datasets (murine ref. 20 and human ref. 39); overfitting risk on other contexts is not excluded

## Evidence

- [abstract] Through a benchmarking process on three proof-of-concept studies, we demonstrate the tool's ability to generate hierarchical mechanistic networks recapitulating novel and known perturbed signaling: "Through a benchmarking process on three proof-of-concept studies, we demonstrate the tool's ability to generate hierarchical mechanistic networks recapitulating novel and known perturbed signaling"
- [discussion] the parameter settings determined during the training process exhibited comparable performance (Fig. 6A) when applied to the validation datasets, demonstrating their broad applicability and limited: "the parameter settings determined during the training process exhibited comparable performance (Fig. 6A) when applied to the validation datasets, demonstrating their broad applicability and limited"
- [discussion] a benchmarking approach consisting of a parameter tuning phase on a training dataset30 and a subsequent validation phase on murine20 and human39 datasets: "a benchmarking approach consisting of a parameter tuning phase on a training dataset30 and a subsequent validation phase on murine20 and human39 datasets"
- [readme] Massacci, G., Venafra, V., Latini, S. et al. A key role of the WEE1-CDK1 axis in mediating TKI-therapy resistance in FLT3-ITD positive acute myeloid leukemia patients. Leukemia (2022).: [evidence span withheld — anchor status: not_found]
