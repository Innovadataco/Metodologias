<!-- source: website/docs/guides/local-llm-on-mac.md -->
# Run Local LLMs on Mac

# Run Local LLMs on Mac

This guide walks you through running a local LLM server on macOS with an OpenAI-compatible API. You get full privacy, zero API costs, and surprisingly good performance on Apple Silicon.

We cover two backends:

| Backend | Install | Best at | Format |
|---------|---------|---------|--------|
| **llama.cpp** | `brew install llama.cpp` | Fastest time-to-first-token, quantized KV cache for low memoria | GGUF |
| **omlx** | [omlx.ai](https://omlx.ai) | Fastest token generation, native Metal optimization | MLX (safetensors) |

Both expose an OpenAI-compatible `/v1/chat/completions` endpoint. Hermes works with either one — just point it at `http://localhost:8080` or `http://localhost:8000`.

:::info Apple Silicon only
This guide targets Macs with Apple Silicon (M1 and later). Intel Macs will work with llama.cpp but without GPU acceleration — expect significantly slower performance.
:::

---

## Choosing a modelo

For getting started, we recommend **Qwen3.5-9B** — it's a strong reasoning modelo that fits comfortably in 8GB+ of unified memoria with quantization.

| Variant | Size on disk | RAM needed (128K contexto) | Backend |
|---------|-------------|---------------------------|---------|
| Qwen3.5-9B-Q4_K_M (GGUF) | 5.3 GB | ~10 GB with quantized KV cache | llama.cpp |
| Qwen3.5-9B-mlx-lm-mxfp4 (MLX) | ~5 GB | ~12 GB | omlx |

**Memoria rule of thumb:** modelo size + KV cache. A 9B Q4 modelo is ~5 GB. The KV cache at 128K contexto with Q4 quantization adds ~4-5 GB. With default (f16) KV cache, that balloons to ~16 GB. The quantized KV cache flags in llama.cpp are the key trick for memoria-constrained systems.

For larger modelos (27B, 35B), you'll need 32 GB+ of unified memoria. The 9B is the sweet spot for 8-16 GB machines.

---

## Option A: llama.cpp

llama.cpp is the most portable local LLM runtime. On macOS it uses Metal for GPU acceleration out of the box.

### Install

```bash
brew install llama.cpp
```

This gives you the `llama-server` command globally.

### Download the modelo

You need a GGUF-format modelo. The easiest source is Hugging Face via the `huggingface-cli`:

```bash
brew install huggingface-cli
```

Then download:

```bash
huggingface-cli download unsloth/Qwen3.5-9B-GGUF Qwen3.5-9B-Q4_K_M.gguf --local-dir ~/modelos
```

:::tip Gated modelos
Some modelos on Hugging Face require autenticación. Run `huggingface-cli login` first if you get a 401 or 404 error.
:::

### Start the server

```bash
llama-server -m ~/modelos/Qwen3.5-9B-Q4_K_M.gguf \
  -ngl 99 \
  -c 131072 \
  -np 1 \
  -fa on \
  --cache-type-k q4_0 \
  --cache-type-v q4_0 \
  --host 0.0.0.0
```

Here's what each flag does:

| Flag | Purpose |
|------|---------|
| `-ngl 99` | Offload all layers to GPU (Metal). Use a high number to ensure nothing stays on CPU. |
| `-c 131072` | Contexto window size (128K tokens). Reduce this if you're low on memoria. |
| `-np 1` | Number of parallel slots. Keep at 1 for single-user use — more slots split your memoria budget. |
| `-fa on` | Flash attention. Reduces memoria usage and speeds up long-contexto inference. |
| `--cache-type-k q4_0` | Quantize the key cache to 4-bit. **This is the big memoria saver.** |
| `--cache-type-v q4_0` | Quantize the value cache to 4-bit. Together with the above, this cuts KV cache memoria by ~75% vs f16. |
| `--host 0.0.0.0` | Listen on all interfaces. Use `127.0.0.1` if you don't need network access. |

The server is ready when you see:

```
main: server is listening on http://0.0.0.0:8080
srv  update_slots: all slots are idle
```

### Memoria optimization for constrained systems

The `--cache-type-k q4_0 --cache-type-v q4_0` flags are the most important optimization for systems with limited memoria. Here's the impact at 128K contexto:

| KV cache type | KV cache memoria (128K ctx, 9B modelo) |
|---------------|--------------------------------------|
| f16 (default) | ~16 GB |
| q8_0 | ~8 GB |
| **q4_0** | **~4 GB** |

On an 8 GB Mac, use `q4_0` KV cache and choose a smaller modelo that can still fit Hermes' 64K minimum contexto. On 16 GB, you can comfortably do 128K contexto. On 32 GB+, you can run larger modelos or multiple parallel slots.

If you're still running out of memoria, reduce contexto only while staying at or above Hermes' 64K minimum; otherwise switch to a smaller modelo or smaller quantization (Q3_K_M instead of Q4_K_M).

### Test it

```bash
curl -s http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "modelo": "Qwen3.5-9B-Q4_K_M.gguf",
    "messages": [{"role": "user", "content": "Hello!"}],
    "max_tokens": 50
  }' | jq .choices[0].message.content
```

### Get the modelo name

If you forget the modelo name, query the modelos endpoint:

```bash
curl -s http://localhost:8080/v1/modelos | jq '.data[].id'
```

---

## Option B: MLX via omlx

[omlx](https://omlx.ai) is a macOS-native app that manages and serves MLX modelos. MLX is Apple's own machine learning framework, optimized specifically for Apple Silicon's unified memoria architecture.

### Install

Download and install from [omlx.ai](https://omlx.ai). It provides a GUI for modelo management and a built-in server.

### Download the modelo

Use the omlx app to browse and download modelos. Search for `Qwen3.5-9B-mlx-lm-mxfp4` and download it. Modelos are stored locally (typically in `~/.omlx/modelos/`).

### Start the server

omlx serves modelos on `http://127.0.0.1:8000` by default. Start serving from the app UI, or use the CLI if available.

### Test it

```bash
curl -s http://127.0.0.1:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "modelo": "Qwen3.5-9B-mlx-lm-mxfp4",
    "messages": [{"role": "user", "content": "Hello!"}],
    "max_tokens": 50
  }' | jq .choices[0].message.content
```

### List available modelos

omlx can serve multiple modelos simultaneously:

```bash
curl -s http://127.0.0.1:8000/v1/modelos | jq '.data[].id'
```

---

## Benchmarks: llama.cpp vs MLX

Both backends tested on the same machine (Apple M5 Max, 128 GB unified memoria) running the same modelo (Qwen3.5-9B) at comparable quantization levels (Q4_K_M for GGUF, mxfp4 for MLX). Five diverse prompts, three runs each, backends tested sequentially to avoid resource contention.

### Results

| Metric | llama.cpp (Q4_K_M) | MLX (mxfp4) | Winner |
|--------|-------------------|-------------|--------|
| **TTFT (avg)** | **67 ms** | 289 ms | llama.cpp (4.3x faster) |
| **TTFT (p50)** | **66 ms** | 286 ms | llama.cpp (4.3x faster) |
| **Generation (avg)** | 70 tok/s | **96 tok/s** | MLX (37% faster) |
| **Generation (p50)** | 70 tok/s | **96 tok/s** | MLX (37% faster) |
| **Total time (512 tokens)** | 7.3s | **5.5s** | MLX (25% faster) |

### What this means

- **llama.cpp** excels at prompt processing — its flash attention + quantized KV cache pipeline gets you the first token in ~66ms. If you're building interactive applications where perceived responsiveness matters (chatbots, autocomplete), this is a meaningful advantage.

- **MLX** generates tokens ~37% faster once it gets going. For batch workloads, long-form generation, or any task where total completion time matters more than initial latency, MLX finishes sooner.

- Both backends are **extremely consistent** — variance across runs was negligible. You can rely on these numbers.

### Which one should you pick?

| Use case | Recommendation |
|----------|---------------|
| Interactive chat, low-latency herramientas | llama.cpp |
| Long-form generation, bulk processing | MLX (omlx) |
| Memoria-constrained (8-16 GB) | llama.cpp (quantized KV cache is unmatched) |
| Serving multiple modelos simultaneously | omlx (built-in multi-modelo support) |
| Maximum compatibility (Linux too) | llama.cpp |

---

## Connect to Hermes

Once your local server is running:

```bash
hermes modelo
```

Select **Custom endpoint** and follow the prompts. It will ask for the base URL and modelo name — use the values from whichever backend you set up above.

---

## Timeouts

Hermes automatically detects local endpoints (localhost, LAN IPs) and relaxes its streaming timeouts. No configuración needed for most setups.

If you still hit timeout errors (e.g. very large contextos on slow hardware), you can override the streaming read timeout:

```bash
# In your .env — raise from the 120s default to 30 minutes
HERMES_STREAM_READ_TIMEOUT=1800
```

| Timeout | Default | Local auto-adjustment | Env var override |
|---------|---------|----------------------|------------------|
| Stream read (socket-level) | 120s | Raised to 1800s | `HERMES_STREAM_READ_TIMEOUT` |
| Stale stream detection | 180s | Disabled entirely | `HERMES_STREAM_STALE_TIMEOUT` |
| API call (non-streaming) | 1800s | No change needed | `HERMES_API_TIMEOUT` |

The stream read timeout is the one most likely to cause issues — it's the socket-level deadline for receiving the next chunk of data. During prefill on large contextos, local modelos may produce no output for minutes while processing the prompt. The auto-detection handles this transparently.

---