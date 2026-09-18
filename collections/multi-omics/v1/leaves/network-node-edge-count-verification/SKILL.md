---
name: network-node-edge-count-verification
description: Use when a multiplex network RData object (e.g. from RWR_make_multiplex
  with delta=0.5) must be validated after construction or reload, by checking that
  Pool_of_Nodes and aggregated edge counts match expected totals.
when_to_use_negative:
- Input is a single-layer (monoplex) network with no multiplex mpo object or Pool_of_Nodes
  to check
- You only need layer-level statistics (e.g. pairwise between-layer Jaccard similarity),
  which RWR_netstats provides directly
- Layer edge-lists have changed intentionally and old count baselines no longer apply
  — recompute rather than verify against the 26,605/918,640 reference
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: RWRtoolkit
  role: Provides RWR_make_multiplex (delta=0.5) to build the multiplex whose node
    and edge counts are verified, and produces the saved RData object
  repo: https://github.com/dkainer/RWRtoolkit
- name: R
  role: Runtime environment (R >= 4.1.0) in which the RData object is loaded and Pool_of_Nodes
    / edge counts are inspected
  repo: https://github.com/dkainer/RWRtoolkit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:94bb50710a9893e72cfa84570be605a4e972da19024238ace7a16bde6e9fcd3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - R
  - RWRtoolkit
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# network-node-edge-count-verification

## Summary

Verifies the structural integrity of a constructed multiplex network object by confirming its unique node count (Pool_of_Nodes) and aggregated edge count against expected values, as demonstrated for the 9-layer comprehensive Arabidopsis thaliana multiplex (26,605 nodes, 918,640 edges).

## When to use

Use when a multiplex network RData object (e.g. from RWR_make_multiplex with delta=0.5) must be validated after construction or reload, by checking that Pool_of_Nodes and aggregated edge counts match expected totals.

## When NOT to use

- Input is a single-layer (monoplex) network with no multiplex mpo object or Pool_of_Nodes to check
- You only need layer-level statistics (e.g. pairwise between-layer Jaccard similarity), which RWR_netstats provides directly
- Layer edge-lists have changed intentionally and old count baselines no longer apply — recompute rather than verify against the 26,605/918,640 reference

## Inputs

- Saved multiplex RData object (nw.mpo, nw.adj, nw.adjnorm), e.g. Comprehensive_Network_AT_d0.5_v02.RData
- flist descriptor file listing each layer's edge-list path, layer name, and layer code (DU, GA, KS, PP, PX, PY, RE, RP, RX)
- Nine A. thaliana network layer edge-list files from the dkainer/RWRtoolkit-data repository

## Outputs

- Verified unique node count for the multiplex Pool_of_Nodes (expected 26,605)
- Verified aggregated network edge count (expected 918,640)
- Pass/fail validation status of the multiplex object

## How to apply

After running RWR_make_multiplex on the flist descriptor (listing the nine A. thaliana layer edge-lists with codes DU, GA, KS, PP, PX, PY, RE, RP, RX and delta = 0.5), save the resulting objects as Comprehensive_Network_AT_d0.5_v02.RData containing nw.mpo, nw.adj, and nw.adjnorm. Reload the saved RData and inspect the mpo object: count unique entries in the multiplex Pool_of_Nodes and count edges in the aggregated network, expecting 26,605 unique nodes and 918,640 edges for the comprehensive Arabidopsis multiplex. Because RWRtoolkit parallelizes multiplex construction, the build should complete in seconds (~59 s for this network) rather than hours, providing a secondary timing signal that the correct code path ran. Mismatched counts indicate a missing or duplicated layer in the flist or a stale RData file.

## Related tools

- **RWRtoolkit** (Provides RWR_make_multiplex (delta=0.5) to build the multiplex whose node and edge counts are verified, and produces the saved RData object) — https://github.com/dkainer/RWRtoolkit
- **R** (Runtime environment (R >= 4.1.0) in which the RData object is loaded and Pool_of_Nodes / edge counts are inspected) — https://github.com/dkainer/RWRtoolkit

## Examples

```
Rscript -e "load('Comprehensive_Network_AT_d0.5_v02.RData'); cat(length(nw.mpo$Pool_of_Nodes), '\n'); cat(length(E(nw.mpo$graph)), '\n')"
```

## Evaluation signals

- Reloaded RData exposes all three objects: nw.mpo, nw.adj, and nw.adjnorm
- length(nw.mpo$Pool_of_Nodes) equals 26,605 for the comprehensive 9-layer Arabidopsis multiplex
- Aggregated network edge count equals 918,640
- Construction completed in seconds (reference: ~59 s), consistent with RWRtoolkit's parallelized build rather than the hours-long unoptimized path
- The flist resolves exactly nine layers with codes DU, GA, KS, PP, PX, PY, RE, RP, RX and no layer is dropped or duplicated

## Limitations

- Counts are specific to the comprehensive 9-layer network at delta = 0.5, version 0.2; other multiplexes (e.g. Close to Phenotype or Expression/Regulation) have different baselines
- Prior to RWRtoolkit's parallelization, RandomWalkRestartMH multiplex construction was unoptimized and computational load increased exponentially with network size and layer count, so timings on the legacy path differ greatly
- No version number is stated for RWRtoolkit/RandomWalkRestartMH in the article, so reproducibility depends on the archived RData object

## Evidence

- [results] The full comprehensive Arabidopsis multiplex, built in 59 seconds, contains 26,605 unique nodes and 918,640 edges.: [evidence span withheld — anchor status: not_found]
- [methods] load the saved RData object and verify the multiplex Pool_of_Nodes contains 26,605 unique nodes and the aggregated network contains 918,640 edges: [evidence span withheld — anchor status: not_found]
- [discussion] With RWRtoolkit we parallelized this process to dramatically reduce the time to construct a large multiplex network from hours to seconds: "With RWRtoolkit we parallelized this process to dramatically reduce the time to construct a large multiplex network from hours to seconds"
- [readme] Contains the following network layers with unweighted edges and delta value = 0.5: "Contains the following network layers with unweighted edges and delta value = 0.5"
