---
name: network-graph-file-serialization
description: Use when a flux-scored edge list and the GGN from pyPARAGON's PageRank-flux
  step are ready and the tau-scaled edge subset has been selected, requiring the final
  context-specific network C(VC, EC) to be written to a GraphML file.
when_to_use_negative:
- The GGN or flux-scored edge list has not yet been produced (GGN construction and
  PPR/flux scoring are upstream prerequisites)
- The task is Louvain community detection or hypergeometric module enrichment on an
  already-inferred network — use pyPARAGON's community analysis module instead
- A different reference interactome or a confidence-threshold prefiltered network
  is needed — the article states the reference interactome is not prefiltered by confidence
  threshold
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: pyPARAGON
  role: Performs the tau x F edge selection on the GGN and serializes the resulting
    context-specific network C(VC, EC) to context_specific_network.graphml
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Runtime environment (e.g. python3 -m venv pyPARAGONenv) in which pyPARAGON
    and NetworkX write the GraphML output
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:4005863d263f30ea7451198cf791785f7f7d362c8b0dcf65930049475843caf7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - Python
  - pyPARAGON
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# network-graph-file-serialization

## Summary

Serialize pyPARAGON's context-specific subnetwork C(VC, EC) — the edges selected from the graphlet-guided network (GGN) by the tau x F flux threshold — into a GraphML graph file (context_specific_network.graphml) using pyPARAGON with Python/NetworkX, so the inferred network can be stored, validated, and reused.

## When to use

Use when a flux-scored edge list and the GGN from pyPARAGON's PageRank-flux step are ready and the tau-scaled edge subset has been selected, requiring the final context-specific network C(VC, EC) to be written to a GraphML file.

## When NOT to use

- The GGN or flux-scored edge list has not yet been produced (GGN construction and PPR/flux scoring are upstream prerequisites)
- The task is Louvain community detection or hypergeometric module enrichment on an already-inferred network — use pyPARAGON's community analysis module instead
- A different reference interactome or a confidence-threshold prefiltered network is needed — the article states the reference interactome is not prefiltered by confidence threshold

## Inputs

- Flux-scored GGN edge list (edges with PageRank-flux scores from pyPARAGON step ii)
- Graphlet-guided network (GGN) produced by pyPARAGON's graphlet module
- Tau scaling factor and total flux F for the tau x F cumulative-flux threshold (Eqs. 5-6)

## Outputs

- context_specific_network.graphml — GraphML file encoding the context-specific subnetwork C(VC, EC) with EC ⊆ EG and VC ⊆ VG

## How to apply

Load the flux-scored edge list and the GGN produced by the Personalized PageRank (PPR) propagation and flux-calculation step, then rank all GGN edges in descending order of flux score. Accumulate edge fluxes from the top-ranked edge downward until the cumulative flux reaches tau x total flux F (Eqs. 5-6), retaining the selected edge set EC and its incident nodes VC as the context-specific subnetwork. Serialize C(VC, EC) to context_specific_network.graphml using pyPARAGON's Python/NetworkX serialization so that EC is a strict subset of the GGN edge set and VC a subset of the GGN nodes. Rationale: pyPARAGON infers the context-specific network by selecting the highly scored edges on the GGN, and the flux calculation is deliberately used as a countermeasure against the curse of hubs, so preserving the exact selected edge set in the serialized file is essential for reproducibility.

## Related tools

- **pyPARAGON** (Performs the tau x F edge selection on the GGN and serializes the resulting context-specific network C(VC, EC) to context_specific_network.graphml) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Runtime environment (e.g. python3 -m venv pyPARAGONenv) in which pyPARAGON and NetworkX write the GraphML output)

## Examples

```
pip install git+https://github.com/metunetlab/pyPARAGON.git
```

## Evaluation signals

- context_specific_network.graphml exists and parses as a valid GraphML file (e.g. loadable by NetworkX)
- Serialized edge set EC is a subset of the input GGN edge set EG, and node set VC is a subset of the GGN nodes VG
- Cumulative flux score of the retained edges meets the tau x F criterion per Eqs. 5-6
- Edge count is strictly smaller than the GGN, consistent with subnetwork reconstruction via selection of highly scored edges
- Running with a different tau yields a monotonically consistent edge count (higher tau → fewer edges retained)

## Limitations

- The reference interactome is not prefiltered by a confidence threshold, so the serialized network inherits all GGN edge confidences
- Hub proteins may dominate the inferred network with unrelated interactions; the flux calculation mitigates but does not eliminate this
- No software version is stated for pyPARAGON, and the README (truncated) does not document a dedicated serialization CLI, so output naming (context_specific_network.graphml) relies on the workflow convention

## Evidence

- [methods] Subnetwork reconstruction with highly scored edges on GGN.: "Finally, pyPARAGON infers the context specific network by selecting the highly scored edges."
- [readme] README step iii description: "The highly weighted edges reported in GGN construct a subnetwork or specific network."
- [discussion] Flux as hub countermeasure: "Similarly, the flux calculation in pyPARAGON is a countermeasure against the curse of hubs beyond"
- [readme] Runtime environment: "python3 -m venv pyPARAGONenv"
