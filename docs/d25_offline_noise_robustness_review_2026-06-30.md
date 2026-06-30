# D25 Offline Noise Robustness Review - 2026-06-30

## Purpose

This review summarizes offline noise-injection tests performed on existing D25
bench traces. No new hardware data was collected and original CSV traces were
not modified.

The goal was to answer:

```text
What happens when signal noise corrupts the actuator traces?
```

## Method

The offline robustness tool injected controlled corruption into copied CSV
traces, converted the corrupted files through the existing hardware trace
converter, replayed the active detector, and summarized activation metrics.

Noise modes tested:

- Gaussian noise.
- Sparse spike noise.
- Dropout / stale-sample noise.
- Slow drift / bias.
- Quantization noise.

Corrupted signals:

- Velocity.
- Current.
- Voltage.
- Combined velocity + current + voltage.

## Main Results

### Fault-Window Robustness

Across the combined multi-signal sweep on the golden fault trace, the detector
had:

- `0` missed fault windows across all tested noise modes and severities.
- Strong robustness to spike and dropout noise.
- Moderate robustness to drift and quantization noise.
- The largest degradation under continuous Gaussian noise.

Worst tested combined-signal cases:

| Noise | Severity | Event mean | Event min | Baseline mean | Baseline max | Misses | Separation |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Gaussian | 0.10 | 0.739 | 0.729 | 0.418 | 0.428 | 0 | 0.322 |
| Spike | 0.10 | 0.966 | 0.960 | 0.148 | 0.180 | 0 | 0.818 |
| Dropout | 0.10 | 0.974 | 0.965 | 0.152 | 0.180 | 0 | 0.822 |
| Drift | 0.10 | 0.807 | 0.615 | 0.382 | 0.520 | 0 | 0.425 |
| Quantization | 0.10 | 0.927 | 0.920 | 0.445 | 0.487 | 0 | 0.482 |

### Clean-Baseline False-Positive Pressure

Clean baselines showed that spike/dropout noise did not meaningfully increase
activation, while Gaussian noise, drift, and some quantization levels increased
baseline activation.

Baseline activation average across the two clean traces:

| Noise | Severity | Baseline activation avg | Worst clean trace |
| --- | ---: | ---: | ---: |
| Gaussian | 0.005 | 0.167 | 0.193 |
| Gaussian | 0.010 | 0.202 | 0.224 |
| Gaussian | 0.020 | 0.236 | 0.263 |
| Gaussian | 0.030 | 0.276 | 0.289 |
| Gaussian | 0.050 | 0.333 | 0.348 |
| Drift | 0.005 | 0.152 | 0.174 |
| Drift | 0.010 | 0.180 | 0.219 |
| Drift | 0.020 | 0.212 | 0.268 |
| Drift | 0.030 | 0.257 | 0.284 |
| Drift | 0.050 | 0.282 | 0.340 |

## Interpretation

The detector's main noise risk is not missed fault windows in the tested range.
The main risk is elevated baseline activation under continuous signal
distortion.

Robust cases:

- Sparse velocity/current/voltage spikes.
- Dropout / stale-sample corruption.

More challenging cases:

- Continuous Gaussian perturbation.
- Slow drift / bias.
- Some quantization settings.

Practical tolerance boundary:

- Gaussian/drift severities near `0.005` to `0.01` remain relatively clean.
- False-positive pressure becomes more visible around `0.02` to `0.03`.
- Severity `0.05` and above should be treated as a stress condition rather than
  ordinary operating noise until calibrated against real noisy field data.

## Public Claim

Safe current claim:

```text
The detector was stress-tested offline by injecting controlled noise into copied
bench traces. In the tested ranges, fault-window misses remained at zero, while
continuous Gaussian/drift/quantization noise mainly increased baseline
activation. This identifies false-positive control as the main robustness target.
```


Tool:

- `tools/offline_noise_robustness.py`

