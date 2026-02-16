# RunAnywhere Model Selection Guide

Comprehensive guide for choosing the right models for your use case.

## LLM Models (Text Generation)

RunAnywhere supports GGUF-format models via llama.cpp. Choose based on device memory and quality requirements.

### Recommended Models

| Model | Size | RAM Required | Quality | Speed | Use Case |
|-------|------|--------------|---------|-------|----------|
| **SmolLM2 360M** | ~400MB | 500MB | Basic | Very Fast | Lightweight apps, quick responses |
| **Qwen 2.5 0.5B** | ~500MB | 600MB | Good | Fast | Multilingual, balanced performance |
| **Llama 3.2 1B** | ~1GB | 1.2GB | Good | Fast | General purpose, good quality |
| **Qwen 2.5 1.5B** | ~1.5GB | 2GB | Very Good | Medium | Advanced reasoning |
| **Llama 3.2 3B** | ~2.5GB | 3GB | Excellent | Medium | High-quality responses |
| **Mistral 7B Q4** | ~4GB | 5GB | Excellent | Slower | Best quality, high-end devices |

### Quantization Levels

Model sizes vary based on quantization (compression):

| Quantization | Bits/Weight | Size Impact | Quality | Speed |
|--------------|-------------|-------------|---------|-------|
| **Q4_0** | ~3.5 | Smallest | Lower | Fastest |
| **Q4_K_M** | ~4.5 | Small | Good | Fast |
| **Q5_0** | ~4.5 | Medium | Better | Medium |
| **Q5_K_M** | ~5.5 | Medium | Better | Medium |
| **Q8_0** | ~8 | Large | Best | Slower |

**Recommendation:** Use Q4_K_M or Q5_K_M for best balance of quality and size.

### Model Sources

**Hugging Face:**

```
https://huggingface.co/<model-repo>/resolve/main/<model-file>.gguf
```

Common repositories:
- `bartowski/` - High-quality GGUF conversions
- `TheBloke/` - Popular model conversions
- `prithivMLmods/` - SmolLM variants
- `Qwen/` - Official Qwen models

**Example URLs:**

```
# SmolLM2 360M Q8_0
https://huggingface.co/prithivMLmods/SmolLM2-360M-GGUF/resolve/main/SmolLM2-360M.Q8_0.gguf

# Qwen 2.5 0.5B Q4_K_M
https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct-GGUF/resolve/main/qwen2.5-0.5b-instruct-q4_k_m.gguf

# Llama 3.2 1B Q5_K_M
https://huggingface.co/bartowski/Llama-3.2-1B-Instruct-GGUF/resolve/main/Llama-3.2-1B-Instruct-Q5_K_M.gguf

# Mistral 7B Q4_K_M
https://huggingface.co/bartowski/Mistral-7B-Instruct-v0.3-GGUF/resolve/main/Mistral-7B-Instruct-v0.3-Q4_K_M.gguf
```

### Device-Specific Recommendations

#### Mobile (< 4GB RAM)
- SmolLM2 360M Q8_0 (~400MB)
- Qwen 2.5 0.5B Q4_K_M (~500MB)
- Llama 3.2 1B Q4_K_M (~1GB)

#### Mid-Range (4-8GB RAM)
- Llama 3.2 1B Q5_K_M (~1GB)
- Qwen 2.5 1.5B Q5_K_M (~1.5GB)
- Llama 3.2 3B Q4_K_M (~2.5GB)

#### High-End (8GB+ RAM)
- Llama 3.2 3B Q5_K_M (~3GB)
- Mistral 7B Q4_K_M (~4GB)
- Mistral 7B Q5_K_M (~5GB)

#### Web Browsers
- Qwen 2.5 0.5B Q4_0 (~300MB)
- SmolLM2 360M Q4_0 (~250MB)
- Llama 3.2 1B Q4_0 (~700MB)

**Note:** Web has stricter memory constraints due to browser limitations.

## Speech-to-Text Models

### Whisper Models (ONNX)

| Model | Size | Languages | Accuracy | Speed | Use Case |
|-------|------|-----------|----------|-------|----------|
| **Whisper Tiny** | ~75MB | English | Good | Very Fast | Real-time transcription |
| **Whisper Base** | ~150MB | Multilingual | Better | Fast | Balanced performance |
| **Whisper Small** | ~500MB | Multilingual | Very Good | Medium | High-quality transcription |
| **Whisper Medium** | ~1.5GB | Multilingual | Excellent | Slower | Best accuracy |

**Recommendation:** Use Whisper Tiny for real-time, Whisper Base for batch transcription.

### Model Downloads

**Sherpa-ONNX (Pre-packaged):**

```
https://github.com/RunanywhereAI/sherpa-onnx/releases/download/runanywhere-models-v1/sherpa-onnx-whisper-tiny.en.tar.gz
https://github.com/RunanywhereAI/sherpa-onnx/releases/download/runanywhere-models-v1/sherpa-onnx-whisper-base.tar.gz
```

These are pre-packaged archives with all required files (encoder, decoder, tokens).

## Text-to-Speech Models

### Piper TTS (ONNX)

| Voice | Size | Language | Quality | Speed | Use Case |
|-------|------|----------|---------|-------|----------|
| **Piper US English** | ~65MB | English (US) | Good | Fast | General purpose |
| **Piper UK English** | ~65MB | English (UK) | Good | Fast | British accent |
| **Piper Amy** | ~35MB | English (US) | Basic | Very Fast | Low-resource devices |
| **Piper Ryan** | ~90MB | English (US) | Excellent | Medium | High-quality voice |

### Model Downloads

**Sherpa-ONNX (Pre-packaged):**

```
https://github.com/RunanywhereAI/sherpa-onnx/releases/download/runanywhere-models-v1/piper-en-us.tar.gz
https://github.com/RunanywhereAI/sherpa-onnx/releases/download/runanywhere-models-v1/piper-en-gb.tar.gz
```

## Voice Activity Detection

### Silero VAD

| Model | Size | Accuracy | Speed | Use Case |
|-------|------|----------|-------|----------|
| **Silero VAD v4** | ~1.5MB | Excellent | Very Fast | Real-time speech detection |

**Download:**

```
https://github.com/snakers4/silero-vad/raw/master/files/silero_vad.onnx
```

## Vision Language Models (VLM)

### Supported Models

| Model | Size | Capabilities | Use Case |
|-------|------|--------------|----------|
| **Qwen2-VL 2B Q4** | ~2GB | Image + Text | Visual understanding |
| **LLaVA 1.5 7B Q4** | ~4GB | Image + Text | High-quality vision |

**Platforms:** iOS (Metal), Web (WebGPU)

**Note:** Not available on Android, React Native, or Flutter yet.

## Model Selection Decision Tree

```
Start Here
    |
    ├─ Need multilingual support?
    │   ├─ Yes → Qwen 2.5 (0.5B/1.5B)
    │   └─ No  → Continue
    │
    ├─ Available RAM?
    │   ├─ < 1GB   → SmolLM2 360M or Qwen 0.5B
    │   ├─ 1-4GB   → Llama 3.2 1B
    │   ├─ 4-8GB   → Llama 3.2 3B
    │   └─ > 8GB   → Mistral 7B
    │
    ├─ Platform?
    │   ├─ Web     → Use Q4_0 quantization (smallest)
    │   ├─ Mobile  → Use Q4_K_M or Q5_K_M (balanced)
    │   └─ Desktop → Use Q5_K_M or Q8_0 (best quality)
    │
    └─ Use Case?
        ├─ Quick responses    → SmolLM2, Qwen 0.5B
        ├─ Balanced           → Llama 3.2 1B/3B
        ├─ Best quality       → Mistral 7B
        └─ Reasoning/Thinking → Qwen 2.5 1.5B+
```

## Performance Optimization

### Model Loading Time

Smaller models load faster:
- SmolLM2 360M: ~0.5-1s
- Llama 3.2 1B: ~1-2s
- Llama 3.2 3B: ~2-4s
- Mistral 7B: ~5-10s

### Token Generation Speed

Higher quantization = faster generation:
- Q4_0: ~15-30 tok/s (mobile), ~30-50 tok/s (desktop)
- Q5_K_M: ~10-25 tok/s (mobile), ~25-40 tok/s (desktop)
- Q8_0: ~8-20 tok/s (mobile), ~20-35 tok/s (desktop)

**Note:** Speeds vary by device. Apple Silicon (M1/M2/M3) and modern Android chips are fastest.

### Memory Usage

Model memory = File size × 1.5-2.0 (runtime overhead)

Examples:
- SmolLM2 360M (~400MB) → ~600-800MB RAM
- Llama 3.2 1B (~1GB) → ~1.5-2GB RAM
- Mistral 7B (~4GB) → ~6-8GB RAM

## Testing Models

### Quick Test Prompts

```
"What is 2+2?"
"Explain quantum computing in one sentence."
"Write a haiku about code."
"Translate 'hello' to Spanish."
"What is the capital of France?"
```

### Performance Metrics

Monitor these metrics:
- **Tokens per second**: Higher is better (target: 10+ tok/s)
- **Latency (first token)**: Lower is better (target: < 1s)
- **Memory usage**: Should be 1.5-2× model size
- **Load time**: Smaller models load faster

### Quality Assessment

Test for:
- **Accuracy**: Correct factual responses
- **Coherence**: Logical flow and consistency
- **Instruction following**: Obeys system prompts
- **Language quality**: Grammar and fluency

## Common Model Issues

### "Insufficient memory" Error

**Solution:** Use smaller model or lower quantization:
- Mistral 7B Q8_0 (5GB) → Mistral 7B Q4_K_M (4GB)
- Llama 3.2 3B Q5_K_M (3GB) → Llama 3.2 1B Q5_K_M (1GB)
- Llama 3.2 1B Q8_0 (1.2GB) → Qwen 0.5B Q4_K_M (500MB)

### Slow Generation

**Solutions:**
1. Use lower quantization (Q4_0 vs Q8_0)
2. Use smaller model
3. Enable GPU acceleration (Metal on iOS, WebGPU on Web)
4. Use streaming for better UX

### Poor Quality Responses

**Solutions:**
1. Use higher quantization (Q5_K_M or Q8_0)
2. Use larger model (1B → 3B → 7B)
3. Adjust generation parameters:
   - Lower temperature (0.7 → 0.5) for more focused responses
   - Add better system prompt
   - Increase max tokens if responses are cut off

### Model Won't Load

**Solutions:**
1. Check file integrity (re-download if corrupted)
2. Verify model format (must be GGUF for LLM)
3. Ensure sufficient storage space
4. Check platform compatibility (some models iOS/Web only)

## Resources

- Hugging Face Model Hub: [huggingface.co/models](https://huggingface.co/models?pipeline_tag=text-generation&sort=trending)
- GGUF Models Search: [huggingface.co/models?library=gguf](https://huggingface.co/models?library=gguf)
- Sherpa-ONNX Models: [k2-fsa.github.io/sherpa/onnx](https://k2-fsa.github.io/sherpa/onnx/)
- Model Benchmarks: [docs.runanywhere.ai/benchmarks](https://docs.runanywhere.ai/benchmarks)
