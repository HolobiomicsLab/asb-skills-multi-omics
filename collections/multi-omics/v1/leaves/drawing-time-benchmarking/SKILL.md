---
name: drawing-time-benchmarking
description: Use when you need to benchmark how rendering/scalability of an automated
  network visualization pipeline (e.g., transomics2cytoscape with Cytoscape) changes
  as the number of stacked network layers increases.
when_to_use_negative:
- You need to reconstruct connectivity data of trans-omic networks automatically —
  the article states these reconstruction procedures 'are not fully automated yet'
- Your pathway data is not in KEGG ID/KGML format — transomics2cytoscape currently
  supports only KEGG, not Reactome, Wikipathways, or SBGN
- You need automated Z-coordinate estimation for layer placement — this is explicitly
  not automated in Case Study I and II
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0605
tools:
- name: transomics2cytoscape
  role: R Bioconductor package that stacks n pathway layers along the Z axis in Cytoscape
    for each timing run
  repo: https://github.com/ecell/transomics2cytoscape
- name: Cytoscape
  role: Network visualization engine whose rendering time is measured; headless under
    xvfb on Linux
  repo: https://github.com/cytoscape/cytoscape/releases/download/3.8.0/cytoscape-unix-3.8.0.tar.gz
- name: RCy3
  role: Cytoscape Automation interface from R used to drive each n-layer rendering
    run (version 2.24.0)
  repo: https://github.com/ecell/transomics2cytoscape
- name: KEGGREST
  role: Retrieves KGML (KEGG pathway XML) for the pathway layers used in the benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:154a59f5a928d93a9c512d9409836e93132371c53cc5ec112d860e5a7b039152
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ecell/transomics2cytoscape
  tools:
  - Cytoscape
  - KEGGREST
  - R
  - RCy3
  - transomics2cytoscape
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# drawing-time-benchmarking

## Summary

Benchmarks the wall-clock drawing time of transomics2cytoscape 2.5D network rendering in Cytoscape as pathway layers are stacked from two to six along the Z axis, verifying that rendering time scales approximately linearly and remains feasible (a few minutes) for realistic trans-omic network sizes.

## When to use

Use when you need to benchmark how rendering/scalability of an automated network visualization pipeline (e.g., transomics2cytoscape with Cytoscape) changes as the number of stacked network layers increases.

## When NOT to use

- You need to reconstruct connectivity data of trans-omic networks automatically — the article states these reconstruction procedures 'are not fully automated yet'
- Your pathway data is not in KEGG ID/KGML format — transomics2cytoscape currently supports only KEGG, not Reactome, Wikipathways, or SBGN
- You need automated Z-coordinate estimation for layer placement — this is explicitly not automated in Case Study I and II

## Inputs

- Case Study I input data from the ecell/transomics2cytoscape GitHub repository (KEGG pathway KGML files and trans-omic edge tables)
- Stack counts n = 2, 3, 4, 5, 6 pathway layers to stack along the Z axis
- Cytoscape 3.8.0/3.10.3 with Cy3D 1.1.3 and KEGGscape 0.9.2 apps, OpenJDK 11, xvfb headless environment

## Outputs

- drawing_time_table.csv with layer-count (n = 2-6) vs wall-clock drawing time measurements
- Validation assessment of approximately linear scaling of drawing time with layer count within the few-minutes regime

## How to apply

Install and launch Cytoscape (version 3.8.0 per the methods, or the paper's tested 3.10.3) with OpenJDK 11, running headless under xvfb on Linux, with the Cy3D 1.1.3 and KEGGscape 0.9.2 Cytoscape apps installed. Load the transomics2cytoscape R package (RCy3 2.24.0) and fetch the Case Study I input data from the ecell/transomics2cytoscape GitHub repository. For each stack count n = 2, 3, 4, 5, 6, run the transomics2cytoscape Case Study I workflow to stack n pathway layers along the Z axis in Cytoscape and measure the wall-clock drawing time of each rendering run. Record layer-count vs drawing-time measurements in drawing_time_table.csv and check that drawing time increases approximately linearly with layer count while remaining within the paper's reported few-minutes visualization regime, implying rendering does not limit realistic use.

## Related tools

- **transomics2cytoscape** (R Bioconductor package that stacks n pathway layers along the Z axis in Cytoscape for each timing run) — https://github.com/ecell/transomics2cytoscape
- **Cytoscape** (Network visualization engine whose rendering time is measured; headless under xvfb on Linux) — https://github.com/cytoscape/cytoscape/releases/download/3.8.0/cytoscape-unix-3.8.0.tar.gz
- **RCy3** (Cytoscape Automation interface from R used to drive each n-layer rendering run (version 2.24.0)) — https://github.com/ecell/transomics2cytoscape
- **KEGGREST** (Retrieves KGML (KEGG pathway XML) for the pathway layers used in the benchmark)

## Examples

```
sudo apt-get install -y openjdk-11-jdk wget xvfb && wget https://github.com/cytoscape/cytoscape/releases/download/3.8.0/cytoscape-unix-3.8.0.tar.gz && tar xf cytoscape-unix-3.8.0.tar.gz
```

## Evaluation signals

- drawing_time_table.csv exists and contains rows for all five stack counts n = 2-6
- Drawing time increases approximately linearly with layer count
- All timings remain within the paper's reported few-minutes visualization regime
- Each run actually renders n stacked pathway layers in Cytoscape via the Cy3D app (2.5D network visible/imported)

## Limitations

- Z-coordinate estimation in Case Study I and II is not automated
- Drawing time is measured on specific software versions (Cytoscape 3.8.0/3.10.3, RCy3 2.24.0, Cy3D 1.1.3, KEGGscape 0.9.2); other versions may differ
- Only KEGG ID and pathway file format supported, so benchmark covers KEGG-based layers only

## Evidence

- [abstract] We confirmed that transomics2cytoscape could be used for rapid visualization of trans-omics networks presented in published papers within a few minutes: "We confirmed that transomics2cytoscape could be used for rapid visualization of trans-omics networks presented in published papers within a few minutes"
- [methods] Install OpenJDK, wget, and xvfb on Linux: "Run `sudo apt-get install -y openjdk-11-jdk wget xvfb` in the terminal."
- [methods] Download Cytoscape 3.8.0 Unix tarball: "wget https://github.com/cytoscape/cytoscape/releases/download/3.8.0/cytoscape-unix-3.8.0.tar.gz"
- [discussion] Stack pathways as layers into a 2.5D network in Cytoscape: "automate40 importing multiple pathway maps and integrating them into a 2.5D network in Cytoscape"
- [readme] transomics2cytoscape README: package creates 3D network visualization with Cytoscape, Cy3D App, and Cytoscape Automation: "an R package that automatically creates
3D network visualization in combination with
Cytoscape, [Cy3D App](http://apps.cytoscape.org/apps/cy3d), and
[Cytoscape Automation]"
