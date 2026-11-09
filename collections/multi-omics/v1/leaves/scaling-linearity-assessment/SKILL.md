---
name: scaling-linearity-assessment
description: Use when evaluating whether automated network rendering (e.g. transomics2cytoscape
  stacking n pathway layers in Cytoscape) remains feasible as the number of layers
  grows, by timing runs across a layer-count series.
when_to_use_negative:
- The goal is 2.5D network visualization quality itself rather than performance/scalability
  characterization of the rendering step
- The pathway data are not in the supported KEGG ID / KGML pathway file format (Reactome,
  Wikipathways, SBGN are not supported)
- The trans-omic connectivity data or inter-layer edges are not yet reconstructed,
  since transomics2cytoscape has not automated retrieval of inter-layer edges from
  databases such as BRENDA, OmniPath, SIGNOR, or IntAct
edam_topics:
- http://edamontology.org/topic_2259
tools:
- name: transomics2cytoscape
  role: R Bioconductor package whose Case Study I workflow is run repeatedly (n =
    2–6 layers) to time 2.5D network stacking
  repo: https://github.com/ecell/transomics2cytoscape
- name: Cytoscape
  role: Network rendering engine (3.8.0 per methods, 3.10.3 tested) whose drawing
    time is measured, run headless under xvfb
  repo: https://github.com/cytoscape/cytoscape
- name: RCy3
  role: Cytoscape Automation bridge (v2.24.0) used by transomics2cytoscape to drive
    network creation and rendering from R
- name: R
  role: Environment for running transomics2cytoscape and recording layer-count vs
    drawing-time measurements
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:0478244d5fde6b4567a0bb199b69547cfaed6f46dcac5bccac330b7a95f8daaf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ecell/transomics2cytoscape
  tools:
  - Cytoscape
  - R
  - RCy3
  - transomics2cytoscape
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# scaling-linearity-assessment

## Summary

Benchmark how the wall-clock time of transomics2cytoscape rendering scales as the number of stacked pathway layers increases from two to six in Cytoscape, to demonstrate that drawing time grows approximately linearly and stays within the paper's few-minutes regime.

## When to use

Use when evaluating whether automated network rendering (e.g. transomics2cytoscape stacking n pathway layers in Cytoscape) remains feasible as the number of layers grows, by timing runs across a layer-count series.

## When NOT to use

- The goal is 2.5D network visualization quality itself rather than performance/scalability characterization of the rendering step
- The pathway data are not in the supported KEGG ID / KGML pathway file format (Reactome, Wikipathways, SBGN are not supported)
- The trans-omic connectivity data or inter-layer edges are not yet reconstructed, since transomics2cytoscape has not automated retrieval of inter-layer edges from databases such as BRENDA, OmniPath, SIGNOR, or IntAct

## Inputs

- Case Study I input data (KEGG KGML pathway maps and layer tables) from the ecell/transomics2cytoscape GitHub repository
- Installed Cytoscape 3.8.0/3.10.3 with Cy3D 1.1.3 and KEGGscape 0.9.2 apps
- transomics2cytoscape R Bioconductor package with RCy3 2.24.0

## Outputs

- drawing_time_table.csv mapping layer count (n = 2–6) to wall-clock drawing time
- Per-run timing records demonstrating approximately linear scaling of rendering time

## How to apply

Set up the paper's headless Linux environment: install OpenJDK 11, wget, and xvfb, then run Cytoscape 3.8.0 (or the tested 3.10.3) under xvfb with the Cy3D 1.1.3 and KEGGscape 0.9.2 apps and the transomics2cytoscape R package (RCy3 2.24.0) loaded. For each stack count n = 2, 3, 4, 5, 6, run the Case Study I workflow from the ecell/transomics2cytoscape GitHub repository to stack n pathway layers along the Z axis and record the wall-clock drawing time per run. Write the layer-count vs drawing-time pairs to drawing_time_table.csv. Validate that all five stack counts are present, that time rises approximately linearly with n, and that each rendering stays within the paper's reported few-minutes visualization regime — confirming rendering does not limit realistic use.

## Related tools

- **transomics2cytoscape** (R Bioconductor package whose Case Study I workflow is run repeatedly (n = 2–6 layers) to time 2.5D network stacking) — https://github.com/ecell/transomics2cytoscape
- **Cytoscape** (Network rendering engine (3.8.0 per methods, 3.10.3 tested) whose drawing time is measured, run headless under xvfb) — https://github.com/cytoscape/cytoscape
- **RCy3** (Cytoscape Automation bridge (v2.24.0) used by transomics2cytoscape to drive network creation and rendering from R)
- **R** (Environment for running transomics2cytoscape and recording layer-count vs drawing-time measurements)

## Examples

```
sudo apt-get install -y openjdk-11-jdk wget xvfb && xvfb-run -a ./cytoscape-unix-3.8.0/cytoscape.sh
```

## Evaluation signals

- drawing_time_table.csv exists and contains rows for all five stack counts (n = 2–6)
- Drawing time increases approximately linearly with the number of stacked networks
- Each n-layer rendering completes within the paper's reported few-minutes visualization regime
- Each run produces a 2.5D network in Cytoscape with n pathway layers stacked along the Z axis and inter-layer edges
- Timing runs execute headlessly under xvfb on Linux with OpenJDK 11 and Cy3D 1.1.3 / KEGGscape 0.9.2 apps installed

## Limitations

- Estimation of Z-coordinates in Case Study I and II is not automated by transomics2cytoscape
- Only the KEGG ID and pathway file format are supported; Reactome, Wikipathways, and SBGN are not supported yet
- Connectivity data of trans-omic networks and retrieval of inter-layer edges (BRENDA, OmniPath, SIGNOR, IntAct) are not fully automated, so timing depends on manually prepared inputs

## Evidence

- [abstract] rapid few-minutes visualization regime: "We confirmed that transomics2cytoscape could be used for rapid visualization of trans-omics networks presented in published papers within a few minutes"
- [discussion] 2.5D integration in Cytoscape: [evidence span withheld — anchor status: not_found]
- [methods] headless Cytoscape environment setup: "Run `sudo apt-get install -y openjdk-11-jdk wget xvfb` in the terminal."
- [methods] Cytoscape version under test: "wget https://github.com/cytoscape/cytoscape/releases/download/3.8.0/cytoscape-unix-3.8.0.tar.gz"
- [discussion] Case Study I input data location: "The data used in this study are available in our GitHub repository https://github.com/ecell/transomics2cytoscape."
