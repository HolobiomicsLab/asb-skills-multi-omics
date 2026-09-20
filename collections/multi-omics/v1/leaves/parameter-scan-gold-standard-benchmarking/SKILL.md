---
name: parameter-scan-gold-standard-benchmarking
description: Use when a network inference pipeline exposes tunable parameters (e.g.,
  PhenoScore switches, ProxPath path_length, zscore_threshold) and a reference phenotype
  activity profile exists to judge which setting recapitulates expected phenotype
  signs.
when_to_use_negative:
- The analysis goal is metabolomic data integration — SignalingProfiler 2.0 explicitly
  does not include metabolomic data, unlike COSMOS
- Input is raw multi-omics data not yet processed by Step 1 (PhosphoScore/VIPER protein
  activity) and Step 2 (CARNIVAL-optimized context-specific network) — the scan requires
  a validated upstream network object
- No independent phenotypic gold standard exists for the perturbation, since parameter
  selection and validation both depend on comparing predicted phenotype activity signs
  against a reference profile
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_0121
tools:
- name: SignalingProfiler 2.0
  role: Hosts the PhenoScore computation, the 96-condition parameter scan switches,
    and the optimize_pheno_network function that re-runs optimization on phenotypic
    activity
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: ProxPath
  role: Provides the protein-phenotype distance table from which significantly close
    phenotypes are selected (path_length, stat, zscore_threshold) before PhenoScore
    computation
- name: CARNIVAL
  role: ILP optimizer used by optimize_pheno_network with cplex solver (mipGAP=0.05,
    timelimit=3600, cplexMemoryLimit=8192) to build the final phenotype-optimized
    network
  repo: https://saezlab.github.io/CARNIVAL/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:933dcd7cac98678ff0a84e98d93077335f4b260e3d33badcdc450f3357218402
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - CARNIVAL
  - ProxPath
  - SignalingProfiler 2.0
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# parameter-scan-gold-standard-benchmarking

## Summary

Scans PhenoScore/ProxPath parameter settings across multiple conditions and scores each against a phenotypic gold standard to select the best-performing configuration, then re-optimizes the final network on phenotype activity with CARNIVAL. This is how SignalingProfiler 2.0 was benchmarked to obtain the final 109-node/298-edge metformin model.

## When to use

Use when a network inference pipeline exposes tunable parameters (e.g., PhenoScore switches, ProxPath path_length, zscore_threshold) and a reference phenotype activity profile exists to judge which setting best recapitulates expected phenotype signs.

## When NOT to use

- The analysis goal is metabolomic data integration — SignalingProfiler 2.0 explicitly does not include metabolomic data, unlike COSMOS
- Input is raw multi-omics data not yet processed by Step 1 (PhosphoScore/VIPER protein activity) and Step 2 (CARNIVAL-optimized context-specific network) — the scan requires a validated upstream network object
- No independent phenotypic gold standard exists for the perturbation, since parameter selection and validation both depend on comparing predicted phenotype activity signs against a reference profile

## Inputs

- Step 2 validated network object (nodes_df, edges_df, igraph_network, sp_graph)
- ProxPath protein-phenotype distance table preprocessed against the quantified proteome and phosphoproteome
- Phenotypic gold standard (e.g., 10 phenotypes with expected up/down activity signs, such as apoptosis/autophagy up and proliferation/biosynthesis down for metformin)
- ILP solver path (manually specified, e.g., CPLEX)

## Outputs

- Top-performing PhenoScore/ProxPath parameter combination from the scan (96 conditions in the metformin case)
- Phenotype-optimized network written to pheno_opt1.sif and pheno_opt1.rds (109 nodes, 298 edges for the metformin model)
- Inferred phenotype activity table (phenoscore < 0 interpreted as down, otherwise up)

## How to apply

Load the Step 2 validated network (nodes_df, edges_df, igraph_network) and the ProxPath protein-phenotype distance table preprocessed against the quantified proteome and phosphoproteome. For each parameter condition, select significantly close phenotypes using path_length 2-4, stat='mean', and zscore_threshold=-1.96, then apply the randomization filter by sampling n_random=1000 random protein lists of equal size and discarding phenotypes whose activating/inhibitory path counts are matched by random lists at pvalue_threshold=0.05. Compute the PhenoScore as the weighted average of regulator activities under each condition's switches (remove_cascade=TRUE, node_idx=TRUE/FALSE, use_carnival_activity=TRUE/FALSE), using the sp_graph to keep only the most downstream regulator in cascades. Score each condition's predicted phenotype activities against the gold standard (e.g., 10 phenotypes) and select the top combination. Re-optimize the final model on phenotypic activity with optimize_pheno_network using CARNIVAL (default_CARNIVAL_options with cplex solver, mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192).

## Related tools

- **SignalingProfiler 2.0** (Hosts the PhenoScore computation, the 96-condition parameter scan switches, and the optimize_pheno_network function that re-runs optimization on phenotypic activity) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **ProxPath** (Provides the protein-phenotype distance table from which significantly close phenotypes are selected (path_length, stat, zscore_threshold) before PhenoScore computation)
- **CARNIVAL** (ILP optimizer used by optimize_pheno_network with cplex solver (mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192) to build the final phenotype-optimized network) — https://saezlab.github.io/CARNIVAL/

## Examples

```
pheno_net <- optimize_pheno_network(sp_network, carnival_options = default_CARNIVAL_options(solver='cplex', mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192))
```

## Evaluation signals

- The reproduced phenotype-optimized network matches the reference model size (109 nodes, 298 edges for the metformin case)
- The PhenoScore profile recapitulates the expected gold standard signs: apoptosis/autophagy up and proliferation/biosynthesis down
- output_matches_reference against the published phenotype-optimized model passes
- Phenotypes surviving the randomization filter have activating/inhibitory path counts not matched by 1000 random protein lists of equal size at pvalue_threshold=0.05
- Selected parameter setting generalizes: the article reports that training-derived settings exhibited comparable performance on murine and human validation datasets, indicating limited overfitting

## Limitations

- Incomplete prior knowledge coverage: regulon and causal interaction resources cover <50% (about 9,000 proteins) of the UniProt-SwissProt proteome, restricting phenotype inference scope
- The ILP solver path must be manually specified by the user before CARNIVAL optimization can run
- Metabolomic data cannot be integrated (limitation relative to COSMOS), and epigenetic, acetylomic, and ubiquitylomic data cannot be integrated into the signaling model
- SignalingProfiler is under active development, so the README suggests deleting and re-installing the package regularly to exploit new features, which may change parameter behavior between runs

## Evidence

- [intro] Step 3 parameter scan: "Step 3 The context-specific network is connected to cellular phenotypes using the ProxPath algorithm24 and the phenotype activity is obtained by integrating upstream protein activities."
- [methods] randomization filter rationale: "we sample a random protein list with the same number of proteins and compute the number of activating and inhibitory paths over the phenotype of interest"
- [methods] remove_cascade decision point: "if two independent regulators are connected in the network (`sp_graph`), consider the most downstream (`remove_cascade = TRUE`)"
- [discussion] gold-standard parameter selection validated: "the parameter settings determined during the training process exhibited comparable performance (Fig. 6A) when applied to the validation datasets, demonstrating their broad applicability and limited"
- [readme] installation route: "devtools::install_github('https://github.com/SaccoPerfettoLab/SignalingProfiler/')"
