---
name: phenotype-activity-scoring-phenoscore
description: Use when a SignalingProfiler 2.0 context-specific network exists and
  you must quantify which cellular phenotypes are significantly activated or inhibited
  by the perturbation, using upstream protein activity scores.
when_to_use_negative:
- Your analysis requires metabolomic data integration — SignalingProfiler 2.0, and
  hence this phenotype scoring step, does not include metabolomic data (unlike COSMOS)
- The analytes of interest are not covered by prior knowledge resources — regulons
  and causal interaction databases cover <50% (about 9,000 proteins) of the UniProt-SwissProt
  proteome, so phenotype paths may be missing for poorly characterized proteins
- You only need network topology without phenotype interpretation — building the CARNIVAL
  context-specific network alone does not require ProxPath scoring
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SignalingProfiler 2.0
  role: 'Pipeline providing phenoscore_computation: integrates upstream protein activities
    into phenotype scores and builds the phenotype-annotated final model'
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: ProxPath
  role: Graph-based tool that connects the context-specific network to cellular phenotypes
    via shortest paths, supplying the table from which significant close proteins
    are selected
- name: CARNIVAL
  role: Produces the sign-coherent context-specific network whose node activities
    feed the phenotype score computation
  repo: https://saezlab.github.io/CARNIVAL/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:0c81d112ee27134891e2ec44845214b6a1d01f62f5c6f8c929f58338051ae679
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# phenotype-activity-scoring-phenoscore

## Summary

Scoring cellular phenotype activities in SignalingProfiler 2.0 by connecting the CARNIVAL-derived context-specific signaling network to hallmark phenotypes via the ProxPath algorithm and integrating upstream protein (kinase/phosphatase/TF) activities into a phenotype-level score.

## When to use

Use when a SignalingProfiler 2.0 context-specific network exists and you must quantify which cellular phenotypes are significantly activated or inhibited by the perturbation, using upstream protein activity scores.

## When NOT to use

- Your analysis requires metabolomic data integration — SignalingProfiler 2.0, and hence this phenotype scoring step, does not include metabolomic data (unlike COSMOS)
- The analytes of interest are not covered by prior knowledge resources — regulons and causal interaction databases cover <50% (about 9,000 proteins) of the UniProt-SwissProt proteome, so phenotype paths may be missing for poorly characterized proteins
- You only need network topology without phenotype interpretation — building the CARNIVAL context-specific network alone does not require ProxPath scoring

## Inputs

- SignalingProfiler 2.0 context-specific network (sp_graph, output of two-shots CARNIVAL / union_of_graphs / expand_and_map_edges)
- Inferred protein activities (TF and kinase/phosphatase scores from run_footprint_based_analysis and phosphoscore_computation)
- Set of target phenotypes (up to ~200 cellular phenotypes from the ProxPath/SIGNOR phenotype layer, e.g. 7 gold-standard phenotypes)
- Phenotype scoring parameters (path_length, stat, zscore_threshold, n_random, pvalue_threshold, remove_cascade)

## Outputs

- Per-phenotype activity scores with z-scores and p-values (e.g., phenotype-level table for 7 gold-standard phenotypes)
- ProxPath table of significantly close proteins for each phenotype (filtered by zscore_threshold, stat)
- Benchmark comparison results (e.g., murine_ac220_benchmark_results.csv with precision/recall against the gold standard)
- Phenotype-annotated final model (network nodes/edges extended with phenotype activity)

## How to apply

Take the sp_graph produced by CARNIVAL (optionally after union_of_graphs and expand_and_map_edges) and connect it to phenotypes using the ProxPath algorithm; select significantly close proteins from the ProxPath table using key parameters zscore_threshold and stat. Call phenoscore_computation with path_length=3, stat='mean', zscore_threshold=-1.96, n_random=1000, pvalue_threshold=0.05, remove_cascade=TRUE, node_idx=FALSE, use_carnival_activity=FALSE. The n_random=1000 parameter drives the null model: random protein lists of the same size are sampled and the number of activating and inhibitory paths over the phenotype of interest is computed, giving a background against which the observed phenotype score is assessed (pvalue_threshold=0.05). If two independent regulators are connected in the network, setting remove_cascade=TRUE keeps only the most downstream regulator so cascades are not double-counted in the phenotype score. Validate the resulting scores against a gold-standard phenotype set (e.g., the 7-phenotype murine ac220 benchmark) and record precision/recall-style comparisons in a benchmark results file.

## Related tools

- **SignalingProfiler 2.0** (Pipeline providing phenoscore_computation: integrates upstream protein activities into phenotype scores and builds the phenotype-annotated final model) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **ProxPath** (Graph-based tool that connects the context-specific network to cellular phenotypes via shortest paths, supplying the table from which significant close proteins are selected)
- **CARNIVAL** (Produces the sign-coherent context-specific network whose node activities feed the phenotype score computation) — https://saezlab.github.io/CARNIVAL/

## Examples

```
phenoscore_computation(sp_graph = sp_graph, path_length = 3, stat = 'mean', zscore_threshold = -1.96, n_random = 1000, pvalue_threshold = 0.05, remove_cascade = TRUE, node_idx = FALSE, use_carnival_activity = FALSE)
```

## Evaluation signals

- All phenotype scores have a p-value derived from n_random=1000 random protein lists, and only phenotypes with p <= 0.05 (pvalue_threshold) are called significant
- Significant phenotype hits satisfy zscore_threshold = -1.96 (z < -1.96) with stat = 'mean' over the ProxPath path scores
- With remove_cascade=TRUE, no two independent regulators connected in sp_graph both appear upstream of the same phenotype — only the most downstream one is retained
- Phenotype scores reproduce the gold standard: in the murine ac220 validation, scores for the 7 gold-standard phenotypes match the reference benchmark within expected precision/recall
- The final model maps phenotype coverage data consistent with the reported phenotype-coverage metrics (Supplementary Data 13, up to 200 cellular phenotypes)

## Limitations

- Regulon databases and causal interaction resources cover <50% (about 9,000 proteins) of the UniProt-SwissProt proteome, limiting phenotype-path availability
- SignalingProfiler 2.0 does not include metabolomic data compared to tools such as COSMOS
- Epigenetic, acetylomic, and ubiquitylomic regulation cannot be integrated into the signaling and represent a future challenge
- Phenotype scores depend on network node activities; using use_carnival_activity=FALSE means input activity estimates, not optimized CARNIVAL values, drive scoring — a choice that affects reproducibility of the benchmark

## Evidence

- [intro] The context-specific network is connected to cellular phenotypes using the ProxPath algorithm24 and the phenotype activity is obtained by integrating upstream protein activities.: "Step 3 The context-specific network is connected to cellular phenotypes using the ProxPath algorithm24 and the phenotype activity is obtained by integrating upstream protein activities."
- [methods] select from the ProxPath table the significantly close proteins next to your phenotypes of interest (key parameters are `zscore_threshold`, `stat`): "select from the ProxPath table the significantly close proteins next to your phenotypes of interest (key parameters are `zscore_threshold`, `stat`)"
- [methods] we sample a random protein list with the same number of proteins and compute the number of activating and inhibitory paths over the phenotype of interest: "we sample a random protein list with the same number of proteins and compute the number of activating and inhibitory paths over the phenotype of interest"
- [methods] if two independent regulators are connected in the network (`sp_graph`), consider the most downstream (`remove_cascade = TRUE`): "if two independent regulators are connected in the network (`sp_graph`), consider the most downstream (`remove_cascade = TRUE`)"
- [discussion] map up to 200 cellular phenotypes onto the final model (Supplementary Data 13): "map up to 200 cellular phenotypes onto the final model (Supplementary Data 13)"
