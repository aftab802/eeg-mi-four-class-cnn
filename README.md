# Four-Class Motor Imagery Classification Using a Residual Convolutional Neural Network

[![tests](https://github.com/aftab802/eeg-mi-four-class-cnn/actions/workflows/tests.yml/badge.svg)](https://github.com/aftab802/eeg-mi-four-class-cnn/actions/workflows/tests.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Six-block CNN with concatenation skips for four-class EEG (left fist, right fist, both feet, rest) on PhysioNet EEGMMIDB.

Code accompanying the manuscript **“Four-Class Motor Imagery Classification Using a Residual Convolutional Neural Network”** by Muhammad Aftab ur Rehman, Muhammad Awais ur Rehman, Muhammad Mohsin Sajjad (submitted, 2026).
Corresponding author: Muhammad Aftab ur Rehman, CYBEX – School of IT Professionals (Pvt) Limited, Faisalabad, Pakistan.

## Installation

```bash
git clone https://github.com/aftab802/eeg-mi-four-class-cnn.git
cd eeg-mi-four-class-cnn
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```
Python 3.10–3.13 (tested on 3.13 with TensorFlow 2.20). A GPU is optional but recommended for the deep-learning experiments.

## Data

PhysioNet EEG Motor Movement/Imagery Database (EEGMMIDB), 109 subjects, 64 channels, 160 Hz —
https://physionet.org/content/eegmmidb/1.0.0/ (DOI [10.13026/C28G6P](https://doi.org/10.13026/C28G6P)).
Nothing needs to be downloaded by hand: the loaders fetch the EDF files through `mne.datasets.eegbci`
on first use and cache them under `~/mne_data` (about 3.4 GB for all subjects; pass `--data-path` to change it).

## Reproducing the paper

```bash
python scripts/run_fourclass_cnn.py --epochs 50     # 50-epoch configuration
python scripts/run_fourclass_cnn.py --epochs 100    # 100-epoch configuration
python scripts/run_fourclass_cnn.py --protocol L0   # subject-wise split (Algorithm 2)
```

Every run writes a JSON file to `results/` with the test metrics, confusion matrix, split sizes and, for subject
data, `train_test_subject_overlap` (0 under the leakage-safe protocol).

## Reproduction status

| Result in the paper | Value | Source of the number | Re-run with this repository? |
|---|---|---|---|
| Test accuracy, 100 epochs | 66 % | Original notebook, output preserved (66.48 %, train 84.5 %) | TF2 port in `run_fourclass_cnn.py`; not re-run here |
| Test accuracy, 50 epochs | 63 % | Not in the preserved notebook output | `--epochs 50`; **re-run to document** |

## Points to resolve against the manuscript

* **Class set.** The extraction keeps T2 (both feet) from runs R06/R10/R14 and drops T1 (both fists); the fourth
  class is **rest**, filled with random 4-second windows from the R01 eyes-open baseline. The classes are
  therefore left fist, right fist, both feet and rest (see `cybex_eeg/physionet.py` and
  `tests/test_physionet_fourclass.py`).
* The original batch normalisation always uses the statistics of the current batch, including at test time
  (the whole test set is one batch). The port keeps this behaviour (`BatchStatNorm`).
* The model has about 15.0 M parameters, 14.7 M of them in the 28,800 × 512 dense layer.

## Repository layout

```
cybex_eeg/        shared, leakage-aware library (splits, normalisation, features, models)
scripts/          one command per experiment in the paper; results are written to results/*.json
tests/            pytest suite run on synthetic data (no download needed)
original_notebooks/  the original Colab notebooks, with their printed outputs, for provenance
```

## Tests

```bash
pytest -q
```
Tests check the split/leakage guards, parameter counts against the paper, the PhysioNet four-class extraction on synthetic EDF files, and end-to-end runs of every model on synthetic EEG.

## Citation

See [`CITATION.cff`](CITATION.cff). Until the article is published, please cite the manuscript title above and
this repository.

## Related repositories

* [eeg-mi-leakage-benchmark](https://github.com/aftab802/eeg-mi-leakage-benchmark) — A Systematic Benchmark of Motor Imagery Classifiers Under Leakage-Safe Evaluation
* [eeg-dementia-integrity-benchmark](https://github.com/aftab802/eeg-dementia-integrity-benchmark) — Data Integrity and Feature Engineering in Small-Cohort EEG Dementia Classification
* [eeg-mi-protocol-leakage-case-study](https://github.com/aftab802/eeg-mi-protocol-leakage-case-study) — A Case Study of Evaluation-Protocol Leakage in Motor Imagery EEG Classification
* [eeg-mi-fbcsp-cnn](https://github.com/aftab802/eeg-mi-fbcsp-cnn) — CSP+CNN: Filter-Bank Common Spatial Patterns with a Convolutional Classifier for Motor Imagery Classification from EEG
* [group-aware-eegnet](https://github.com/aftab802/group-aware-eegnet) — Group-Aware EEGNet: Subject-Independent Motor Imagery Classification from EEG
* [eeg-dementia-label-correction](https://github.com/aftab802/eeg-dementia-label-correction) — Label Correction and Spectral Features for Multiclass Dementia Detection from EEG
* [lead-eeg](https://github.com/aftab802/lead-eeg) — LEAD-EEG: A Lightweight Ensemble Framework for Alzheimer's Detection from EEG

## License

MIT — see [LICENSE](LICENSE). The datasets are distributed by their owners under their own terms.
