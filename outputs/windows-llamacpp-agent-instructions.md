# Windows llama.cpp Agent Instructions

Target machine:

- OS: Windows
- GPU: NVIDIA GeForce MX550, likely 2 GB VRAM
- CPU: Intel Core i7 12th gen
- iGPU: Intel, no NPU
- Runtime target: local `llama.cpp`
- Models: user will provide local `.gguf` paths

Hard rule:

- Never keep the main generation model loaded at the same time as embedding + reranker.
- Operating modes are mutually exclusive:
  - Retrieval mode: embedding server + reranker server may run together.
  - Generation mode: main model server runs alone.

## 1. Install Locally Into The Repo

Use PowerShell.

Required system software:

- NVIDIA driver with working `nvidia-smi`
- Git for Windows
- CMake
- Visual Studio Build Tools 2022 with:
  - Desktop development with C++
  - MSVC C++ toolset
  - Windows SDK
  - CMake tools for Windows
- NVIDIA CUDA Toolkit 12.x

Check prerequisites:

```powershell
nvidia-smi
git --version
cmake --version
where cl
nvcc --version
```

If `where cl` fails, run commands from "Developer PowerShell for VS 2022".

Clone and build `llama.cpp` inside the project repo:

```powershell
mkdir .\vendor -Force
git clone https://github.com/ggml-org/llama.cpp .\vendor\llama.cpp

cmake -S .\vendor\llama.cpp `
  -B .\vendor\llama.cpp\build `
  -DGGML_CUDA=ON `
  -DLLAMA_CURL=ON

cmake --build .\vendor\llama.cpp\build --config Release -j
```

Expected binary:

```powershell
.\vendor\llama.cpp\build\bin\Release\llama-server.exe --help
```

If CUDA build fails, build CPU-only as fallback:

```powershell
cmake -S .\vendor\llama.cpp `
  -B .\vendor\llama.cpp\build-cpu `
  -DLLAMA_CURL=ON

cmake --build .\vendor\llama.cpp\build-cpu --config Release -j
```

Expected CPU fallback binary:

```powershell
.\vendor\llama.cpp\build-cpu\bin\Release\llama-server.exe --help
```

Primary sources:

- `llama.cpp` build docs: https://github.com/ggml-org/llama.cpp/blob/master/docs/build.md
- `llama-server` docs: https://github.com/ggml-org/llama.cpp/blob/master/tools/server/README.md

## 2. Model Candidates

Use these as ranked candidates. The user supplies exact local paths.

### Main Model: Summary And Wiki Page Synthesis

Tier A, recommended:

- Repo: `unsloth/Qwen3.5-2B-GGUF`
- File: `Qwen3.5-2B-Q4_K_M.gguf`
- Weights: about 1.28 GB
- Expected mode: partial CUDA offload on MX550
- Start with `ctx=4096`

Tier B, more conservative:

- Repo: `unsloth/Qwen3.5-0.8B-GGUF`
- File: `Qwen3.5-0.8B-Q4_K_M.gguf`
- Weights: about 533 MB
- Expected mode: full or near-full CUDA offload possible
- Use if Qwen3.5-2B is too slow or unstable

Tier C, higher quality but outside strict 2 GB VRAM:

- Repo: `unsloth/Qwen3.5-4B-GGUF`
- File: `Qwen3.5-4B-Q4_K_M.gguf`
- Weights: about 2.74 GB
- Expected mode: CPU-heavy offload only on MX550
- Do not use if strict GPU residency is required

Gemma 4 note:

- `unsloth/gemma-4-E2B-it-GGUF`, `gemma-4-E2B-it-Q4_K_M.gguf`, is about 3.11 GB.
- It is not a strict 2 GB VRAM model.
- Test only with CPU offload if Qwen3.5 quality is unacceptable.

### Embedding + Reranker Tiers

Tier 1, quality:

- Embedding repo: `Qwen/Qwen3-Embedding-0.6B-GGUF`
- Embedding file: `Qwen3-Embedding-0.6B-Q8_0.gguf`
- Reranker repo: `Voodisss/Qwen3-Reranker-0.6B-GGUF-llama_cpp`
- Reranker file: `Qwen3-Reranker-0.6B-Q4_K_M.gguf`
- Weights total: about 1.04 GB
- Expected VRAM: about 1.6-1.9 GB with `ctx=2048`

Tier 2, balanced VRAM:

- Embedding repo: `majentik/Qwen3-Embedding-0.6B-GGUF-Q5_K_M`
- Embedding file: `qwen3-emb-0.6b-Q5_K_M.gguf`
- Reranker repo: `Voodisss/Qwen3-Reranker-0.6B-GGUF-llama_cpp`
- Reranker file: `Qwen3-Reranker-0.6B-Q5_K_M.gguf`
- Weights total: about 868-888 MB
- Expected VRAM: about 1.3-1.65 GB with `ctx=2048`

Tier 3, emergency VRAM:

- Embedding repo: `andquant/Qwen3-Embedding-0.6B-Q4_K_M-GGUF`
- Embedding file: `qwen3-embedding-0.6b-q4_k_m.gguf`
- Reranker repo: `Voodisss/Qwen3-Reranker-0.6B-GGUF-llama_cpp`
- Reranker file: `Qwen3-Reranker-0.6B-Q3_K_M.gguf`
- Weights total: about 743 MB
- Use only if Tier 2 fails on VRAM

## 3. VRAM Diagnostics

Always record VRAM before, during, and after each server launch.

One-shot:

```powershell
nvidia-smi --query-gpu=name,driver_version,memory.total,memory.used,memory.free,utilization.gpu --format=csv
```

Live monitor:

```powershell
nvidia-smi --query-gpu=timestamp,name,memory.total,memory.used,memory.free,utilization.gpu --format=csv -l 1
```

Process view:

```powershell
nvidia-smi pmon -s um
```

If a launch fails:

1. Record the exact command.
2. Record the last 80 lines of `llama-server` output.
3. Record `nvidia-smi` output.
4. Retry with lower settings in this order:
   - lower `-c`
   - lower `-ub`
   - lower `-ngl`
   - switch to smaller quant/model tier

Do not claim a model fits until the server is running and a test request completes.

## 4. Operating Modes

Set paths in PowerShell before launching:

```powershell
$LLAMA = ".\vendor\llama.cpp\build\bin\Release\llama-server.exe"

$MODELS_ROOT = "$env:USERPROFILE\.lmstudio\models"

$EMBED_MODEL = "$MODELS_ROOT\Qwen\Qwen3-Embedding-0.6B-GGUF\Qwen3-Embedding-0.6B-Q8_0.gguf"
$RERANK_MODEL = "$MODELS_ROOT\Voodisss\Qwen3-Reranker-0.6B-GGUF-llama_cpp\Qwen3-Reranker-0.6B-Q4_K_M.gguf"
$MAIN_MODEL = "$MODELS_ROOT\unsloth\Qwen3.5-2B-GGUF\Qwen3.5-2B-Q4_K_M.gguf"
```

For every listed model path, see `outputs/windows-model-paths.md`.

Verify selected paths:

```powershell
@($EMBED_MODEL, $RERANK_MODEL, $MAIN_MODEL) | ForEach-Object {
  if (-not (Test-Path $_)) {
    Write-Error "Missing model file: $_"
  } else {
    Write-Host "OK: $_"
  }
}
```

### Retrieval Mode

Start embedding server:

```powershell
& $LLAMA `
  -m $EMBED_MODEL `
  --embedding `
  --pooling last `
  -c 2048 `
  -ub 512 `
  -ngl 999 `
  --cache-type-k q8_0 `
  --cache-type-v q8_0 `
  --host 127.0.0.1 `
  --port 8001
```

Start reranker server in a second PowerShell window:

```powershell
& $LLAMA `
  -m $RERANK_MODEL `
  --reranking `
  --embedding `
  --pooling rank `
  -c 2048 `
  -ub 512 `
  -ngl 999 `
  --cache-type-k q8_0 `
  --cache-type-v q8_0 `
  --host 127.0.0.1 `
  --port 8002
```

If either fails, reduce:

```text
-ub 512 -> 256 -> 128
-c 2048 -> 1024
-ngl 999 -> 24 -> 16 -> 12 -> 8 -> 0
```

Embedding test:

```powershell
$body = @{
  model = "embedding"
  input = @("Testowy fragment dokumentacji wewnetrznej.")
} | ConvertTo-Json -Depth 5

Invoke-RestMethod `
  -Uri "http://127.0.0.1:8001/v1/embeddings" `
  -Method Post `
  -ContentType "application/json" `
  -Body $body
```

Rerank test:

```powershell
$body = @{
  model = "reranker"
  query = "Jak zresetowac haslo?"
  documents = @(
    "Procedura resetowania hasla jest dostepna w panelu konta.",
    "Urlop nalezy zglosic w systemie HR."
  )
} | ConvertTo-Json -Depth 5

Invoke-RestMethod `
  -Uri "http://127.0.0.1:8002/v1/rerank" `
  -Method Post `
  -ContentType "application/json" `
  -Body $body
```

### Generation Mode

Before launching the main model:

```powershell
Get-Process llama-server -ErrorAction SilentlyContinue | Stop-Process
nvidia-smi
```

Start Qwen3.5-2B main model:

```powershell
& $LLAMA `
  -m $MAIN_MODEL `
  -c 4096 `
  -ub 128 `
  -ngl 16 `
  --cache-type-k q8_0 `
  --cache-type-v q8_0 `
  --flash-attn `
  --host 127.0.0.1 `
  --port 8003
```

If it fails, retry:

```text
Attempt 1: -c 4096 -ub 128 -ngl 16
Attempt 2: -c 4096 -ub 64  -ngl 12
Attempt 3: -c 2048 -ub 64  -ngl 12
Attempt 4: -c 2048 -ub 64  -ngl 8
Attempt 5: -c 2048 -ub 64  -ngl 0
```

Chat/synthesis test:

```powershell
$body = @{
  model = "main"
  messages = @(
    @{
      role = "system"
      content = "You write concise internal wiki pages. Preserve factual details. Do not invent missing facts."
    },
    @{
      role = "user"
      content = "Zsyntetyzuj krotka strone wiki z tych notatek: 1. Reset hasla jest w panelu konta. 2. MFA trzeba potwierdzic aplikacja. 3. W razie blokady kontakt z helpdeskiem."
    }
  )
  temperature = 0.2
  max_tokens = 500
} | ConvertTo-Json -Depth 8

Invoke-RestMethod `
  -Uri "http://127.0.0.1:8003/v1/chat/completions" `
  -Method Post `
  -ContentType "application/json" `
  -Body $body
```

## 5. Query Flow For The App

Retrieval flow:

1. Chunk source documents into 600-900 tokens.
2. Use 80-150 token overlap.
3. Embed all chunks through port `8001`.
4. Store vectors in the app's vector DB.
5. For a user query, embed query through port `8001`.
6. Retrieve top 30-50 chunks by vector similarity.
7. Rerank those chunks through port `8002`.
8. Keep final top 5-8 chunks.
9. Stop embedding/reranker servers.
10. Start main model on port `8003`.
11. Generate the synthesized wiki page from selected chunks.

Generation prompt constraints:

- Use low temperature: `0.1-0.3`.
- Ask for structured wiki output.
- Tell the model not to invent facts.
- Require a "Missing information" section when inputs are incomplete.

## 6. Operational Rules

Use CUDA build first. Use CPU build only as fallback.

On MX550:

- Prefer partial offload for main model.
- Use full offload only for small embedding/reranker models if VRAM allows.
- Avoid long context by default.
- Do not use Gemma 4 as a strict 2 GB VRAM model.
- Do not run LM Studio at the same time if it has models loaded.

Default settings:

```text
embedding: -c 2048 -ub 512 -ngl 999
reranker:  -c 2048 -ub 512 -ngl 999
main:      -c 4096 -ub 128 -ngl 16
```

Conservative settings:

```text
embedding: -c 2048 -ub 256 -ngl 16
reranker:  -c 2048 -ub 256 -ngl 16
main:      -c 2048 -ub 64  -ngl 8
```

Failure policy:

- If CUDA OOM occurs, do not keep retrying the same command.
- Reduce `-c`, `-ub`, then `-ngl`.
- If still failing, move down one model tier.
- Log all tested command lines and measured VRAM.
