---
name: runanywhere-ai
description: Integrate RunAnywhere on-device AI (LLMs, STT, TTS, voice agents, VLM) into applications. Use when implementing offline AI features, local LLM inference, on-device speech processing, privacy-first AI, vision language models, or when mentions of "RunanywhereAI", "on-device AI", "local inference", "offline AI", "GGUF models", "llama.cpp", "VLM", or "on-device vision" appear. Supports Swift (iOS/macOS), Kotlin (Android), Web (WebAssembly), React Native, and Flutter platforms.
---

# RunAnywhere AI Integration

Comprehensive guide for integrating RunAnywhere on-device AI into your applications across all supported platforms.

## What is RunAnywhere?

RunAnywhere enables privacy-first, on-device AI inference for:
- **LLM Text Generation** - Run LFM2, Llama, Mistral, Qwen, SmolLM locally via llama.cpp
- **Vision Language Models (VLM)** - On-device visual understanding with camera/image input (iOS/Web)
- **Speech-to-Text** - Whisper-based transcription
- **Text-to-Speech** - Neural voice synthesis via Piper
- **Voice Agent Pipeline** - Complete VAD → STT → LLM → TTS orchestration
- **Tool Calling & Structured Output** - Function calling and JSON schema-guided generation

All processing happens locally—no cloud, no latency, no data leaves the device.

## Workflow

### 1. Choose Your Platform

Select the platform-specific guide:

- **Swift (iOS/macOS)** → Read [swift.md](references/swift.md)
- **Kotlin (Android)** → Read [kotlin.md](references/kotlin.md)
- **Web (Browser)** → Read [web.md](references/web.md)
- **React Native** → Read [react-native.md](references/react-native.md)
- **Flutter** → Read [flutter.md](references/flutter.md)

Each guide contains complete installation, setup, and usage instructions.

### 2. Select Models

Choose appropriate models for your use case:

Read [models.md](references/models.md) for:
- Model size vs quality tradeoffs
- Device-specific recommendations
- Quantization level guidance
- Download URLs

**Quick recommendations:**
- **Lightweight** (< 1GB RAM): LFM2 350M, SmolLM2 360M, Qwen 0.5B
- **Balanced** (1-4GB RAM): LFM2 1.2B Tool, Llama 3.2 1B
- **High Quality** (4GB+ RAM): Llama 3.2 3B, Mistral 7B
- **Vision (VLM)**: LFM2-VL 450M (iOS/Web only)

### 3. Core Integration Pattern

All platforms follow the same three-step pattern:

```
1. Initialize SDK
   ↓
2. Download & Load Model
   ↓
3. Generate / Transcribe / Synthesize
```

Platform-specific implementation details are in each reference guide.

## Common Integration Tasks

### Integrate LLM into Swift iOS App

1. Read [swift.md](references/swift.md)
2. Add RunAnywhere via Swift Package Manager
3. Initialize SDK and register LlamaCPP module
4. Download and load a model (e.g., smollm2-360m)
5. Use `RunAnywhere.chat()` or `RunAnywhere.generate()` for inference
6. Optional: Use streaming for better UX

### Set Up Speech-to-Text in React Native

1. Read [react-native.md](references/react-native.md)
2. Install `@runanywhere/core` and `@runanywhere/onnx`
3. Register ONNX module and add Whisper model
4. Download and load STT model
5. Use `RunAnywhere.transcribe()` for audio transcription

### Build Voice Assistant on Android

1. Read [kotlin.md](references/kotlin.md)
2. Add RunAnywhere Kotlin SDK dependencies
3. Register LlamaCPP and ONNX modules
4. Download LLM, STT, and TTS models
5. Configure and start VoiceAgent with `RunAnywhere.startVoiceSession()`
6. Handle voice session events (listening, transcribed, responded, speaking)

### Deploy On-Device LLM to Web

1. Read [web.md](references/web.md)
2. Install `@runanywhere/web`, `@runanywhere/web-llamacpp`, and `@runanywhere/web-onnx` via npm
3. Configure bundler (Vite/Webpack) for WASM files
4. Set Cross-Origin headers for SharedArrayBuffer
5. Register `LlamaCPP` and `ONNX` backends
6. Register models with `RunAnywhere.registerModels()`
7. Generate with streaming via `TextGeneration.generateStream()`

### Add Vision Language Model (VLM) to Web App

1. Read [web.md](references/web.md)
2. Install 3-package Web SDK (`@runanywhere/web`, `@runanywhere/web-llamacpp`, `@runanywhere/web-onnx`)
3. Create a VLM Web Worker with `startVLMWorkerRuntime()`
4. Wire `VLMWorkerBridge` to `RunAnywhere.setVLMLoader()`
5. Use `VideoCapture` to capture camera frames
6. Process frames with `VLMWorkerBridge.shared.process(rgbPixels, width, height, prompt)`

## Key Concepts

### Quantization

Models are compressed using quantization:
- **Q4_0**: Smallest size, fastest, lower quality (~3.5 bits/weight)
- **Q5_K_M**: Balanced size and quality (~5.5 bits/weight)
- **Q8_0**: Largest size, best quality, slower (~8 bits/weight)

### Model Formats

- **LLM**: GGUF format (via llama.cpp) — `LLMFramework.LlamaCpp`
- **VLM**: GGUF format (model + mmproj files) — `LLMFramework.LlamaCpp`
- **STT**: ONNX format (Whisper models) — `LLMFramework.ONNX`
- **TTS**: ONNX format (Piper voices) — `LLMFramework.ONNX`
- **VAD**: ONNX format (Silero VAD v5) — `LLMFramework.ONNX`

### Memory Requirements

Rule of thumb: Device RAM should be 2× model size
- SmolLM2 360M (~400MB) → Need 800MB+ RAM
- Llama 3.2 1B (~1GB) → Need 2GB+ RAM
- Mistral 7B (~4GB) → Need 8GB+ RAM

## Error Handling Best Practices

### Common Errors Across All Platforms

**Model Not Found**
```
Solution: Download model first before loading
Check: Model exists at expected path
```

**Insufficient Memory**
```
Solution: Use smaller model or lower quantization
Check: Available device RAM vs model requirements
```

**Slow Generation**
```
Solution: Use streaming for better UX, lower quantization, or smaller model
Check: Tokens per second metric (target: 10+ tok/s)
```

**Download Fails**
```
Solution: Implement retry with exponential backoff
Check: Network connectivity, storage space
```

## Performance Tips

### Optimize Model Selection

- Web: Use Q4_0 quantization (browser memory limits)
- Mobile: Use Q4_K_M or Q5_K_M (balanced)
- Desktop: Use Q5_K_M or Q8_0 (best quality)

### Improve UX

- Use **streaming** to display tokens as they generate
- Show **progress bars** during model downloads
- Display **generation metrics** (tok/s) to users
- Implement **cancellation** for long-running operations

### Memory Management

- Unload models when not in use
- Check available memory before loading
- Use smaller models on low-memory devices
- Monitor memory usage during inference

## Debugging

### Enable Verbose Logging

All platforms support development mode with verbose logging:

**Swift:**
```swift
try RunAnywhere.initialize(environment: .development)
```

**Kotlin:**
```kotlin
RunAnywhere.initialize(environment = SDKEnvironment.DEVELOPMENT)
```

**Web:**
```typescript
await RunAnywhere.initialize({ environment: 'development', debug: true })
```

**React Native/Flutter:** Same pattern as above

### Platform-Specific Logs

**iOS:**
```bash
log stream --predicate 'subsystem CONTAINS "com.runanywhere"' --info --debug
```

**Android:**
```bash
adb logcat | grep "RunAnywhere"
```

**Web:**
```javascript
console.log()  // Standard browser console
```

## Additional Resources

### Official Documentation

- Website: [runanywhere.ai](https://runanywhere.ai)
- Docs: [docs.runanywhere.ai](https://docs.runanywhere.ai)
- GitHub: [github.com/RunanywhereAI/runanywhere-sdks](https://github.com/RunanywhereAI/runanywhere-sdks)
- Discord: [discord.gg/N359FBbDVd](https://discord.gg/N359FBbDVd)

### Example Apps

Each platform has a complete demo app:
- iOS: [examples/ios/RunAnywhereAI/](https://github.com/RunanywhereAI/runanywhere-sdks/tree/main/examples/ios/RunAnywhereAI)
- Android: [examples/android/RunAnywhereAI/](https://github.com/RunanywhereAI/runanywhere-sdks/tree/main/examples/android/RunAnywhereAI)
- Web: [web-starter-app](https://github.com/RunanywhereAI/web-starter-app) (Chat, Vision, Voice tabs)
- React Native: [examples/react-native/RunAnywhereAI/](https://github.com/RunanywhereAI/runanywhere-sdks/tree/main/examples/react-native/RunAnywhereAI)
- Flutter: [examples/flutter/RunAnywhereAI/](https://github.com/RunanywhereAI/runanywhere-sdks/tree/main/examples/flutter/RunAnywhereAI)

## Quick Reference

| Task | Command/Method | Platform |
|------|----------------|----------|
| Initialize SDK | `RunAnywhere.initialize()` | All |
| Register Backend | `LlamaCPP.register()` / `ONNX.register()` | Web |
| Register Models | `RunAnywhere.registerModels(models)` | Web |
| Download Model | `ModelManager.downloadModel(id)` | Web |
| Load Model | `ModelManager.loadModel(id)` | Web |
| Generate Text | `TextGeneration.generate(prompt)` | All |
| Stream Generation | `TextGeneration.generateStream(prompt)` | All |
| VLM Process | `VLMWorkerBridge.shared.process(rgb, w, h, prompt)` | iOS/Web |
| Capture Camera | `VideoCapture.captureFrame(dim)` | Web |
| Transcribe Audio | `STT.transcribe(audio)` | All |
| Synthesize Speech | `TTS.synthesize(text)` | All |
| Voice Agent | `VoicePipeline.start()` | All |

---

**Next Steps:**

1. Choose your platform and read the corresponding reference guide
2. Review [models.md](references/models.md) to select appropriate models
3. Follow the platform-specific setup instructions
4. Start with a simple text generation example
5. Explore advanced features (STT, TTS, voice agents) as needed
