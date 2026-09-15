# Measurement-Oriented TinyML for RUL-Derived Turbofan Degradation-State Classification

This repository supports the revised Measurement manuscript **MEAS-D-26-17511** and the associated reproducibility workflow for NASA C-MAPSS FD001–FD004.

> **Scope note:** the repository name `tinyml-sensor-health-deployment` is retained for continuity, but the revised study is **not** framed as direct sensor-health diagnosis. The supervised target is an **RUL-derived turbofan degradation state** (Healthy, Degrading, Critical) obtained from simulated C-MAPSS run-to-failure trajectories.

## Revised scientific scope

The study evaluates a compact 1D CNN/TCN as a test vehicle for a **measurement-oriented validation protocol**, rather than claiming a new learning mechanism. The revised workflow separates:

- leakage-controlled predictive evaluation on unseen engine units;
- engine-cluster bootstrap uncertainty;
- repeated-seed training variability;
- controlled synthetic measurement perturbations;
- matched continued-training controls for magnitude pruning;
- Keras FP32 → TFLite FP32 conversion fidelity;
- full-integer INT8 post-training quantization (PTQ);
- computational and storage accounting;
- explicit separation of host-side evaluation from physical MCU validation.

## Data construction

The executed configuration used in the revised analysis is:

```text
Dataset: NASA C-MAPSS FD001–FD004
Window length: 30 samples
Stride: 5 samples
Effective overlap: 83.33%
Label position: final sample of each window
Healthy: RUL > 120
Degrading: 40 <= RUL <= 120
Critical: RUL < 40
Train unit fraction: 0.68
Validation unit fraction: 0.15
Test: remaining engine units
```

Complete engine trajectories are assigned to mutually exclusive train/validation/test partitions **before** window generation. The scaler is fitted on training-unit data only.

### Dataset-native channels

The frozen revised implementation uses the following C-MAPSS sensor indices:

```text
s20, s8, s9, s1, s2, s3, s4, s5, s7, s11
```

The earlier W31→Wf physical proxy interpretation has been removed.

## Main reproducibility notebook

`Measurement_TinyML_Final_Reproducible.ipynb`

The notebook contains the corrected data construction and the reviewer-requested controls, including:

- repeated independent training seeds;
- matched continued-unpruned controls;
- 30% and 50% global unstructured magnitude sparsification;
- FP32 TFLite conversion control;
- INT8 PTQ evaluation;
- precise robustness-operator definitions;
- FLOPs/MACs and activation-memory accounting;
- compact baseline and architecture-ablation comparisons.

## Key revised results

### Frozen submitted FP32 reference

- Accuracy: **73.59%**
- Macro-F1: **75.22%**
- ROC-AUC: **0.8758**
- Average Precision: **0.9280**
- Parameters: **63,836**

### Ten-seed repeatability

- Accuracy: **75.01% ± 1.60%**
- Macro-F1: **76.97% ± 1.65%**
- ROC-AUC: **0.8825 ± 0.0062**
- AP: **0.9317 ± 0.0031**

### Matched pruning controls

The earlier single-run increase after 30% pruning is **not attributed to pruning** after introducing a matched continued-training control.

- Continued unpruned accuracy: **75.55% ± 0.75%**
- 30% sparsity accuracy: **75.66% ± 0.66%**
- 50% sparsity accuracy: **75.59% ± 0.47%**
- 30% vs continued-unpruned paired accuracy difference: **+0.11 percentage points**
- Paired t-test: **p = 0.291**

Interpretation: 30–50% unstructured magnitude sparsity preserves predictive performance under the tested matched-training protocol; no pruning-specific regularization gain is claimed.

### FP32 TFLite conversion control and INT8 PTQ

Controlled conversion results:

| Runtime/artifact | Accuracy | Macro-F1 | ROC-AUC |
|---|---:|---:|---:|
| Keras FP32 | 73.19% | 74.06% | 0.8910 |
| TFLite FP32 | 73.19% | 74.06% | 0.8910 |
| TFLite INT8 | 48.55% | 41.99% | 0.7663 |

Maximum absolute probability difference between Keras FP32 and TFLite FP32: **3.70 × 10⁻⁶**.

This shows that ordinary FP32 TFLite conversion is numerically faithful, while the tested full-integer INT8 transformation does not preserve diagnostic equivalence.

### Computational accounting

- Approximate FLOPs per window: **2.21 MFLOPs**
- Approximate MACs per window: **1.10 MMACs**
- Approximate peak single-layer FP32 activation: **23.44 kB**
- FP32 TFLite artifact: **256.68 kB**
- INT8 TFLite artifact: **102.69 kB**

No direct MCU latency, peak target-device RAM, electrical power, or energy measurement is claimed.

## Robustness operators

The robustness experiments are **synthetic sensitivity tests**, not physical sensor-fault qualification.

- Gaussian noise: 20 dB SNR computed over the complete standardized window.
- Dropout: 5% independent element-wise mask; dropped standardized entries are set to zero.
- Temporal jitter: whole-window circular shift sampled from {-3, …, +3} samples.
- Selected-channel offset: additive standardized offset applied to dataset-native channels s1–s4.
- Burst corruption: synthetic transient corruption with explicitly defined burst length, amplitude, rate parameter, start-position sampling, and random seed.

These coordinates must not be interpreted as physical °C offsets, measured EMI rates, or calibrated packet-loss/failure probabilities.

## Reviewer-response outputs

The folder `reviewer_response_outputs/` contains the machine-readable outputs used to support the revised manuscript and response to reviewers, including:

- window-construction audit;
- repeated-seed summaries;
- matched pruning statistics;
- FP32/TFLite/INT8 control results;
- TFLite output mapping;
- robustness-operator specification;
- computational accounting;
- baseline and architecture-ablation results.

## Dataset

The NASA C-MAPSS data are **not redistributed** here. Obtain the original FD001–FD004 training files from the NASA Prognostics Center of Excellence data repository.

## Important reporting boundaries

1. C-MAPSS is simulated run-to-failure data, not operational aircraft sensor data.
2. The target is RUL-derived engine degradation state, not an independently labelled sensor-health condition.
3. Pruning is global **unstructured magnitude sparsification**, not structured channel pruning.
4. INT8 performance is measured by executing the exported TFLite artifact on the held-out test set.
5. Host/notebook timing is not MCU timing.
6. No direct electrical power or energy measurement is reported.
7. QAT is not reported as a successful experimental result in the revised study.
8. No multi-node/fleet-level inference result is claimed.

## Citation

If you use this repository, please cite the associated Measurement manuscript once publication details are available.

## License

Code is released under the MIT License. NASA C-MAPSS remains subject to its original source terms and is not redistributed here.
