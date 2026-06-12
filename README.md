# Local LLM Wiki Model List

Model shortlist for a local workplace wiki/RAG setup using `llama.cpp` on Windows.

Target hardware:

- NVIDIA GeForce MX550, likely 2 GB VRAM
- Intel 12th gen i7 CPU
- No NPU requirement

Operational rule:

- Retrieval mode: embedding + reranker can be loaded together.
- Generation mode: main model must run alone.
- Do not keep main model loaded at the same time as embedding + reranker.

## Main Model

Use for summaries and synthesized wiki pages.

### Option 1: Recommended

Search/download:

```text
unsloth/Qwen3.5-2B-GGUF
Qwen3.5-2B-Q4_K_M.gguf
```

Notes:

- Best default for quality under the hardware limit.
- About 1.28 GB model weights.
- Expected to need partial GPU offload on MX550.

### Option 2: Alternative

Search/download:

```text
jordanwoodson/Qwen3.5-2B-heretic-GGUF
Qwen3.5-2B-heretic-Q4_K_M.gguf
```

Notes:

- Same size class as Qwen3.5-2B.
- Use if regular Qwen3.5 refuses harmless internal docs/procedures too often.
- Treat as an A/B test model, not the first production default.

### Option 3: VRAM Fallback

Search/download:

```text
unsloth/Qwen3.5-0.8B-GGUF
Qwen3.5-0.8B-Q4_K_M.gguf
```

Notes:

- Much easier on VRAM.
- Lower quality for synthesis across multiple source chunks.

### Outside Strict 2 GB VRAM

Search/download:

```text
unsloth/Qwen3.5-4B-GGUF
Qwen3.5-4B-Q4_K_M.gguf
```

```text
unsloth/gemma-4-E2B-it-GGUF
gemma-4-E2B-it-Q4_K_M.gguf
```

Notes:

- Better candidates only with CPU offload or more VRAM.
- Do not treat these as strict MX550 2 GB models.

## Embedding

Use for indexing chunks and embedding user queries.

### Option 1: Quality

Search/download:

```text
Qwen/Qwen3-Embedding-0.6B-GGUF
Qwen3-Embedding-0.6B-Q8_0.gguf
```

Notes:

- Preferred embedding model.
- About 639 MB model weights.

### Option 2: Balanced

Search/download:

```text
majentik/Qwen3-Embedding-0.6B-GGUF-Q5_K_M
qwen3-emb-0.6b-Q5_K_M.gguf
```

Notes:

- Preferred balanced embedding option.
- About 424-444 MB model weights.
- Better quality margin than Q4 with modest extra VRAM.

### Option 3: VRAM Saver

Search/download:

```text
andquant/Qwen3-Embedding-0.6B-Q4_K_M-GGUF
qwen3-embedding-0.6b-q4_k_m.gguf
```

Notes:

- Use when Q8 and Q5 leave too little VRAM.
- About 396 MB model weights.

## Reranker

Use after vector search to reorder the top chunks.

### Option 1: Recommended

Search/download:

```text
Voodisss/Qwen3-Reranker-0.6B-GGUF-llama_cpp
Qwen3-Reranker-0.6B-Q4_K_M.gguf
```

Notes:

- Preferred default.
- About 396 MB model weights.

### Option 2: Higher Quality If VRAM Allows

Search/download:

```text
Voodisss/Qwen3-Reranker-0.6B-GGUF-llama_cpp
Qwen3-Reranker-0.6B.Q8_0.gguf
```

Notes:

- About 639 MB model weights.
- Use only if retrieval mode still fits in VRAM.

### Option 3: Emergency VRAM

Search/download:

```text
Voodisss/Qwen3-Reranker-0.6B-GGUF-llama_cpp
Qwen3-Reranker-0.6B-Q3_K_M.gguf
```

Notes:

- Last resort when Q4 does not fit.
- About 347 MB model weights.

## Recommended Combos

Quality retrieval:

```text
embedding: Qwen/Qwen3-Embedding-0.6B-GGUF / Qwen3-Embedding-0.6B-Q8_0.gguf
reranker:  Voodisss/Qwen3-Reranker-0.6B-GGUF-llama_cpp / Qwen3-Reranker-0.6B-Q4_K_M.gguf
```

Balanced retrieval:

```text
embedding: majentik/Qwen3-Embedding-0.6B-GGUF-Q5_K_M / qwen3-emb-0.6b-Q5_K_M.gguf
reranker:  Voodisss/Qwen3-Reranker-0.6B-GGUF-llama_cpp / Qwen3-Reranker-0.6B-Q4_K_M.gguf
```

Main generation:

```text
main: unsloth/Qwen3.5-2B-GGUF / Qwen3.5-2B-Q4_K_M.gguf
```

## Runtime Notes

Default `llama.cpp` starting points:

```text
embedding: -c 2048 -ub 512 -ngl 999
reranker:  -c 2048 -ub 512 -ngl 999
main:      -c 4096 -ub 128 -ngl 16
```

If VRAM fails:

```text
1. Lower -c
2. Lower -ub
3. Lower -ngl
4. Move down one model tier
```

Suggested RAG settings:

```text
chunk size:       600-900 tokens
overlap:          80-150 tokens
vector top-k:     30-50
rerank final top: 5-8
```

See also:

- `outputs/llm-wiki-model-picks.md`
- `outputs/windows-llamacpp-agent-instructions.md`
