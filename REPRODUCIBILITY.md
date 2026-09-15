# Reproducibility Notes — Revised Measurement Analysis

## Scope

This repository reproduces the revised experiments for manuscript **MEAS-D-26-17511** using NASA C-MAPSS FD001–FD004 simulated run-to-failure trajectories.

## Audited preprocessing configuration

- Random seed for the frozen split: 42
- Window length: 30 samples
- **Stride: 5 samples**
- Effective overlap: **83.33%**
- Label position: end of window
- Healthy: RUL > 120
- Degrading: 40 <= RUL <= 120
- Critical: RUL < 40
- Train unit fraction: 0.68
- Validation unit fraction: 0.15
- Test: remaining units
- StandardScaler fitted on training data only

The earlier documentation of 80% overlap / stride 1 was inconsistent with the executed pipeline and has been corrected.

## Input channels

The revised pipeline reports dataset-native C-MAPSS indices:

```text
s20, s8, s9, s1, s2, s3, s4, s5, s7, s11
```

The unsupported W31→Wf proxy interpretation has been removed.

## Evaluation layers

The revised workflow intentionally separates several evidence classes:

1. **Frozen held-out performance** on unseen engine units.
2. **Engine-unit cluster bootstrap** uncertainty.
3. **Ten-seed training repeatability**.
4. **Matched continued-training versus pruning** comparisons.
5. **FP32 TFLite conversion fidelity**.
6. **INT8 PTQ artifact execution**.
7. **Synthetic robustness sensitivity**.
8. **Computational/storage accounting**.

## Pruning

Pruning is global unstructured magnitude sparsification of kernel tensors. Target sparsities are 30% and 50%.

Each pruning run is compared against a matched unpruned model receiving the same additional training opportunity. The revised interpretation is performance preservation under sparsity, not a pruning-induced regularization gain.

## Quantization

The conversion sequence is evaluated as:

```text
Keras FP32 -> TFLite FP32 -> TFLite INT8
```

The FP32 TFLite control confirms that ordinary TFLite conversion/output mapping is numerically faithful. The full-integer INT8 path is then evaluated independently.

Representative calibration, tensor scales/zero-points, input quantization, output dequantization, and the three-class output mapping are recorded in the reviewer-response outputs.

## Robustness operators

All robustness experiments are synthetic sensitivity operators.

- **20 dB Gaussian noise:** signal power is computed over the complete standardized window; IID Gaussian noise is added using the documented seed.
- **5% dropout:** independent element-wise time × channel mask; dropped standardized values are set to zero.
- **Temporal jitter:** common circular shift of the complete window by an integer sampled from -3 to +3 samples.
- **Selected-channel offset:** standardized additive offset on channels s1–s4; the coordinate is in training-SD units, not °C.
- **Burst corruption:** artificial transient corruption using the documented burst length, amplitude, rate coordinate, start-position sampling, and seed.

These experiments do not constitute physical environmental qualification.

## Computational accounting

The revised analysis reports parameters, approximate FLOPs/MACs, activation footprint, and TFLite artifact sizes. It does **not** claim MCU latency, MCU RAM, measured power, or measured energy.

## QAT

Quantization-aware training is not reported as a successful result because the available TensorFlow Model Optimization / Keras configuration did not provide a reliable compatible path for this model in the executed environment.

## Recommended archival practice

Archive the exact notebook and generated machine-readable outputs used for the manuscript revision. A DOI-backed release (for example through Zenodo) is recommended after acceptance or public release.
