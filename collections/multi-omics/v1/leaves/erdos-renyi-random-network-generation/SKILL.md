---
name: erdos-renyi-random-network-generation
description: Use when synthetic network layers are needed to benchmark or stress-test
  multiplex network construction scalability across a grid of layer counts (L=10–50)
  and node counts (N=50–5000), rather than using real multi-omic data.
when_to_use_negative:
- When the goal is biological interpretation of real multi-omic layers (e.g., AraNet,
  StringDB, YeastNet or Arabidopsis multiplex layers) — Erdős–Rényi layers are synthetic
  benchmarks, not evidence networks
- When networks are small or few-layered (e.g., L=3 layers as in Close_To_Phenotype
  multiplex) where construction is fast and benchmarking the parallelized path adds
  no value
- When running downstream RWR analysis (RWR_CV, RWR_LOE, RWR_ShortestPaths) on curated
  gene sets — random ER layers carry no functional signal for cross-validation
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: igraph
  role: Generates the L random Erdős–Rényi networks of size N for each grid combination
- name: RWRtoolkit
  role: Parallelized multiplex construction (create.multiplex, compute.adjacency.matrix,
    compute.transition.matrix) benchmarked against the original implementation
  repo: github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Original sequential single-threaded implementation used as the timing baseline
    with a 6-hour timeout
  repo: github.com/alberto-valdeolivas/RandomWalkRestartMH
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:d673d1680ba1720e0cdd1a0b50a0a22fe6db4bfaa5a3c4e04153243af811ac61
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - RWRtoolkit
  - RandomWalkRestartMH
  - igraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# erdos-renyi-random-network-generation

## Summary

Generates synthetic random network layers with the Erdős–Rényi model in the R igraph package as input to multiplex-network construction benchmarks. Used in a grid computation over layer count L (10–50) and nodes per layer N (50–5000) to compare sequential RandomWalkRestartMH versus parallelized RWRtoolkit construction times.

## When to use

Use when synthetic network layers are needed to benchmark or stress-test multiplex network construction scalability across a grid of layer counts (L=10–50) and node counts (N=50–5000), rather than using real multi-omic data.

## When NOT to use

- When the goal is biological interpretation of real multi-omic layers (e.g., AraNet, StringDB, YeastNet or Arabidopsis multiplex layers) — Erdős–Rényi layers are synthetic benchmarks, not evidence networks
- When networks are small or few-layered (e.g., L=3 layers as in Close_To_Phenotype multiplex) where construction is fast and benchmarking the parallelized path adds no value
- When running downstream RWR analysis (RWR_CV, RWR_LOE, RWR_ShortestPaths) on curated gene sets — random ER layers carry no functional signal for cross-validation

## Inputs

- Grid parameters: layer count L (10–50) and node number per layer N (50–5000)
- igraph package (R) for Erdős–Rényi random network generation
- flist file (file path, layer name, group triples) describing the generated layers for RWR_make_multiplex/create.multiplex

## Outputs

- L igraph network objects per (L,N) grid combination
- Multiplex network object built via create.multiplex / RWR_make_multiplex
- Per-step wall-clock timing table (e.g., multiplex_benchmark_timings.csv) contrasting original vs. parallelized construction

## How to apply

For each (L,N) combination in the grid computation, generate L random networks of size N using the Erdős–Rényi model in the igraph package (e.g., L=50 layers of N=5000 nodes). Feed the resulting igraph layer list into the multiplex construction pipeline — create.multiplex (builds the multiplex list of igraph objects), compute.adjacency.matrix (builds the supra-adjacency matrix, parallelized over 32 cores in RWRtoolkit), and compute.transition.matrix (normalizes to a transition matrix) — recording wall-clock time per step. Rationale: RandomWalkRestartMH multiplex construction was not optimized and its computational load increases exponentially with network size and layer count, so synthetic ER layers with controlled L and N isolate the scaling behavior. Apply a 6-hour timeout on the original single-threaded implementation so time-outs are recorded as data points, and compare against the parallelized RWRtoolkit timings (e.g., 17.69 s for the 50-layer, 5000-node case).

## Related tools

- **igraph** (Generates the L random Erdős–Rényi networks of size N for each grid combination)
- **RWRtoolkit** (Parallelized multiplex construction (create.multiplex, compute.adjacency.matrix, compute.transition.matrix) benchmarked against the original implementation) — github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Original sequential single-threaded implementation used as the timing baseline with a 6-hour timeout) — github.com/alberto-valdeolivas/RandomWalkRestartMH

## Evaluation signals

- Timing table must reproduce the reported contrast: original single-threaded construction exceeds the 6-hour timeout while 32-core parallelized construction completes in approximately 17.69 seconds for (L,N)=(50,5000)
- Each generated layer contains exactly N nodes (e.g., 5000) and is a valid igraph object
- create.multiplex accepts the full L-layer list and produces a multiplex object covering all layers
- compute.transition.matrix yields a row-normalized transition matrix suitable for Random.Walk.Restart.Multiplex iteration
- Timings are recorded per step (create.multiplex, compute.adjacency.matrix, compute.transition.matrix) for both implementations, not only end-to-end

## Limitations

- The original RandomWalkRestartMH multiplex construction was not optimized: computational load increases exponentially as network size and total number of layers increase, requiring a hard timeout (6 hours) for large grids
- No version number is stated for RWRtoolkit or RandomWalkRestartMH in the article, complicating exact reproduction of the benchmark
- Erdős–Rényi layers are synthetic; results speak to construction scalability, not to biological network properties or downstream RWR ranking quality

## Evidence

- [methods] For each combination (L,N), we generated L random networks of size N using the Erdős–Rényi model in the igraph package: "For each combination (L,N), we generated L random networks of size N using the Erdős–Rényi model in the igraph package"
- [discussion] With RWRtoolkit we parallelized this process to dramatically reduce the time to construct a large multiplex network from hours to seconds: "With RWRtoolkit we parallelized this process to dramatically reduce the time to construct a large multiplex network from hours to seconds"
- [discussion] As the size of networks and the total number of layers increases, the computational load for constructing the multiplex increases exponentially.: "As the size of networks and the total number of layers increases, the computational load for constructing the multiplex increases exponentially."
- [methods] create.multiplex, which constructs the multiplex network list of igraph objects: "create.multiplex, which constructs the multiplex network list of igraph objects"
