---
name: parallel-benchmark-timing-comparison
description: Use when a package update claims to parallelize a previously sequential
  construction or computation step and you must quantify the speedup (e.g. multiplex
  network construction time for large L-layer networks).
when_to_use_negative:
- The goal is to benchmark RWR query execution (Random.Walk.Restart.Multiplex iterations)
  rather than multiplex network construction
- The networks are small (low layer count L and node count N), where the exponential
  computational load of sequential construction does not yet dominate and a timeout
  comparison is uninformative
- The task is correctness/validation of RWR rankings (e.g. tau seed-weighting behavior)
  rather than timing performance
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: RWRtoolkit
  role: Provides the updated parallelized multiplex-generation functions (create.multiplex,
    compute.adjacency.matrix, compute.transition.matrix) whose construction time is
    benchmarked against the original implementation
  repo: https://github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Provides the original sequential baseline implementation (RandomWalkRestartMHOriginal)
    that times out on large (L,N) combinations
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: igraph
  role: Used to generate the L random Erdős–Rényi networks of size N that serve as
    synthetic benchmark input layers
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:a6e88099222661a172a40e6f5319743e10e3ca8689ef1c2bb00ed6b86bb916a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - R
  - RWRtoolkit
  - RandomWalkRestartMH
  - igraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# parallel-benchmark-timing-comparison

## Summary

Benchmark a parallelized software implementation against its original sequential predecessor by recording per-step wall-clock times (e.g. multiplex construction via create.multiplex, compute.adjacency.matrix, compute.transition.matrix) on synthetic grid networks, demonstrating speedups such as hours-to-seconds for large multiplex networks.

## When to use

Use when a package update claims to parallelize a previously sequential construction or computation step and you must quantify the speedup (e.g. multiplex network construction time for large L-layer networks).

## When NOT to use

- The goal is to benchmark RWR query execution (Random.Walk.Restart.Multiplex iterations) rather than multiplex network construction
- The networks are small (low layer count L and node count N), where the exponential computational load of sequential construction does not yet dominate and a timeout comparison is uninformative
- The task is correctness/validation of RWR rankings (e.g. tau seed-weighting behavior) rather than timing performance

## Inputs

- L random Erdős–Rényi networks of N nodes each generated with the igraph package (e.g. L=50, N=5000)
- Original sequential implementation (RandomWalkRestartMH) with functions create.multiplex, compute.adjacency.matrix, compute.transition.matrix
- Updated parallelized implementation (RWRtoolkit) run with a specified core count (e.g. 32 cores)
- Timeout threshold for the sequential run (e.g. 6 hours)

## Outputs

- Per-step wall-clock timing table for both implementations (e.g. multiplex_benchmark_timings.csv)
- Summary contrast: sequential construction exceeding the timeout vs. parallelized construction completing in seconds (e.g. ~17.69 s for (L,N)=(50,5000))

## How to apply

Follow the article's grid-computation design: for a chosen (L,N) combination — here L=50 layers and N=5000 nodes per layer — generate L random networks of size N using the Erdős–Rényi model in the igraph package. Run the original single-threaded implementation (RandomWalkRestartMHOriginal) through create.multiplex, compute.adjacency.matrix, and compute.transition.matrix, recording wall-clock time per step under a timeout (6 hours in the reference benchmark). Run the identical three multiplex-generation functions with the updated RWRtoolkit implementation using 32 cores for supra-adjacency matrix computation, recording each step's wall-clock time. Compile per-step timings for both implementations into a table (e.g. multiplex_benchmark_timings.csv) showing the contrast between the sequential timeout and the parallelized completion time. Rationale: the article reports the original RandomWalkRestartMH multiplex construction was not optimized and its computational load increases exponentially with network size and layer count, so a fixed grid point plus a hard timeout makes the sequential bottleneck observable and comparable against the parallelized run.

## Related tools

- **RWRtoolkit** (Provides the updated parallelized multiplex-generation functions (create.multiplex, compute.adjacency.matrix, compute.transition.matrix) whose construction time is benchmarked against the original implementation) — https://github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Provides the original sequential baseline implementation (RandomWalkRestartMHOriginal) that times out on large (L,N) combinations) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **igraph** (Used to generate the L random Erdős–Rényi networks of size N that serve as synthetic benchmark input layers)

## Examples

```
library(RWRtoolkit); mpo <- RWRtoolkit::RWR_make_multiplex(flist=flist)
```

## Evaluation signals

- The timing table must reproduce the reported contrast: original single-threaded construction exceeds the 6-hour timeout while the 32-core parallelized construction completes in approximately 17.69 seconds for (L,N)=(50,5000)
- Per-step timings are recorded separately for create.multiplex, compute.adjacency.matrix, and compute.transition.matrix in both implementations
- The same grid-computation design (L=50 random Erdős–Rényi networks of N=5000 nodes each from igraph) is applied to both implementations so the comparison is apples-to-apples
- The sequential run terminates at the applied timeout (6 hours) rather than completing, matching the exponential-load expectation stated in the article
- Output is written to a file (e.g. multiplex_benchmark_timings.csv) with columns distinguishing implementation and construction step

## Limitations

- The original RandomWalkRestartMH multiplex construction was not optimized; computational load increases exponentially with network size and layer count, so sequential timings at large (L,N) can only be reported as timeout-censored values
- Benchmark results depend on the chosen grid point (L from 10-50 layers, N from 50-5000 nodes) and on the number of cores used for the parallelized supra-adjacency computation
- The article provides no version number for RWRtoolkit or RandomWalkRestartMH, so exact reproducibility of absolute timings may vary across software versions

## Evidence

- [discussion] With RWRtoolkit we parallelized this process to dramatically reduce the time to construct a large multiplex network from hours to seconds: "With RWRtoolkit we parallelized this process to dramatically reduce the time to construct a large multiplex network from hours to seconds"
- [discussion] the construction of the multiplex was not optimized. As the size of networks and the total number of layers increases, the computational load for constructing the multiplex increases exponentially.: "the construction of the multiplex was not optimized. As the size of networks and the total number of layers increases, the computational load for constructing the multiplex increases exponentially."
- [methods] For each combination (L,N), we generated L random networks of size N using the Erdős–Rényi model in the igraph package: "For each combination (L,N), we generated L random networks of size N using the Erdős–Rényi model in the igraph package"
- [methods] create.multiplex, which constructs the multiplex network list of igraph objects: "create.multiplex, which constructs the multiplex network list of igraph objects"
- [methods] compute.adjacency.matrix, which generates the supra-adjacency matrix for the multiplex: "compute.adjacency.matrix, which generates the supra-adjacency matrix for the multiplex"
