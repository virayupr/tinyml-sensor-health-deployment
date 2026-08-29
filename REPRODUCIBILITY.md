# Reproducibility Notes

## Experimental scope

The reproducibility workflow is designed around NASA C-MAPSS FD001–FD004 simulated run-to-failure trajectories. Engine units are split before window generation so that overlapping windows from one engine cannot occur across development and test partitions.

## Fixed configuration

- Random seed: 42
- Window length: 30 samples
- Window overlap: 80%
- Healthy: RUL > 120
- Degrading: 40 <= RUL <= 120
- Critical: RUL < 40
- Train unit fraction: 0.68
- Validation unit fraction: 0.15
- Test: remaining units
- StandardScaler is fitted on training data only.

## Input channels

The executed pipeline uses Wf, Nf, Nc, T2, T24, T30, T50, P2, P30 and Ps30. The C-MAPSS W31 variable is retained as the Wf proxy used in the implementation.

## Evaluation

The same held-out engine-unit test set is used to evaluate the floating-point reference, pruning experiments, quantized artifact, and controlled robustness transformations. Reported classification metrics include accuracy, macro-F1, class-wise precision/recall/F1, and support. Binary anomaly ranking uses ROC-AUC and Average Precision.

## Pruning

Pruning is global unstructured magnitude pruning of kernel tensors. Reported target sparsities are 30% and 50%. Zero-valued weights alone do not imply proportional dense-runtime latency or Flash reduction.

## Quantization

INT8 PTQ uses a representative validation subset for calibration. The exported TensorFlow Lite artifact is executed on the held-out test samples. The quantized model must therefore be judged by both its artifact size and its post-conversion predictive performance.

## Robustness

Controlled transformations include additive noise, random sample dropout, temporal jitter, standardized temperature-channel offsets, and synthetic burst corruption. Standardized offsets and burst-rate coordinates are sensitivity parameters, not calibrated physical qualification levels.

## Runtime and energy

Notebook wall-time profiling is host-runtime profiling only. It is not an STM32H7 measurement. FP32 Keras and INT8 TFLite use different software paths and should not be presented as a controlled MCU speedup comparison. Energy based on an assumed 0.33 W is an estimate, not direct electrical measurement.

## QAT

Quantization-aware training is attempted only when compatible with the installed TensorFlow/TFMOT environment. Unsupported QAT is recorded as such; no numerical result should be inferred or fabricated.

## Recommended archival practice

For a publication release, archive the exact executed notebook together with the generated results ZIP, split IDs, scaler statistics, model artifacts, and software versions. Consider creating a DOI-backed release through Zenodo after acceptance or when the manuscript is made public.
