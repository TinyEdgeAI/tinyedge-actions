# TinyEdge Validate — GitHub Action

Benchmark your model on **real edge devices** on every change and **fail the build
when it regresses** against a saved baseline (latency, memory, accuracy, thermals).

It's a thin wrapper around the `tinyedge` CLI: it installs the SDK, runs
`tinyedge validate`, and the exit code gates your job.

## Prerequisites

1. A TinyEdge API key → store it as a repo secret, e.g. `TINYEDGE_API_KEY`.
2. A **baseline** to compare against. Create one from a known-good run:
   - in the console at **Validation → save a baseline**, or
   - `tinyedge baseline save <job-id> --name "v1 / oppo-a74"` → copy the id.

## Usage

```yaml
name: Edge validation
on: [pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Validate on real devices
        uses: TinyEdgeAI/tinyedge-agent/actions/validate@main
        with:
          api-key: ${{ secrets.TINYEDGE_API_KEY }}
          model: ./build/model.onnx
          baseline: bl_xxxxxxxx          # your baseline id
          device: oppo-a74               # optional; defaults to the baseline's device
          tolerance: 10                  # fail if a perf metric is >10% worse than baseline
          max-latency: 220               # hard p50 latency budget (ms)
          min-accuracy: 0.85             # hard accuracy floor
```

The job **fails (exit 1)** if any metric regresses past your thresholds, with a
per-metric breakdown in the log, and **passes (exit 0)** otherwise.

## Inputs

| Input | Required | Description |
|---|---|---|
| `api-key` | yes | TinyEdge API key (use a secret). |
| `baseline` | yes | Baseline id to compare against. |
| `model` | – | Path to the model file (`.onnx`/`.gguf`). Omit if using `model-id`. |
| `model-id` | – | Id of an already-uploaded model. |
| `device` | – | Device id. Defaults to the baseline's device. |
| `runtime` | – | Runtime override (e.g. `onnxruntime`, `tensorrt`, `llamacpp`). |
| `precision` | – | `fp32` (default) `\| fp16 \| int8 \| int4`. |
| `dataset` | – | Labeled test set (folder/archive) for on-device accuracy. |
| `tolerance` | – | Allowed % perf drift vs baseline (default 10). |
| `max-latency` | – | Absolute p50 latency budget (ms). |
| `max-rss` | – | Absolute peak-RAM budget (MB). |
| `min-accuracy` | – | Absolute minimum top-1 accuracy (0–1). |
| `api-base` | – | API base URL (default `https://tinyedge.ai/api`). |
| `tinyedge-version` | – | Pin the SDK version (default: latest, needs ≥ 0.8.0). |

## Notes

- Needs the `tinyedge` SDK **≥ 0.8.0** on PyPI (the version that ships
  `tinyedge validate`).
- A device must be online to claim the job; the run blocks until it completes or
  times out.
