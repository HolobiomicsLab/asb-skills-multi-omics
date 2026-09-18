---
name: network-propagation-benchmarking
description: Use when comparing network propagation or context-specific subnetwork
  inference tools against a curated pathway gold standard (e.g., NetPath nodes and
  edges) across multiple reference interactomes.
when_to_use_negative:
- Input omic data are single-cell; pyPARAGON integrates bulk omic data for static
  network models, and single-cell extension is only planned
- No curated gold-standard node/edge sets exist for the pathways of interest, making
  node- and edge-level AUPRC/F1 evaluation impossible
- The aim is isoform- or PTM-specific network reconstruction; pyPARAGON generalizes
  alternative copies of hits (protein isoforms, post-translational modifications)
  and that information can be lost
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: pyPARAGON
  role: 'Method under test: GGN construction, Personalized PageRank propagation, and
    flux-based edge scoring for subnetwork reconstruction'
  repo: https://github.com/netlab-ku/pyPARAGON
- name: PL
  role: Baseline tool for node propagation and edge inference comparison on NetPath
    pathways
- name: OI2
  role: Baseline tool for node propagation and edge inference comparison on NetPath
    pathways
- name: DOMINO
  role: Baseline tool for node propagation and edge inference comparison on NetPath
    pathways
- name: Python
  role: Execution environment for pyPARAGON and the benchmark workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:ebd55205ef190d2d43c7c5f18d9e0a61348c4c5406c01e067fdef6f6a333e198
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - DOMINO
  - OI2
  - PL
  - Python
  - pyPARAGON
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# network-propagation-benchmarking

## Summary

Benchmarks network propagation and context-specific network inference methods (pyPARAGON vs PL, Omics Integrator 2, DOMINO) by reconstructing curated NetPath signaling pathways and scoring node-propagation and edge-inference performance with AUPRC and F1.

## When to use

Use when comparing network propagation or context-specific subnetwork inference tools against a curated pathway gold standard (e.g., NetPath nodes and edges) across multiple reference interactomes.

## When NOT to use

- Input omic data are single-cell; pyPARAGON integrates bulk omic data for static network models, and single-cell extension is only planned
- No curated gold-standard node/edge sets exist for the pathways of interest, making node- and edge-level AUPRC/F1 evaluation impossible
- The aim is isoform- or PTM-specific network reconstruction; pyPARAGON generalizes alternative copies of hits (protein isoforms, post-translational modifications) and that information can be lost

## Inputs

- Reference interactomes: ConsensusPathDB, HIPPIE v2.2, HIPPIE v2.3
- NetPath pathway seed gene (omic-hit) sets
- Curated NetPath gold-standard node and edge sets per pathway

## Outputs

- Per tool/interactome/pathway node-level AUPRC and F1 scores
- Per tool/interactome/pathway edge-level AUPRC and F1 scores
- Aggregated node-propagation and edge-inference comparison panels (reproducing Fig. 2B-C)

## How to apply

Load the three reference interactomes (ConsensusPathDB, HIPPIE v2.2, HIPPIE v2.3) and the NetPath pathway seed gene sets into a Python environment with pyPARAGON installed. For each pathway seed set, run pyPARAGON's three-step inference — graphlet-guided network (GGN) construction using nine non-isomorphic graphlets (2–4 nodes) tested against 100 permuted networks by z-test (p < 0.05, z-score > 1.65), Personalized PageRank propagation from the omic-hits, flux-based edge scoring considering PageRank scores, edge confidence, and node degrees — with grid-searched parameters. Run the three baselines PL, OI2, and DOMINO on the same seed sets and interactomes, also with grid-searched parameters, to ensure a fair comparison. For each tool/interactome/pathway combination, compute node-level and edge-level AUPRC and F1 against the curated NetPath gold-standard node and edge sets. Aggregate metrics across pathways and interactomes and render node-level and edge-level comparison panels reproducing Fig. 2B-C of the paper.

## Related tools

- **pyPARAGON** (Method under test: GGN construction, Personalized PageRank propagation, and flux-based edge scoring for subnetwork reconstruction) — https://github.com/netlab-ku/pyPARAGON
- **PL** (Baseline tool for node propagation and edge inference comparison on NetPath pathways)
- **OI2** (Baseline tool for node propagation and edge inference comparison on NetPath pathways)
- **DOMINO** (Baseline tool for node propagation and edge inference comparison on NetPath pathways)
- **Python** (Execution environment for pyPARAGON and the benchmark workflow)

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- pyPARAGON should achieve the highest node-propagation and edge-inference AUPRC/F1 relative to PL, OI2, and DOMINO, matching the reported outperformance on NetPath benchmark pathways
- GGN graphlet significance must use the stated thresholds: z-test p < 0.05 and z-score > 1.65 against 100 permuted networks
- All four tools must be evaluated with grid-searched parameters on the same seed sets and the same three interactomes for a fair comparison
- Metrics must be computed both at node level and edge level against the curated NetPath gold-standard sets, per tool/interactome/pathway combination
- In F1 comparison, expected pattern is pyPARAGON and PL better at propagation, while pyPARAGON and OI2 better at network inference

## Limitations

- No single-cell omic support in the current version; a single-cell extension is only planned for the next version
- Incomplete reference interactomes bias inferred networks toward protein complexes over metabolic pathways, and generic biological processes (e.g., transcription, replication) frequently appear
- The reference interactome is not prefiltered by a confidence threshold, and hub proteins may dominate inferred networks with unrelated interactions (partially counteracted by flux calculation)
- Tool cannot handle alternative copies of hits such as protein isoforms and post-translational modifications; that information is generalized and potentially lost
- No software version stated for pyPARAGON

## Evidence

- [abstract] Through comprehensive evaluations on benchmark signaling pathways, we demonstrate that pyPARAGON outperforms state-of-the-art approaches in node propagation and edge inference.: "Through comprehensive evaluations on benchmark signaling pathways, we demonstrate that pyPARAGON outperforms state-of-the-art approaches in node propagation and edge inference."
- [discussion] we compared our method with three popular tools: PL, OI2, and DOMINO: "we compared our method with three popular tools: PL, OI2, and DOMINO"
- [readme] Using the z-test, we compared the targeted graphlet count in the reference and permuted networks (p < 0.05, z-score > 1.65).: "Using the z-test, we compared the targeted graphlet count in the reference and permuted networks (p < 0.05, z-score > 1.65)."
- [readme] pyPARAGON propagates signals from omic-hits through the personalized PageRank algorithm in re-scoring all proteins in the reference: "pyPARAGON propagates signals from omic-hits through the personalized PageRank algorithm in re-scoring all proteins in the reference"
- [discussion] Thus, the reference interactome is not prefiltered based on a confidence threshold.: "Thus, the reference interactome is not prefiltered based on a confidence threshold."
