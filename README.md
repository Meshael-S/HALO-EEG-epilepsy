# HALO-EEG: code for reading the dataset

HALO-EEG (Hierarchically Annotated and Spatially Localized EEG) contains 70 adult scalp EEG recordings from a university hospital in Riyadh, Saudi Arabia, recorded between 2014 and 2024. The recordings were made with a Cadwell Arc system at 250 Hz from 19 scalp electrodes (10–20 system) and total about 25 h. Each recording is provided as a band-pass filtered (0.5–40 Hz) MNE-Python FIF file with expert annotations in four classes: **A** normal, **B** slowing, **C** epileptiform discharges and **D** electrographic seizures. The recordings are also provided as labelled 1 s windows.

This repository contains the code to read and explore the dataset. The data themselves are on Zenodo (see [Citation](#citation)).

## Contents

| File | Purpose |
|---|---|
| `read_dataset.ipynb` | Reads the FIF recordings, the annotation workbook and the 1 s windows, plots examples and creates a split by recording |
| `requirements.txt` | Required Python packages |

Tested with MNE-Python 1.13.2. `requirements.txt`:

```
mne>=1.13
numpy
pandas
openpyxl
matplotlib
scikit-learn
jupyter
```

## Quick start

1. Download the dataset from Zenodo and note the path of the `HALO-EEG` folder.
2. Open `read_dataset.ipynb` and set `ROOT` to that path.
3. Run all cells.

Minimal example:

```python
import mne

raw = mne.io.read_raw_fif("EEG_data/SZ01_filtered_eeg.fif", preload=True)
print(raw.annotations)   # onset (s), duration (s), description (e.g. "C5,C7", "D1")
```

Signals are stored in volts. Annotation onsets follow the MNE convention of the FIF file; the workbook gives them in seconds from the start of each recording.

## Dataset layout

```
HALO-EEG/
├── EEG_data/<RecordingID>_filtered_eeg.fif    continuous recordings with embedded annotations
├── annotations/EEG_annotations.xlsx           annotation workbook
└── windows/
    └── eeg_dataset.npz                        labelled 1 s windows
```

`eeg_dataset.npz` contains:

| Array | Content |
|---|---|
| `X` | windows × 19 channels × 250 samples (float32, volts) |
| `y` | class label (A, B, C or D) |
| `subjects` | recording number (1–70), listed in the `Recordings` sheet of the workbook |
| `win_mode` | `test` (35,528 evaluation windows) or `train` (37,899 windows, including 3,403 augmented class C windows) |
| `ch_names` | channel order of `X` |

## Annotation codes

Each code combines a class letter and a number, for example `C5` or `D1`. In classes B–D, number 1 means generalized, even numbers mean the right hemisphere and odd numbers the left. An event covering several regions carries several comma-separated codes (`C5,C7,C9`). All 26 codes and their electrodes are listed in the `Codes` sheet of the workbook.

## Using the data

- Split training and test data by **recording**, never by window or event: windows from the same recording are strongly correlated.
- Time without an annotation is unlabelled; it is not normal EEG.
- Class frequencies do not reflect clinical prevalence, because the recordings were selected.
- The `train` windows include augmented class C windows that overlap each other. Evaluate on the `test` windows only.

## Citation

If you use HALO-EEG, please cite the dataset and the data descriptor:

- Alharbi N, Aldayel M. HALO-EEG: Hierarchically Annotated and Spatially Localized EEG for Epileptiform Abnormality Detection [dataset]. Zenodo; 2026. https://doi.org/10.5281/zenodo.24048460

Earlier work using part of these recordings:

- Alharbi N, Aldayel M, Alsenan S, Alyami R, Almowalad E, Alkethiry E. From Annotation to Prediction: Hospital-Grade Early Seizure Risk Prediction from Adult EEG. *Diagnostics*. 2026;16(3):492. https://doi.org/10.3390/diagnostics16030492
