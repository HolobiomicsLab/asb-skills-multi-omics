---
name: single-cell-training-dynamics-scoring
description: Use when an AnnData object carries an observation-column cell-type or
  condition label and you need per-cell goodness-of-match scores to detect mislabeled
  cells, ambiguous states, or label-difficulty patterns before downstream clustering
  or reannotation.
when_to_use_negative:
- Data are convolved or aggregated spatial transcriptomics measurements — Annotatability
  does not directly handle such data
- Annotations are probabilistic or continuous rather than nominal categories — the
  method currently treats input annotations as nominal categories only
- The goal is standard neighborhood-based clustering via sc.pp.neighbors — this would
  overwrite adata.obsp['connectivities'] rather than store the trainability-aware
  graph
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: Annotatability
  role: Core package providing models.follow_training_dyn_neural_net (DNN training-dynamics
    recording) and models.probability_list_to_confidence_and_var (confidence/variability
    scoring)
  repo: https://github.com/nitzanlab/Annotatability
- name: PyTorch
  role: Backend for training the neural network and tensor handling; device selection
    and optimization (lr=0.001, momentum=0.9)
- name: Scanpy
  role: AnnData handling and preprocessing of the single-cell data on which training
    dynamics are computed; code base of Annotatability
  repo: https://scanpy.readthedocs.io/en/stable/
- name: Annotatability_notebooks
  role: Reproducibility repository with annotated MERFISH and other datasets and tutorial
    notebooks (e.g. tutorial_retina.ipynb, tutorial_covid.ipynb) demonstrating the
    scoring workflow
  repo: https://github.com/nitzanlab/Annotatability_notebooks
merged_aliases:
- single-cell-training-dynamics-analysis
- single-cell-training-dynamics-confidence-scoring
merged_alias_records:
- alias: single-cell-training-dynamics-analysis
  slug: single-cell-training-dynamics-analysis
  jaccard_score: 0.6667
  method: token-set-jaccard
  decision: needs_review
- alias: single-cell-training-dynamics-confidence-scoring
  slug: single-cell-training-dynamics-confidence-scoring
  cosine_score: 0.8686
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_002
  - task_007
  - task_003
  - task_006
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:cf8455a350e8cfce249ec71ae08bcb8c6097bb0c20b46b52d247797b20212b08
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/nitzanlab/Annotatability
  repo_url: https://github.com/nitzanlab/Annotatability
  tools:
  - Annotatability
  - Annotatability_notebooks
  - PyTorch
  - Python
  - Scanpy
  - numpy
  - pandas
  - scReClassify
  - scipy
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-training-dynamics-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores every cell of an annotated AnnData single-cell or spatial omics dataset with 'confidence' (mean probability the DNN assigns to the correct label across epochs) and 'variability' (standard deviation of that probability) computed from per-epoch training dynamics of a PyTorch neural network, using the Annotatability package, to flag erroneously annotated, ambiguous, or biologically intermediate cells.

## When to use

Use when an AnnData object carries an observation-column cell-type or condition label and you need per-cell goodness-of-match scores to detect mislabeled cells, ambiguous states, or label-difficulty patterns before downstream clustering or reannotation.

## When NOT to use

- Data are convolved or aggregated spatial transcriptomics measurements — Annotatability does not directly handle such data
- Annotations are probabilistic or continuous rather than nominal categories — the method currently treats input annotations as nominal categories only
- The goal is standard neighborhood-based clustering via sc.pp.neighbors — this would overwrite adata.obsp['connectivities'] rather than store the trainability-aware graph

## Inputs

- Annotated AnnData object (single-cell RNA-seq or spatial omics, e.g. MERFISH) with cell labels in an observation column such as adata.obs['label']
- Label key string identifying the annotation to audit
- Device specification ('cuda' if GPU available, else 'cpu')

## Outputs

- adata.obs['conf']: per-cell confidence (average probability assigned to the correct label across epochs)
- adata.obs['var']: per-cell variability (standard deviation of that probability across epochs)
- Per-epoch probability list (prob_list) from follow_training_dyn_neural_net
- Cell categorization into easy-to-learn (correctly annotated), hard-to-learn (erroneously annotated), and ambiguous classes

## How to apply

Load the annotated AnnData (label stored as an observation column, e.g. 'label') and preprocess per the paper's filters (e.g. keep genes expressed in >3 cells, use ~3,000 highly variable genes; scale each gene to variance 1 where required). Train the DNN while recording per-epoch predictions with models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=epoch_num, device=device) using defaults lr=0.001, momentum=0.9, batch_size=256, weighted_sampler=True, num_layers=3, on GPU (cuda) if available; epoch_num is commonly 50–100. Convert the returned probability list into scores via models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=epoch_num) and store them as adata.obs['conf'] and adata.obs['var']. Interpret scores as: high confidence + low variability = easy-to-learn (probably correctly labeled); low confidence + low variability = probably mislabeled; moderate confidence + high variability = ambiguous/intermediate states. Optionally classify cells by thresholding (the hard-to-learn percentile q is dataset-dependent, 25%–90%) and, if desired, feed the scores into the reannotation module or the trainability-aware graph (metrics.make_conf_graph).

## Related tools

- **Annotatability** (Core package providing models.follow_training_dyn_neural_net (DNN training-dynamics recording) and models.probability_list_to_confidence_and_var (confidence/variability scoring)) — https://github.com/nitzanlab/Annotatability
- **PyTorch** (Backend for training the neural network and tensor handling; device selection and optimization (lr=0.001, momentum=0.9))
- **Scanpy** (AnnData handling and preprocessing of the single-cell data on which training dynamics are computed; code base of Annotatability) — https://scanpy.readthedocs.io/en/stable/
- **Annotatability_notebooks** (Reproducibility repository with annotated MERFISH and other datasets and tutorial notebooks (e.g. tutorial_retina.ipynb, tutorial_covid.ipynb) demonstrating the scoring workflow) — https://github.com/nitzanlab/Annotatability_notebooks

## Examples

```
from Annotatability import models; epoch_num=50; prob_list = models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=epoch_num, device=device); all_conf, all_var = models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=epoch_num); adata.obs['conf']=all_conf.detach().numpy(); adata.obs['var']=all_var.detach().numpy()
```

## Evaluation signals

- adata.obs['conf'] and adata.obs['var'] exist for all n_obs cells with finite numeric values after probability_list_to_confidence_and_var
- Cells with high confidence and low variability correspond to the correctly annotated fraction; low-confidence/low-variability cells enriched for known mislabels
- AUROC of mislabel detection against a correct/incorrect ground-truth flag increases as the synthetic mislabeling fraction grows (per the MERFISH perturbation benchmark)
- Confidence/variability distributions separate annotation classes, e.g. larger spread and fewer easy-to-learn cells when mSTZ data are annotated by condition/treatment than by cell type
- Downstream reannotation restores marker-gene agreement, e.g. Gad1 mean normalized expression rises from 1.458 (erroneous) to 6.795 after reannotation vs 6.830 for correctly annotated inhibitory neurons

## Limitations

- Hyperparameters such as training time (iterNum) are not standardized and influence the confidence and variability scores
- Thresholding on confidence/variability for categorization introduces arbitrary cutoffs; the hard-to-learn percentile q is chosen dataset-dependently between 25% and 90%
- Annotatability treats input annotations as nominal categories only; probabilistic and continuous annotations are deferred to future work
- Annotatability does not directly handle convolved or aggregated data, such as that obtained from certain spatial transcriptomics technologies
- No PyTorch software version is reported, limiting reproducibility of the training procedure

## Evidence

- [discussion] Confidence definition: [evidence span withheld — anchor status: not_found]
- [discussion] Dataset-dependent thresholding caveat: [evidence span withheld — anchor status: not_found]
