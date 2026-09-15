# 2026 Measurement Major-Revision Update

This update aligns the repository with the revised manuscript **MEAS-D-26-17511**.

## Main corrections

- Reframed the task from sensor-health monitoring to **RUL-derived turbofan degradation-state classification**.
- Corrected the executed window construction to **30 samples, stride 5, 83.33% overlap**.
- Replaced unsupported physical channel aliases with dataset-native indices.
- Added ten-seed repeatability analysis.
- Added matched continued-training controls for 30% and 50% unstructured magnitude sparsity.
- Added Keras FP32 → TFLite FP32 → TFLite INT8 conversion control.
- Added exact robustness-operator specifications.
- Added computational accounting (parameters, FLOPs/MACs, activation and artifact size).
- Added controlled baselines and architecture ablations.
- Removed MCU latency/energy implications unsupported by physical hardware measurements.

## Interpretation changes

The revised study does **not** claim that pruning improves accuracy or acts as regularization. It shows performance preservation under 30–50% unstructured sparsity relative to matched continued training.

The revised study also does **not** claim successful INT8 deployment. FP32 TFLite conversion is faithful, whereas the tested INT8 PTQ path exhibits substantial predictive degradation.
