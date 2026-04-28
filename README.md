# Breast Cancer Imagine - CNN Baseline and Dataset Quality Investigation

A PyTorch project that began as a CNN classifier on a public breast cancer imaging dataset and evolved into an investigation of label quality issues in the dataset. The repository documents both the engineering pipeline (modular, Dockerized environment, reproducible training) and the finding on why off-the-shelf public medical imaging datasets often cap achievable model reliability. 

This project was built as part of self-directed specialisation in clinical data and MedTech, where data quality matters more than headline metrics.

---
## TL;DR

- **What this is:** A reproducible PyTorch + Docker pipeline for training a simple CNN on the Kaggle dataset [`hayder17/breast-cancer-detection`](https://www.kaggle.com/datasets/hayder17/breast-cancer-detection).
- **What I found:** During training I observed inconsistent and likely mislabelled samples in the dataset, which set a hard ceiling on achievable model performance regardless of architecture or training choices.
- **Why I'm publishing it that way:** In clinical data and MedTech contexts, recognising that a dataset is unreliable is more important than producing a polished accuracy number on top of unreliable ground truth.

--- 

## Why this project exists

Coming from a Master's in Biomedical Engineering and 2.5 years as a Data Analyst (Python, SQL), I wanted hands-on experience with the full lifecycle of a medical-imaging machine-learning pipeline: dataset acquisition, preprocessing, training, evaluation, model serialisation, inference, and containerisation. I picked breast cancer imaging because it sits at a realistic intersection of clinical data and Software-as-a-Medical-Device (SaMD) workflows.

The honest experience of doing this end-to-end, including discovering that the public dataset I'd chosen was not accurate enough to draw any clinical conclusions from — turned out to be the most valuable part of the project.

---

## Findings on dataset quality

While iterating on the model and reviewing predictions, I observed:

- **Inconsistent labelling between visually similar images.** Pairs of images with comparable visual features were assigned different class labels, and pairs of images with clearly different presentations occasionally shared a label.
- **No documented annotation protocol.** The published dataset does not ship with information about who annotated the images, how disagreements were resolved, or what the inter-rater reliability is. For a medical dataset, the absence of this information is itself a finding.
- **Performance plateau independent of model choices.** Beyond a certain point, training accuracy improved on the training set but validation performance stopped tracking it — a pattern consistent with a model fitting label noise rather than a useful signal. Architectural changes, augmentation, learning-rate tuning, and longer training did not move the validation ceiling meaningfully.

The conclusion I reached is that, on this dataset, **the achievable classification reliability is bounded by ground-truth quality, not by model design**. Reporting a "best" accuracy number on top of unreliable labels would be misleading

In a real clinical or SaMD context this kind of investigation would normally trigger a data-quality review, a relabelling effort by a qualified clinician, or sourcing of an alternative dataset with documented annotation provenance, all of which sit outside the scope of an individual learning project, but which are the right next steps.

---

## What this repository does demonstrate

Independently of the modelling outcome, the codebase demonstrates a reproducible, modular ML engineering workflow:

- **Reproducible environment.** A `Dockerfile` and pinned `requirements.txt` ensure the project runs identically on any host. A pre-built image is published on Docker Hub.
- **Separation of concerns.** Each step of the pipeline is its own module inside `src/`: data loading, transforms, model, loss, optimizer, training loop, threshold selection, evaluation, model save/load, inference. There is no notebook-only logic.
- **Notebook used for orchestration, not for hidden state.** `train.ipynb` imports from `src/` and runs the same code that would run inside Docker, it is for inspection, not the source of truth.
- **Inference path.** A separate inference module loads a saved model and applies it to new images, matching the structure expected of a deployable workflow.

This is the engineering hygiene I would bring to a clinical data, SaMD, or MedTech analytics role, independent of the modelling result on this particular dataset.



---

## Repository structure

```
.
├── Dockerfile                     # Reproducible runtime
├── .dockerignore
├── requirements.txt               # Pinned dependencies
├── dataset.py                     # Kaggle dataset download
├── train.ipynb                    # Orchestration notebook
└── src/
    ├── __init__.py
    ├── config.py                  # Centralised paths and hyperparameters
    ├── create_annotation_file.py
    ├── create_dataset.py
    ├── dataloader.py              # PyTorch DataLoader wrappers
    ├── transforms.py              # Image transforms / augmentations
    ├── display_random_images.py
    ├── simple_cnn_model.py        # Baseline CNN architecture
    ├── loss_function.py
    ├── optimizer.py
    ├── train_model.py             # Training loop with validation
    ├── threshold.py               # Decision-threshold selection
    ├── evaluate.py                # Metrics and evaluation
    ├── save_model.py
    ├── load_model.py
    └── inference.py               # Inference on new images
```

---

## Getting started

### Option 1 — Docker (recommended)

```bash
docker pull bhanarkarjetal/breast-cancer-detection:v1
docker run -it --rm bhanarkarjetal/breast-cancer-detection:v1
```

### Option 2 — Clone and run locally

```bash
git clone https://github.com/bhanarkarjetal/breast_cancer_detection.git
cd breast_cancer_detection

# Create a virtual environment
python3 -m venv .venv
source .venv/bin/activate          # Linux / macOS
# .venv\Scripts\activate           # Windows

pip install -r requirements.txt

jupyter notebook train.ipynb
```

---

### Pipeline order

1. **Download dataset** — `python dataset.py` pulls `hayder17/breast-cancer-detection` via `kagglehub`.
2. **Build annotations and dataset** — `src/create_annotation_file.py` then `src/create_dataset.py`.
3. **Train** — `src/train_model.py` (with loss and optimizer from `src/loss_function.py` and `src/optimizer.py`).
4. **Evaluate** — `src/evaluate.py` and `src/threshold.py` for metrics and threshold selection.
5. **Save / Load / Infer** — `src/save_model.py`, `src/load_model.py`, `src/inference.py`.

---

## What I would do differently

Treating this as a learning artefact rather than a finished product, the honest list of improvements is:

- **Validate the dataset before committing to it.** Before any modelling, spend time with a clinician or with the dataset documentation understanding the annotation process; small samples should be cross-checked against published clinical references.
- **Use a dataset with documented provenance.** Public radiology benchmarks such as CBIS-DDSM or RSNA's Breast Cancer Detection challenge data ship with annotation documentation and would be more appropriate for any serious modelling.
- **Make ground-truth uncertainty a first-class output.** Rather than only reporting accuracy, report calibration, per-class confusion patterns, and identify the subset of samples on which the model is consistently uncertain, those are the candidates for re-annotation.
- **Decouple the engineering pipeline from the dataset.** The current code is already mostly dataset-agnostic; making this explicit and adding tests would make it straightforward to re-run with a higher-quality dataset.

---

## Context

Built as part of a self-directed specialisation in clinical data, EU MDR and ISO 13485 compliance, and medical imaging workflows. The intention was to learn the full pipeline end-to-end, including the parts that most tutorials skip, like noticing when your dataset is the problem.

---

## Licence

MIT.
