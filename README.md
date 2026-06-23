# TinyEdge GitHub Actions

Run [TinyEdge](https://tinyedge.ai) on **real edge devices** from your CI — benchmark, validate, and pick the best quantized build of a model, on every change.

Both actions install the `tinyedge` SDK and call it; you supply a `TINYEDGE_API_KEY` secret and have a device online during the run.

| Action | Use | Docs |
|--------|-----|------|
| **[`decide`](decide)** | sweep the quant ladder on a device, pick the best variant for a deployment profile, emit a **signed manifest** | [decide/](decide/README.md) |
| **[`validate`](validate)** | benchmark a model vs a saved baseline and **fail the build on regression** | [validate/](validate/README.md) |

## Quick start

```yaml
# .github/workflows/edge.yml
on: { workflow_dispatch: {} }
jobs:
  decide:
    runs-on: ubuntu-latest
    steps:
      - uses: TinyEdgeAI/tinyedge-actions/decide@v1
        with:
          api-key: ${{ secrets.TINYEDGE_API_KEY }}
          model: "hf:bartowski/Llama-3.2-1B-Instruct-GGUF"
          device: jetson-orin-nano
          profile: max_quality
      - uses: actions/upload-artifact@v4
        if: steps.decide.outputs.verdict == 'ok'
        with: { name: manifest, path: tinyedge-manifest.json }
```

See a full example project at [TinyEdgeAI/edge-llm-ci-example](https://github.com/TinyEdgeAI/edge-llm-ci-example).

## Versioning

Reference a stable major with `@v1`. Each action's inputs/outputs are documented in its folder.

---

<sub>Powered by [TinyEdge](https://tinyedge.ai). The SDK these actions wrap is `pip install tinyedge`.</sub>
