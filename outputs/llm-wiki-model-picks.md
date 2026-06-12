# Local LLM Wiki: Embedding + Reranker Picks

Goal: local RAG/wiki search with llama.cpp / LM Studio, targeting about 2 GB VRAM.

## Pick 1: Quality First

Embedding:

- Repo: `Qwen/Qwen3-Embedding-0.6B-GGUF`
- File: `Qwen3-Embedding-0.6B-Q8_0.gguf`
- Size: about 639 MB

Reranker:

- Repo: `Voodisss/Qwen3-Reranker-0.6B-GGUF-llama_cpp`
- File: `Qwen3-Reranker-0.6B-Q4_K_M.gguf`
- Size: about 396 MB

Estimated budget:

- Model weights: about 1.04 GB
- Practical VRAM: about 1.6-1.9 GB with `ctx=2048`
- With `ctx=4096`, add roughly 250-400 MB

## Pick 2: Balanced VRAM

Embedding:

- Repo: `majentik/Qwen3-Embedding-0.6B-GGUF-Q5_K_M`
- File: `qwen3-emb-0.6b-Q5_K_M.gguf`
- Size: about 424-444 MB

Reranker:

- Repo: `Voodisss/Qwen3-Reranker-0.6B-GGUF-llama_cpp`
- File: `Qwen3-Reranker-0.6B-Q5_K_M.gguf`
- Size: about 444 MB

Estimated budget:

- Model weights: about 868-888 MB
- Practical VRAM: about 1.3-1.65 GB with `ctx=2048`
- Better margin for `ctx=4096`

## Main Model: Summary And Wiki Page Synthesis

Recommended local generation model:

- Repo: `unsloth/Qwen3.5-2B-GGUF`
- File: `Qwen3.5-2B-Q4_K_M.gguf`
- Size: about 1.28 GB

Estimated budget:

- Practical VRAM: about 1.9-2.5 GB depending on context length and KV cache
- Start with `ctx=4096`
- Use `ctx=8192` only if memory allows

More conservative fallback:

- Repo: `unsloth/Qwen3.5-0.8B-GGUF`
- File: `Qwen3.5-0.8B-Q4_K_M.gguf`
- Size: about 533 MB

Higher-quality upgrade:

- Repo: `unsloth/Qwen3.5-4B-GGUF`
- File: `Qwen3.5-4B-Q4_K_M.gguf`
- Size: about 2.74 GB
- This is outside a strict 2 GB VRAM budget

Gemma 4 option:

- Repo: `unsloth/gemma-4-E2B-it-GGUF`
- File: `gemma-4-E2B-it-Q4_K_M.gguf`
- Size: about 3.11 GB
- This is outside a strict 2 GB VRAM budget, but worth testing with CPU offload

Avoid for generation in this setup:

- Gemma 3 generation models
- Qwen3 generation models
- They remain acceptable only for the embedding/reranker roles listed above

## Runtime Notes

Start with:

```bash
-c 2048 -ub 512
```

If chunks are longer or important procedures get split too aggressively:

```bash
-c 4096 -ub 256
```

Suggested RAG settings:

- Chunk size: 600-900 tokens
- Overlap: 80-150 tokens
- Vector top-k: 30-50
- Rerank final top-k: 5-8

## llama.cpp Example

Embedding server:

```bash
llama-server \
  -m Qwen3-Embedding-0.6B-Q8_0.gguf \
  --embedding --pooling last \
  -c 2048 -ub 512 --port 8001
```

Reranker server:

```bash
llama-server \
  -m Qwen3-Reranker-0.6B-Q4_K_M.gguf \
  --reranking --pooling rank \
  -c 2048 -ub 512 --port 8002
```

## Public Repo Hygiene

Safe to publish:

- Public model identifiers
- Public Hugging Face links
- Generic llama.cpp commands
- Generic VRAM estimates
- Generic chunking settings

Do not publish:

- Company documents or examples
- Internal URLs, hostnames, repo names, wiki paths
- API keys, tokens, secrets
- Work-specific prompts or system prompts
- Benchmarks/results based on confidential data
