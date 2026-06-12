# Windows Model Path Templates

Expected local model directory layout:

```text
<MODELS_ROOT>\<hf-owner-or-org>\<hf-repo-name>\<gguf-file>
```

LM Studio usually stores models under:

```text
%USERPROFILE%\.lmstudio\models
```

For example:

```text
%USERPROFILE%\.lmstudio\models\Voodisss\Qwen3-Reranker-0.6B-GGUF-llama_cpp\Qwen3-Reranker-0.6B-Q4_K_M.gguf
```

Use this PowerShell root variable:

```powershell
$MODELS_ROOT = "$env:USERPROFILE\.lmstudio\models"
```

If models are stored somewhere else, change only `$MODELS_ROOT`.

## Main Model Paths

Recommended:

```powershell
$MAIN_QWEN35_2B_Q4 = "$MODELS_ROOT\unsloth\Qwen3.5-2B-GGUF\Qwen3.5-2B-Q4_K_M.gguf"
```

Alternative:

```powershell
$MAIN_QWEN35_2B_HERETIC_Q4 = "$MODELS_ROOT\jordanwoodson\Qwen3.5-2B-heretic-GGUF\Qwen3.5-2B-heretic-Q4_K_M.gguf"
```

VRAM fallback:

```powershell
$MAIN_QWEN35_08B_Q4 = "$MODELS_ROOT\unsloth\Qwen3.5-0.8B-GGUF\Qwen3.5-0.8B-Q4_K_M.gguf"
```

Outside strict 2 GB VRAM:

```powershell
$MAIN_QWEN35_4B_Q4 = "$MODELS_ROOT\unsloth\Qwen3.5-4B-GGUF\Qwen3.5-4B-Q4_K_M.gguf"
$MAIN_GEMMA4_E2B_Q4 = "$MODELS_ROOT\unsloth\gemma-4-E2B-it-GGUF\gemma-4-E2B-it-Q4_K_M.gguf"
```

## Embedding Paths

Quality:

```powershell
$EMBED_QWEN3_06B_Q8 = "$MODELS_ROOT\Qwen\Qwen3-Embedding-0.6B-GGUF\Qwen3-Embedding-0.6B-Q8_0.gguf"
```

Balanced:

```powershell
$EMBED_QWEN3_06B_Q5 = "$MODELS_ROOT\majentik\Qwen3-Embedding-0.6B-GGUF-Q5_K_M\qwen3-emb-0.6b-Q5_K_M.gguf"
```

VRAM saver:

```powershell
$EMBED_QWEN3_06B_Q4 = "$MODELS_ROOT\andquant\Qwen3-Embedding-0.6B-Q4_K_M-GGUF\qwen3-embedding-0.6b-q4_k_m.gguf"
```

## Reranker Paths

Recommended:

```powershell
$RERANK_QWEN3_06B_Q4 = "$MODELS_ROOT\Voodisss\Qwen3-Reranker-0.6B-GGUF-llama_cpp\Qwen3-Reranker-0.6B-Q4_K_M.gguf"
```

Balanced upgrade:

```powershell
$RERANK_QWEN3_06B_Q5 = "$MODELS_ROOT\Voodisss\Qwen3-Reranker-0.6B-GGUF-llama_cpp\Qwen3-Reranker-0.6B-Q5_K_M.gguf"
```

Highest quality if VRAM allows:

```powershell
$RERANK_QWEN3_06B_Q8 = "$MODELS_ROOT\Voodisss\Qwen3-Reranker-0.6B-GGUF-llama_cpp\Qwen3-Reranker-0.6B.Q8_0.gguf"
```

Emergency VRAM:

```powershell
$RERANK_QWEN3_06B_Q3 = "$MODELS_ROOT\Voodisss\Qwen3-Reranker-0.6B-GGUF-llama_cpp\Qwen3-Reranker-0.6B-Q3_K_M.gguf"
```

## Recommended Assignments

Quality retrieval:

```powershell
$EMBED_MODEL = $EMBED_QWEN3_06B_Q8
$RERANK_MODEL = $RERANK_QWEN3_06B_Q4
```

Balanced retrieval:

```powershell
$EMBED_MODEL = $EMBED_QWEN3_06B_Q5
$RERANK_MODEL = $RERANK_QWEN3_06B_Q5
```

VRAM saver retrieval:

```powershell
$EMBED_MODEL = $EMBED_QWEN3_06B_Q4
$RERANK_MODEL = $RERANK_QWEN3_06B_Q3
```

Main generation:

```powershell
$MAIN_MODEL = $MAIN_QWEN35_2B_Q4
```

## Existence Check

Run this before starting `llama-server`:

```powershell
$requiredModels = @(
  $EMBED_MODEL,
  $RERANK_MODEL,
  $MAIN_MODEL
)

$requiredModels | ForEach-Object {
  if (-not (Test-Path $_)) {
    Write-Error "Missing model file: $_"
  } else {
    Write-Host "OK: $_"
  }
}
```

When running retrieval mode only, checking `$MAIN_MODEL` is optional. When running generation mode only, checking `$EMBED_MODEL` and `$RERANK_MODEL` is optional.
