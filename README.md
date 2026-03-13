# Soprano ONNX Streaming — Instant Text‑to‑Speech in the Browser (CPU-Optimized WASM)

[![Upstream](https://img.shields.io/badge/Upstream-ekwek1%2Fsoprano-black?logo=github)](https://github.com/ekwek1/soprano)
[![Hugging Face Model](https://img.shields.io/badge/HuggingFace-Model-orange?logo=huggingface)](https://huggingface.co/KevinAHM/soprano-onnx)
[![Hugging Face Demo for Soprano Web Onnx](https://img.shields.io/badge/HuggingFace-Demo-yellow?logo=huggingface)](https://huggingface.co/spaces/KevinAHM/soprano-web-onnx)

A **static, client-side** browser demo that runs the Soprano TTS pipeline using **onnxruntime-web**.

This is a conversion/port of the original Soprano project:
https://github.com/ekwek1/soprano

---

## Requirements

- A modern browser (Chrome, Edge, Firefox, Safari).
- You must serve this folder over HTTP (opening `index.html` via `file://` usually breaks `fetch()` / module loading).
- The demo loads `onnxruntime-web` and `@huggingface/transformers` from a CDN by default (network required unless you vendor them).
- The model files are large; plan to use **Git LFS** or GitHub Releases if you publish them.

---

## Folder layout

Place model artifacts under `./models/`:

```text
.
├─ index.html
├─ onnx-streaming.js
├─ PCMPlayerWorklet.js
├─ style.css
├─ onnx/
│  ├─ soprano_backbone_kv.onnx
│  ├─ soprano_decoder.onnx
│  └─ soprano_decoder.onnx.data
├─ tokenizer.json
├─ tokenizer_config.json
├─ special_tokens_map.json
├─ config.json
└─ generation_config.json
```

Notes:
- ONNX models live in `onnx/` following HuggingFace convention.
- The decoder uses external weights (`.onnx.data` file must be present alongside the `.onnx` file).
- Tokenizer files are in the root directory.

---

## Run locally

Use any static file server from this directory, for example:

```bash
python -m http.server 8085
```

Then open `http://localhost:8085`.

### For maximum CPU throughput

The runtime now enables the fastest browser CPU path that `onnxruntime-web` can use automatically:

- **WASM SIMD** is enabled by default to take advantage of vectorized CPU execution, which improves performance on modern browsers running on CPUs with strong SIMD support such as AVX2-capable x86 processors.
- **Threaded WASM** is enabled automatically when the page is served in a **cross-origin isolated** context, so multi-core CPUs can be used for inference.
- If you are running with a native ONNX Runtime binding that exposes **OpenVINO**, you can request it explicitly with `?ep=openvino,wasm` and the app will fall back to WASM if OpenVINO is unavailable.

To unlock threaded WASM in browsers, serve the app with these response headers:

```text
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

---

## Configuration

Model paths are defined near the top of `onnx-streaming.js` in the `MODELS` object.

Sampling defaults are set in `onnx-streaming.js` (constructor):
- `temperature`
- `topK`
- `topP`
- `repetitionPenalty`

Runtime defaults are also tuned in `onnx-streaming.js` for CPU inference:
- `ort.env.wasm.simd = true`
- `ort.env.wasm.numThreads` is auto-sized from `navigator.hardwareConcurrency` when cross-origin isolation is available
- `graphOptimizationLevel = 'all'`
- inference yields to the UI less frequently and reuses decode buffers to reduce per-token overhead

---

## Troubleshooting

- **"Load failed" / model never becomes Ready**
  - Verify the `onnx/` filenames match `MODELS` in `onnx-streaming.js`
  - Check DevTools → Network for a missing `.onnx.data` file (404)
  - Confirm `/` contains `tokenizer.json` (and related files)
- **Performance notes**
  - Both backbone and decoder run on CPU-oriented execution providers
  - Browser builds prefer tuned **WASM SIMD** and will use multithreaded WASM when the page is cross-origin isolated
  - Native environments can opt into **OpenVINO** with `?ep=openvino,wasm`
  - Achieves better real-time streaming on modern hardware by reducing per-token JS overhead

---

## License & attribution

Soprano is released under **Apache-2.0** in the upstream repository:
https://github.com/ekwek1/soprano




