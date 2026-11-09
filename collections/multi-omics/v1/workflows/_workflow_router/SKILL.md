---
name: multi-omics-workflow-router
description: Use when a user has a whole multi-omics analysis goal rather than a single step — select the appropriate end-to-end composite workflow, run its stages, and ground them against their source papers.
license: CC-BY-4.0
metadata:
  kind: workflow-router
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/multi-omics/v1
  indexes: [workflows_index.json]
  bound_by: perspicacite-semantic
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# ASB Multi-Omics — composite workflow router

This is the goal-level entry point for the ASB Multi-Omics collection. The leaf router (`multi-omics-router`) finds one atomic skill; this router selects an end-to-end composite workflow—an ordered pipeline of stages that delegate to vetted leaf skills.

## 1. Select — match the goal to a workflow

Run the collection's retrieval command:

```bash
python bin/search_skills.py --query "<the user's goal>" \
  --collection . --target workflows --k 3
```

For manual lookup, load `workflows_index.json`, which has one row per composite workflow, and match the user's goal against each row's description.

Available workflows (published as outlines):

| workflow | what it does |
|---|---|
| `single-cell-transcriptome-proteome-integration` | Normalize paired RNA and protein counts, construct pruned modality graphs, train a joint representation, and assess clustering across resolutions. |
| `spatial-transcriptomics-niche-analysis` | Load spatial images and transcripts, resolve segmentation across patches, aggregate cell measurements, derive image-based niches and export a spatial analysis store with geometry tables and visualization files. |

If no workflow fits the goal, fall back to the leaf router (`multi-omics-router`) and assemble steps from atomic skills.

## 2. Run — execute the workflow's stages

Read the chosen `workflows/<slug>/SKILL.md` and follow its **Stages** in order. Each stage carries a goal, candidate leaf skills with the primary first, the tools to install or invoke, and its typed inputs and outputs. The machine-readable `workflows/<slug>/workflow.yaml` is the DAG (`after`, `inputs_from`). Automatic grading of that DAG (`asb solve-workflow`) is not part of this release: no released ASB version loads these files, so run the stages yourself. Honor the I/O contract: each stage consumes the prior stage's declared outputs. Optional stages are marked.

For a stage's leaf skills, read each `leaves/<leaf-slug>/SKILL.md` for the procedure, or use the leaf router to pick among the candidates for the exact data.

## 3. Ground — verify each stage against its source papers

Before trusting a parameter or default, ground the stage's leaves against the papers they were distilled from. Each stage's `grounding.kb_slugs` and `dois` in `workflow.yaml` name the source papers and the `asb-paper-<doi>` knowledge bases.

> These workflows are published as **outlines**: the stage structure is validated, the execution is not. Bindings were chosen by semantic retrieval plus deterministic selection. `derived_from_workflows` is a provenance record; no ablation experiment consuming it is released.
