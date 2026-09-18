---
name: randomization-based-phenotype-significance-testing
description: Use when connecting a Step 2 context-specific network to cellular phenotypes
  via ProxPath and you need to prune spuriously proximal phenotypes before computing
  PhenoScore or re-optimizing a phenotype network with CARNIVAL.
when_to_use_negative:
- You have not yet built a Step 2 context-specific network and lack a condition-specific
  protein list to randomize against
- The phenotype of interest is absent from the ProxPath/SIGNOR phenotype annotation,
  so path counts cannot be computed
- The goal is protein-level (not phenotype-level) activity inference — use the PhosphoScore/VIPER
  Step 1 footprint-based methods instead
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ProxPath
  role: Graph-based tool that computes activating/inhibitory path counts from proteins
    to SIGNOR-annotated phenotypes; the randomization filter validates its proximity
    results
- name: SignalingProfiler 2.0
  role: Pipeline implementing the randomization filter and PhenoScore computation
    in Step 3
  repo: github.com/SaccoPerfettoLab/SignalingProfiler
- name: CARNIVAL
  role: ILP optimization used downstream (default_CARNIVAL_options, cplex solver,
    mipGAP=0.05) to re-optimize the phenotype network after filtering
  repo: saezlab.github.io/CARNIVAL
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:347015604eb8c9aed7ccf6e2bcc0de5ee06105242292b26689013cb469f3f4ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - CARNIVAL
  - ProxPath
  - SignalingProfiler 2.0
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.16666666666666666
n_claims: 6
---

# randomization-based-phenotype-significance-testing

## Summary

A statistical filter within SignalingProfiler 2.0 Step 3 that samples n_random=1000 random protein lists of equal size and discards phenotypes whose activating/inhibitory ProxPath path counts are matched by random lists at pvalue_threshold=0.05, ensuring only significantly close protein-phenotype associations feed the PhenoScore.

## When to use

Use when connecting a Step 2 context-specific network to cellular phenotypes via ProxPath and you need to prune spuriously proximal phenotypes before computing PhenoScore or re-optimizing a phenotype network with CARNIVAL.

## When NOT to use

- You have not yet built a Step 2 context-specific network and lack a condition-specific protein list to randomize against
- The phenotype of interest is absent from the ProxPath/SIGNOR phenotype annotation, so path counts cannot be computed
- The goal is protein-level (not phenotype-level) activity inference — use the PhosphoScore/VIPER Step 1 footprint-based methods instead

## Inputs

- ProxPath protein-phenotype distance table preprocessed against the quantified proteome and phosphoproteome
- Step 2 validated network object (nodes_df, edges_df, igraph_network from toy_opt_network)
- Condition-specific protein list (significantly inferred signaling proteins from Step 2)

## Outputs

- Filtered set of significantly close phenotypes (randomization-filtered ProxPath table)
- Phenotype activity table (down = PhenoScore < 0, up otherwise)
- Phenotype-optimized network (e.g. pheno_opt1.sif and pheno_opt1.rds, 109 nodes / 298 edges for the metformin model)

## How to apply

After selecting significantly close phenotypes from the ProxPath table (path_length 2-4, stat='mean', zscore_threshold=-1.96), sample 1000 random protein lists with the same number of proteins as the condition's protein list and compute how many activating and inhibitory paths reach each phenotype of interest. Discard phenotypes whose observed path counts are matched by the random lists at pvalue_threshold=0.05, retaining only phenotypes that are significantly closer to the inferred signaling proteins than expected by chance. This randomization filter precedes the PhenoScore computation (weighted average of regulator activities under switches such as remove_cascade=TRUE, node_idx=TRUE/FALSE, use_carnival_activity=TRUE/FALSE) and the final phenotype-optimized CARNIVAL run (mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192). The rationale is to prevent random protein-phenotype proximity from inflating phenotype activity estimates in the 109-node/298-edge final metformin model.

## Related tools

- **ProxPath** (Graph-based tool that computes activating/inhibitory path counts from proteins to SIGNOR-annotated phenotypes; the randomization filter validates its proximity results)
- **SignalingProfiler 2.0** (Pipeline implementing the randomization filter and PhenoScore computation in Step 3) — github.com/SaccoPerfettoLab/SignalingProfiler
- **CARNIVAL** (ILP optimization used downstream (default_CARNIVAL_options, cplex solver, mipGAP=0.05) to re-optimize the phenotype network after filtering) — saezlab.github.io/CARNIVAL

## Evaluation signals

- Filtered phenotype set should exclude any phenotype whose activating/inhibitory path counts are matched by >=5% of 1000 random protein lists (pvalue_threshold=0.05)
- Only phenotypes within path_length 2-4 with zscore <= -1.96 (stat='mean') should enter the randomization test
- Reproduced phenotype-optimized network must match the reported final model size (109 nodes, 298 edges)
- PhenoScore profile must recapitulate the expected signs: apoptosis/autophagy up and proliferation/biosynthesis down versus the 10-phenotype gold standard
- Random lists must contain the same number of proteins as the observed condition's list (size-matched sampling)

## Limitations

- Regulon and causal interaction resources are incomplete, covering <50% (~9,000 proteins) of the UniProt-SwissProt proteome, so some phenotypes may lack supporting proteins for path-count testing
- Prior knowledge coverage limits which phenotypes can be tested; ProxPath maps onto phenotypes annotated in SIGNOR only
- Results depend on Step 3 parameter settings tested across 96 conditions; performance of the chosen setting should be judged against a phenotypic gold standard

## Evidence

- [methods] Randomization filter for phenotype path counts: "we sample a random protein list with the same number of proteins and compute the number of activating and inhibitory paths over the phenotype of interest"
- [intro] ProxPath role in Step 3: "we recently developed ProxPath24, a graph-based tool designed to estimate the regulatory impact of proteins on phenotypes annotated in SIGNOR17"
- [intro] Step 3 connects network to phenotypes and infers activity: "Step 3 The context-specific network is connected to cellular phenotypes using the ProxPath algorithm24 and the phenotype activity is obtained by integrating upstream protein activities."
- [readme] Pipeline scope from README: [evidence span withheld — anchor status: not_found]
