---
name: reproducible-r-vignette-execution
description: Use when a paper claims a vignette or scripted workflow reproduces a
  published result (e.g., a trans-omic network figure) and the goal is to rerun that
  code verbatim on the deposited data and confirm the output.
when_to_use_negative:
- The target workflow requires pathways from non-KEGG databases (Reactome, Wikipathways)
  or formats other than KEGG KGML, since transomics2cytoscape currently supports only
  KEGG IDs and pathway file format
- The vignette depends on steps the authors explicitly did not automate, such as Z-coordinate
  estimation in Case Study I and II, so a fully hands-off reproduction is not possible
- The paper provides no deposited data, no vignette code, and no reference figure
  to compare against, so the reproduction cannot be judged
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0605
tools:
- name: transomics2cytoscape
  role: R Bioconductor package whose usecase1 vignette is executed to regenerate the
    2.5D trans-omic network
  repo: https://github.com/ecell/transomics2cytoscape
- name: Cytoscape
  role: Desktop application in which the 2.5D network is rendered; must be running
    with KEGGscape and Cy3D apps before the vignette executes
  repo: https://github.com/cytoscape/cytoscape
- name: RCy3
  role: R package used by transomics2cytoscape to automate importing pathway maps
    and building the network in Cytoscape via Cytoscape Automation
  repo: https://github.com/ecell/transomics2cytoscape
- name: KEGGREST
  role: Retrieves KGML (KEGG pathway XML) for the pathway IDs used in the vignette
- name: Cy3D
  role: Cytoscape app required to render the 3D/2.5D stacked-layer network
- name: KEGGscape
  role: Cytoscape app required to import KEGG pathway maps as network layers
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:0e3375854c2a2a72829e41023a35fb61596cc98f7393e7c3ba0bc7303c4fb408
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ecell/transomics2cytoscape
  tools:
  - Cy3D
  - Cytoscape
  - KEGGREST
  - KEGGscape
  - R
  - RCy3
  - transomics2cytoscape
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# reproducible-r-vignette-execution

## Summary

Executes a published R/Bioconductor vignette end-to-end against deposited datasets to verify that the documented figure, network, or result regenerates automatically, recording runtime and comparing output against the paper's reference figure.

## When to use

Use when a paper claims a vignette or scripted workflow reproduces a published result (e.g., a trans-omic network figure) and the goal is to rerun that code verbatim on the deposited data and confirm the output.

## When NOT to use

- The target workflow requires pathways from non-KEGG databases (Reactome, Wikipathways) or formats other than KEGG KGML, since transomics2cytoscape currently supports only KEGG IDs and pathway file format
- The vignette depends on steps the authors explicitly did not automate, such as Z-coordinate estimation in Case Study I and II, so a fully hands-off reproduction is not possible
- The paper provides no deposited data, no vignette code, and no reference figure to compare against, so the reproduction cannot be judged

## Inputs

- Published vignette R code (e.g., transomics2cytoscape usecase1 vignette)
- Deposited dataset/zenodo archive DOI 10.5281/zenodo.8201898 with trans-omics connectivity tables
- KEGG pathway IDs in KGML format retrieved via KEGGREST (rno04910, rno01100)

## Outputs

- Cytoscape session file (usecase1_2.5d_network.cys) containing the regenerated 2.5D trans-omic network
- Runtime record (runtime.txt) with elapsed execution time
- Comparison against the reference figure (Fig. 4 three-layer network: rno04910 over two rno01100 layers)

## How to apply

Set up the runtime environment exactly as the vignette specifies: for the transomics2cytoscape usecase1 vignette this means OpenJDK 11, Cytoscape 3.8.0 (directory added to PATH, validated against Cytoscape 3.10.3) with KEGGscape 0.9.2 and Cy3D 1.1.3 apps installed, and R 4.4.1 / Bioconductor 3.19 with RCy3 2.24.0 connecting to the running Cytoscape instance. Execute the vignette code unmodified: it uses KEGGREST to fetch KGML for KEGG pathways (rno04910 and rno01100) and loads connectivity tables from the Zenodo deposit 10.5281/zenodo.8201898, then calls transomics2cytoscape to stack the two rno01100 layers beneath the rno04910 layer with inter-layer edges forming the 2.5D network. Save the resulting session (e.g., usecase1_2.5d_network.cys) and record elapsed runtime to runtime.txt. Judge success by whether the regenerated network reproduces the paper's reference configuration (here, the Fig. 4 three-layer layout) and whether the run completes within the minutes-scale time the authors reported (3 minutes on a 12th Gen Intel Core i7-12700 Windows 11 PC with 128 GB RAM).

## Related tools

- **transomics2cytoscape** (R Bioconductor package whose usecase1 vignette is executed to regenerate the 2.5D trans-omic network) — https://github.com/ecell/transomics2cytoscape
- **Cytoscape** (Desktop application in which the 2.5D network is rendered; must be running with KEGGscape and Cy3D apps before the vignette executes) — https://github.com/cytoscape/cytoscape
- **RCy3** (R package used by transomics2cytoscape to automate importing pathway maps and building the network in Cytoscape via Cytoscape Automation) — https://github.com/ecell/transomics2cytoscape
- **KEGGREST** (Retrieves KGML (KEGG pathway XML) for the pathway IDs used in the vignette)
- **Cy3D** (Cytoscape app required to render the 3D/2.5D stacked-layer network)
- **KEGGscape** (Cytoscape app required to import KEGG pathway maps as network layers)

## Examples

```
Rscript -e 'library(transomics2cytoscape); usecase1()'
```

## Evaluation signals

- The regenerated Cytoscape network reproduces the reference figure's configuration (Fig. 4: rno04910 layer stacked over two rno01100 layers with inter-layer regulatory edges)
- The vignette runs to completion without errors in a few minutes (paper reports 3 minutes on a 12th Gen Intel Core i7-12700 Windows 11 PC with 128 GB RAM and RTX 4070)
- The saved session file (usecase1_2.5d_network.cys) exists and contains the expected 2.5D network when reopened in Cytoscape
- Runtime recorded in runtime.txt matches the minutes-scale expectation
- Required apps (KEGGscape 0.9.2, Cy3D 1.1.3) and RCy3 2.24.0 versions match the validated environment before execution

## Limitations

- Z-coordinate estimation in Case Study I and II is not automated, so some manual setup may precede the vignette
- Connectivity data of trans-omic networks are provided after reconstruction procedures that are not fully automated
- Retrieval of inter-layer edges from databases (BRENDA, OmniPath, SIGNOR, IntAct) is not automated; only the deposited connectivity tables are used
- Only KEGG ID and pathway file format are supported; Reactome, Wikipathways, and SBGN are not
- Vignette was validated against Cytoscape 3.10.3 with RCy3 2.24.0 on R 4.4.1 / Bioconductor 3.19; other version combinations are untested

## Evidence

- [other] Running the usecase1 vignette code reproduces the three-layered trans-omic network of insulin action in rat hepatoma Fao cells automatically: [evidence span withheld — anchor status: not_found]
- [abstract] We confirmed that transomics2cytoscape could be used for rapid visualization of trans-omics networks presented in published papers within a few minutes: "We confirmed that transomics2cytoscape could be used for rapid visualization of trans-omics networks presented in published papers within a few minutes"
- [discussion] Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML).: "Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML)."
- [methods] wget https://github.com/cytoscape/cytoscape/releases/download/3.8.0/cytoscape-unix-3.8.0.tar.gz: "wget https://github.com/cytoscape/cytoscape/releases/download/3.8.0/cytoscape-unix-3.8.0.tar.gz"
- [other] the vignette was validated against Cytoscape 3.10.3: [evidence span withheld — anchor status: not_found]
