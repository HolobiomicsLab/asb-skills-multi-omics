---
name: interactome-dataset-integration
description: Use when a task requires feeding a large prior-knowledge reference interactome
  plus a seed set of omic-hits into a network-propagation pipeline to infer a context-specific
  signaling subnetwork.
when_to_use_negative:
- 'No single-cell omic support: the current version integrates bulk omic data for
  static network models only; single-cell data must wait for the planned extension.'
- 'Hits include protein isoforms or distinct post-translational modifications: pyPARAGON
  cannot handle alternative copies of hits, and isoform/PTM information is generalized
  and potentially lost in the network.'
- The analysis goal is sample-specific dynamic network inference rather than a context-specific
  subnetwork from seed propagation (tools like SWEET target the former).
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: pyPARAGON
  role: 'Main tool: performs graphlet-guided network construction, Personalized PageRank
    propagation, flux-based edge scoring, and subnetwork reconstruction on the integrated
    interactome and seed set'
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Runtime environment for installing and running pyPARAGON (virtualenv setup,
    pip install of the wheel)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:9565b394b813aca72524d9a4a733cdd29051b1ff5c4725681d3f96a68af02819
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# interactome-dataset-integration

## Summary

Integrates curated reference interactomes (e.g., ConsensusPathDB, HIPPIE v2.2/v2.3) with omic-hit seed gene sets as input to pyPARAGON's graphlet-guided network construction, Personalized PageRank propagation, and flux-based edge scoring for context-specific subnetwork inference.

## When to use

Use when a task requires feeding a large prior-knowledge reference interactome plus a seed set of omic-hits into a network-propagation pipeline to infer a context-specific signaling subnetwork.

## When NOT to use

- No single-cell omic support: the current version integrates bulk omic data for static network models only; single-cell data must wait for the planned extension.
- Hits include protein isoforms or distinct post-translational modifications: pyPARAGON cannot handle alternative copies of hits, and isoform/PTM information is generalized and potentially lost in the network.
- The analysis goal is sample-specific dynamic network inference rather than a context-specific subnetwork from seed propagation (tools like SWEET target the former).

## Inputs

- Reference interactome(s) (e.g., ConsensusPathDB, HIPPIE v2.2, HIPPIE v2.3) with edge confidence scores
- Seed node set of omic-hits (e.g., NetPath pathway seed gene sets, phosphoproteomic hits from CPTAC breast cancer samples)
- Optional prior-knowledge annotation sets for module testing (e.g., GOA biological processes, KEGG pathways)

## Outputs

- Graphlet-guided network (GGN) — union of significantly frequent graphlet motifs
- Personalized PageRank-scored nodes and flux-weighted GGN edges
- Context-specific inferred subnetwork of highly scored edges
- Louvain communities with hypergeometric-test-significant module annotations

## How to apply

Load the reference interactome(s) — in the benchmark this meant three interactomes (ConsensusPathDB, HIPPIE v2.2, HIPPIE v2.3) — together with the seed gene sets (e.g., NetPath pathway seed genes or omic-hits from the CPTAC breast cancer phosphoproteomic data of 105 tumors). Do not prefilter the interactome by a confidence threshold; pyPARAGON's graphlet-guided network (GGN) module trims the network by retaining only graphlets of 2-, 3-, and 4-nodes (nine non-isomorphic graphlets G0–G8) whose counts are significant against 100 permuted networks by z-test (p < 0.05, z-score > 1.65). Edge confidence scores are not used for prefiltering but enter the flux calculation, which weights each GGN edge using Personalized PageRank node scores, edge confidence scores, and node interaction counts. Finally, the context-specific subnetwork is reconstructed by selecting highly scored edges on the GGN, and optional Louvain community detection with hypergeometric tests against prior knowledge (GOA, KEGG) yields functional modules.

## Related tools

- **pyPARAGON** (Main tool: performs graphlet-guided network construction, Personalized PageRank propagation, flux-based edge scoring, and subnetwork reconstruction on the integrated interactome and seed set) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Runtime environment for installing and running pyPARAGON (virtualenv setup, pip install of the wheel))

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- The GGN retains only graphlets significant at p < 0.05 and z-score > 1.65 versus 100 permuted networks with the same initial node set — verify significance filtering was applied.
- The reference interactome should NOT have been prefiltered by a confidence threshold; confidence scores must instead appear as terms in the flux calculation.
- Reconstruction of curated NetPath pathways should yield node- and edge-level AUPRC/F1 against gold-standard NetPath node and edge sets, with pyPARAGON outperforming PL, OI2, and DOMINO at node propagation and edge inference.
- The inferred subnetwork should show reduced dominance of highly connected hub nodes relative to the raw reference interactome (graphlet trimming removes unrelated hub interactions).
- Community analysis outputs should correspond to Louvain communities with hypergeometric-significant overrepresentation of prior-knowledge annotations (e.g., GOA biological processes, KEGG pathways).

## Limitations

- Incomplete knowledge in large reference interactomes biases inferred networks toward protein complexes over metabolic pathways.
- Generic biological processes (e.g., transcription, replication) frequently appear in inferred networks, hindering interpretation.
- Hub proteins may dominate the inferred network with unrelated interactions; flux calculation is only a countermeasure against the curse of hubs.
- No software version is stated for pyPARAGON, complicating reproducibility of interactome integration runs.

## Evidence

- [discussion] Thus, the reference interactome is not prefiltered based on a confidence threshold.: "Thus, the reference interactome is not prefiltered based on a confidence threshold."
- [readme] Then, the flux calculation weights each edge of the GGN, considering the PageRank scores of nodes, the confidence scores of edges, and the number of node interactions.: "Then, the flux calculation weights each edge of the GGN, considering the PageRank scores of nodes, the confidence scores of edges, and the number of node interactions."
- [readme] pyPARAGON is a novel method that infers a context specific subnetwork from a given reference network, utilizing omic-hits as a seed node set: "pyPARAGON (PAgeRAnk-flux on Graphlet-guided network for multi-Omic data integratioN) is a novel method that infers a context specific subnetwork from a given reference network, utilizing omic-hits as"
- [readme] Using the z-test, we compared the targeted graphlet count in the reference and permuted networks (p < 0.05, z-score > 1.65). The union of significant graphlets constructs GGN.: "Using the z-test, we compared the targeted graphlet count in the reference and permuted networks (p < 0.05, z-score > 1.65). The union of significant graphlets constructs GGN."
- [discussion] we compared our method with three popular tools: PL, OI2, and DOMINO: "we compared our method with three popular tools: PL, OI2, and DOMINO"
