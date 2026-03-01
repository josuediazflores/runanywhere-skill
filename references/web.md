# RunAnywhere Web SDK Reference

Complete guide for integrating RunAnywhere on-device AI into web applications via WebAssembly.

## Installation

The Web SDK uses a three-package architecture:

```bash
npm install @runanywhere/web @runanywhere/web-llamacpp @runanywhere/web-onnx
```

| Package | Purpose | Exports |
|---------|---------|---------|
| `@runanywhere/web` | Core TypeScript API (no WASM) | `RunAnywhere`, `ModelManager`, `ModelCategory`, `LLMFramework`, `SDKEnvironment`, `EventBus`, `VideoCapture`, `AudioCapture`, `AudioPlayback`, `OPFSStorage`, `detectCapabilities`, `CompactModelDef` |
| `@runanywhere/web-llamacpp` | LLM/VLM backend via llama.cpp WASM | `LlamaCPP`, `TextGeneration`, `VLMWorkerBridge`, `startVLMWorkerRuntime` |
| `@runanywhere/web-onnx` | STT/TTS/VAD backend via sherpa-onnx WASM | `ONNX`, `STT`, `TTS`, `VAD` |

## Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **Browser** | Chrome 96+ / Edge 96+ | Chrome 120+ / Edge 120+ |
| **WebAssembly** | Required | Required |
| **SharedArrayBuffer** | For multi-threaded WASM | Requires Cross-Origin Isolation headers |
| **WebGPU** | Optional (GPU acceleration) | Chrome 120+ |
| **RAM** | 2GB | 4GB+ for larger models |

## Setup

### Bundler Configuration

**Vite (recommended):**

```typescript
// vite.config.ts
import { defineConfig, Plugin } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';
import fs from 'fs';

// Custom plugin to copy WASM files from node_modules to dist
function copyWasmPlugin(): Plugin {
  return {
    name: 'copy-wasm',
    writeBundle() {
      const outDir = path.resolve(__dirname, 'dist/assets');
      fs.mkdirSync(outDir, { recursive: true });

      // Copy llama.cpp WASM files
      const llamaWasm = path.resolve(__dirname, 'node_modules/@runanywhere/web-llamacpp/wasm');
      for (const f of fs.readdirSync(llamaWasm)) {
        fs.copyFileSync(path.join(llamaWasm, f), path.join(outDir, f));
      }

      // Copy sherpa-onnx WASM files
      const onnxWasm = path.resolve(__dirname, 'node_modules/@runanywhere/web-onnx/wasm');
      for (const f of fs.readdirSync(onnxWasm)) {
        if (f === 'sherpa') {
          // Copy sherpa subdirectory
          const sherpaDir = path.join(outDir, 'sherpa');
          fs.mkdirSync(sherpaDir, { recursive: true });
          for (const sf of fs.readdirSync(path.join(onnxWasm, 'sherpa'))) {
            fs.copyFileSync(path.join(onnxWasm, 'sherpa', sf), path.join(sherpaDir, sf));
          }
        } else {
          fs.copyFileSync(path.join(onnxWasm, f), path.join(outDir, f));
        }
      }
    },
  };
}

export default defineConfig({
  plugins: [react(), copyWasmPlugin()],
  assetsInclude: ['**/*.wasm'],
  optimizeDeps: {
    exclude: ['@runanywhere/web-llamacpp', '@runanywhere/web-onnx'],
  },
  worker: { format: 'es' },
  server: {
    headers: {
      'Cross-Origin-Opener-Policy': 'same-origin',
      'Cross-Origin-Embedder-Policy': 'credentialless',
    },
  },
});
```

**Webpack:**

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      { test: /\.wasm$/, type: 'asset/resource' },
    ],
  },
};
```

### Cross-Origin Isolation Headers

Required for SharedArrayBuffer support (multi-threaded WASM):

```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: credentialless
```

Without these headers, the SDK falls back to single-threaded mode with degraded performance.

## Quick Start

### 1. Initialize SDK

```typescript
import {
  RunAnywhere,
  SDKEnvironment,
  ModelManager,
  ModelCategory,
  LLMFramework,
  type CompactModelDef,
} from '@runanywhere/web';

import { LlamaCPP } from '@runanywhere/web-llamacpp';
import { ONNX } from '@runanywhere/web-onnx';

// Step 1: Initialize core SDK
await RunAnywhere.initialize({
  environment: SDKEnvironment.Development,
  debug: true,
});

// Step 2: Register WASM backends
await LlamaCPP.register();
await ONNX.register();

// Step 3: Register model catalog
const MODELS: CompactModelDef[] = [
  {
    id: 'lfm2-350m-q4_k_m',
    name: 'LFM2 350M Q4_K_M',
    repo: 'LiquidAI/LFM2-350M-GGUF',
    files: ['LFM2-350M-Q4_K_M.gguf'],
    framework: LLMFramework.LlamaCpp,
    modality: ModelCategory.Language,
    memoryRequirement: 250_000_000,
  },
];
RunAnywhere.registerModels(MODELS);
```

### 2. Text Generation (LLM)

```typescript
import { TextGeneration } from '@runanywhere/web-llamacpp';

// Generate (non-streaming)
const result = await TextGeneration.generate('Explain quantum computing briefly.');
console.log(result.text);
console.log(`${result.tokensUsed} tokens in ${result.latencyMs}ms`);
console.log(`Speed: ${result.tokensPerSecond} tok/sec`);

// Stream tokens
const { stream, result: resultPromise, cancel } = TextGeneration.generateStream(
  'Write a haiku about code.',
  { maxTokens: 512, temperature: 0.7 }
);

for await (const token of stream) {
  process.stdout.write(token); // Real-time output
}

const finalResult = await resultPromise;
console.log(`${finalResult.tokensPerSecond} tok/sec`);

// Cancel mid-stream if needed
// cancel();
```

### 3. Speech-to-Text (STT)

```typescript
import { STT } from '@runanywhere/web-onnx';

await STT.loadModel({
  modelId: 'whisper-tiny',
  type: STTModelType.Whisper,
  modelFiles: {
    encoder: '/models/encoder.onnx',
    decoder: '/models/decoder.onnx',
    tokens: '/models/tokens.txt'
  },
  sampleRate: 16000,
});

const result = await STT.transcribe(audioFloat32Array);
console.log(result.text);
```

### 4. Text-to-Speech (TTS)

```typescript
import { TTS } from '@runanywhere/web-onnx';

await TTS.loadVoice({
  modelId: 'piper-us-english',
  modelPath: '/models/piper-model.onnx',
  tokensPath: '/models/tokens.txt'
});

const { audioData, sampleRate } = await TTS.synthesize('Hello from RunAnywhere!');
// audioData is Float32Array PCM audio
```

## Advanced Usage

### Vision Language Models (VLM)

VLM enables on-device visual understanding in the browser. It runs in a dedicated Web Worker to keep the UI responsive.

**Step 1: Create VLM Web Worker**

```typescript
// src/workers/vlm-worker.ts
import { startVLMWorkerRuntime } from '@runanywhere/web-llamacpp';
startVLMWorkerRuntime();
```

**Step 2: Wire up VLM during SDK initialization**

```typescript
import { RunAnywhere, SDKEnvironment, ModelCategory, LLMFramework, type CompactModelDef } from '@runanywhere/web';
import { LlamaCPP, VLMWorkerBridge } from '@runanywhere/web-llamacpp';

// Import worker URL (Vite syntax)
import vlmWorkerUrl from './workers/vlm-worker?worker&url';

await RunAnywhere.initialize({ environment: SDKEnvironment.Development });
await LlamaCPP.register();

// Register VLM model (requires model + mmproj files)
RunAnywhere.registerModels([
  {
    id: 'lfm2-vl-450m-q4_0',
    name: 'LFM2-VL 450M Q4_0',
    repo: 'runanywhere/LFM2-VL-450M-GGUF',
    files: ['LFM2-VL-450M-Q4_0.gguf', 'mmproj-LFM2-VL-450M-Q8_0.gguf'],
    framework: LLMFramework.LlamaCpp,
    modality: ModelCategory.Multimodal,
    memoryRequirement: 500_000_000,
  },
]);

// Wire VLM worker bridge
VLMWorkerBridge.shared.workerUrl = vlmWorkerUrl;
RunAnywhere.setVLMLoader({
  get isInitialized() { return VLMWorkerBridge.shared.isInitialized; },
  init: () => VLMWorkerBridge.shared.init(),
  loadModel: (params) => VLMWorkerBridge.shared.loadModel(params),
  unloadModel: () => VLMWorkerBridge.shared.unloadModel(),
});
```

**Step 3: Capture camera frames and process with VLM**

```typescript
import { VideoCapture } from '@runanywhere/web';
import { VLMWorkerBridge } from '@runanywhere/web-llamacpp';

// Start camera
const camera = new VideoCapture({ facingMode: 'environment' });
await camera.start();

// Capture a frame (256px — CLIP resizes internally)
const frame = camera.captureFrame(256);

// Process with VLM
const result = await VLMWorkerBridge.shared.process(
  frame.rgbPixels,     // RGB pixel data
  frame.width,
  frame.height,
  'What do you see in this image? Describe the scene.',
  { maxTokens: 80, temperature: 0.7 }
);
console.log(result.text);

// Stop camera when done
camera.stop();
```

**Live mode (continuous VLM processing):**

```typescript
// Poll every 2.5 seconds for live descriptions
const interval = setInterval(async () => {
  if (!camera.isCapturing) return;
  const frame = camera.captureFrame(256);
  const result = await VLMWorkerBridge.shared.process(
    frame.rgbPixels, frame.width, frame.height,
    'Briefly describe what you see.',
    { maxTokens: 30, temperature: 0.7 }
  );
  console.log(result.text);
}, 2500);
```

### Voice Activity Detection (VAD)

```typescript
import { VAD } from '@runanywhere/web-onnx';

await VAD.initialize();

// Process audio samples
const result = VAD.processSamples(audioFloat32Array);

// Get speech segments
const segment = VAD.popSpeechSegment();
if (segment) {
  console.log('Speech detected:', segment);
}

// Callback-based detection
VAD.onSpeechActivity((isSpeaking) => {
  console.log(isSpeaking ? 'Speech started' : 'Speech ended');
});
```

### Tool Calling

```typescript
import { TextGeneration } from '@runanywhere/web-llamacpp';

const tools = [
  {
    name: 'get_weather',
    description: 'Get current weather for a location',
    parameters: {
      type: 'object',
      properties: {
        location: { type: 'string', description: 'City name' }
      },
      required: ['location']
    }
  }
];

const result = await TextGeneration.generateWithTools(
  'What\'s the weather in Paris?',
  tools
);

if (result.toolCall) {
  console.log('Tool:', result.toolCall.name);
  console.log('Args:', result.toolCall.arguments);
}
```

**Note:** Use a tool-calling model like LFM2 1.2B Tool for best results.

### Structured Output

```typescript
import { TextGeneration } from '@runanywhere/web-llamacpp';

const schema = {
  type: 'object',
  properties: {
    name: { type: 'string' },
    age: { type: 'number' },
    hobbies: {
      type: 'array',
      items: { type: 'string' }
    }
  },
  required: ['name', 'age']
};

const result = await TextGeneration.generateStructured(
  'Generate a profile for a person who likes coding',
  schema
);
const data = JSON.parse(result.text);
console.log(data);  // { name: "...", age: ..., hobbies: [...] }
```

### Embeddings

```typescript
import { Embeddings } from '@runanywhere/web';

await Embeddings.loadModel('/models/all-minilm-l6-v2.gguf');

// Single text embedding
const embedding = await Embeddings.embed('Hello world');
console.log(embedding);  // Float32Array

// Batch embeddings
const embeddings = await Embeddings.embedBatch([
  'First text',
  'Second text',
  'Third text'
]);
```

### Voice Pipeline

Complete VAD → STT → LLM → TTS orchestration:

```typescript
import { VoicePipeline } from '@runanywhere/web';

await VoicePipeline.initialize({
  vadModel: '/models/silero-vad.onnx',
  sttModel: 'whisper-tiny',
  llmModel: 'lfm2-350m',
  ttsModel: 'piper-us-english'
});

// Callbacks
VoicePipeline.onTranscription = (text) => {
  console.log('User said:', text);
};

VoicePipeline.onResponse = (text) => {
  console.log('AI responded:', text);
};

VoicePipeline.onAudio = (audioData) => {
  // Play audio
};

// Start listening
await VoicePipeline.start();

// Stop
await VoicePipeline.stop();
```

## Model Management

### Model Registration (CompactModelDef)

Register models with their download source, framework, and memory requirements:

```typescript
import { ModelCategory, LLMFramework, type CompactModelDef } from '@runanywhere/web';

const MODELS: CompactModelDef[] = [
  // GGUF model from Hugging Face repo
  {
    id: 'lfm2-350m-q4_k_m',
    name: 'LFM2 350M Q4_K_M',
    repo: 'LiquidAI/LFM2-350M-GGUF',           // HF repo
    files: ['LFM2-350M-Q4_K_M.gguf'],           // Files to download
    framework: LLMFramework.LlamaCpp,
    modality: ModelCategory.Language,
    memoryRequirement: 250_000_000,              // bytes
  },
  // VLM model (requires model + mmproj)
  {
    id: 'lfm2-vl-450m-q4_0',
    name: 'LFM2-VL 450M Q4_0',
    repo: 'runanywhere/LFM2-VL-450M-GGUF',
    files: ['LFM2-VL-450M-Q4_0.gguf', 'mmproj-LFM2-VL-450M-Q8_0.gguf'],
    framework: LLMFramework.LlamaCpp,
    modality: ModelCategory.Multimodal,
    memoryRequirement: 500_000_000,
  },
  // Tool-calling model
  {
    id: 'lfm2-1.2b-tool-q4_k_m',
    name: 'LFM2 1.2B Tool Q4_K_M',
    repo: 'LiquidAI/LFM2-1.2B-Tool-GGUF',
    files: ['LFM2-1.2B-Tool-Q4_K_M.gguf'],
    framework: LLMFramework.LlamaCpp,
    modality: ModelCategory.Language,
    memoryRequirement: 800_000_000,
  },
  // ONNX archive from direct URL
  {
    id: 'sherpa-onnx-whisper-tiny.en',
    name: 'Whisper Tiny English (ONNX)',
    url: 'https://huggingface.co/runanywhere/sherpa-onnx-whisper-tiny.en/resolve/main/sherpa-onnx-whisper-tiny.en.tar.gz',
    framework: LLMFramework.ONNX,
    modality: ModelCategory.SpeechRecognition,
    memoryRequirement: 105_000_000,
    artifactType: 'archive',
  },
  // TTS archive
  {
    id: 'vits-piper-en_US-lessac-medium',
    name: 'Piper TTS US English (Lessac)',
    url: 'https://huggingface.co/runanywhere/vits-piper-en_US-lessac-medium/resolve/main/vits-piper-en_US-lessac-medium.tar.gz',
    framework: LLMFramework.ONNX,
    modality: ModelCategory.SpeechSynthesis,
    memoryRequirement: 65_000_000,
    artifactType: 'archive',
  },
  // Single ONNX file
  {
    id: 'silero-vad-v5',
    name: 'Silero VAD v5',
    url: 'https://huggingface.co/runanywhere/silero-vad-v5/resolve/main/silero_vad.onnx',
    files: ['silero_vad.onnx'],
    framework: LLMFramework.ONNX,
    modality: ModelCategory.Audio,
    memoryRequirement: 5_000_000,
  },
];

RunAnywhere.registerModels(MODELS);
```

### Download & Load Models

```typescript
import { ModelManager, EventBus } from '@runanywhere/web';

// Track download progress
EventBus.shared.on('model.downloadProgress', (evt) => {
  console.log(`${evt.modelId}: ${(evt.progress * 100).toFixed(1)}%`);
});

// Download model
await ModelManager.downloadModel('lfm2-350m-q4_k_m');

// Load model (coexist: true allows multiple model categories loaded simultaneously)
await ModelManager.loadModel('lfm2-350m-q4_k_m', { coexist: false });

// Check loaded model
const loaded = ModelManager.getLoadedModel(ModelCategory.Language);
console.log('Loaded:', loaded?.id);

// List all registered models
const allModels = ModelManager.getModels();
```

### React Hook for Model Loading

```typescript
import { useState, useCallback } from 'react';
import { ModelManager, ModelCategory, EventBus } from '@runanywhere/web';

type LoaderState = 'idle' | 'downloading' | 'loading' | 'ready' | 'error';

function useModelLoader(category: ModelCategory, coexist = false) {
  const [state, setState] = useState<LoaderState>('idle');
  const [progress, setProgress] = useState(0);
  const [error, setError] = useState<string | null>(null);

  const ensure = useCallback(async () => {
    // Already loaded?
    if (ModelManager.getLoadedModel(category)) {
      setState('ready');
      return true;
    }

    const model = ModelManager.getModels().find(m => m.modality === category);
    if (!model) { setError('No model registered'); return false; }

    try {
      if (model.status !== 'downloaded') {
        setState('downloading');
        EventBus.shared.on('model.downloadProgress', (evt) => {
          if (evt.modelId === model.id) setProgress(evt.progress);
        });
        await ModelManager.downloadModel(model.id);
      }

      setState('loading');
      await ModelManager.loadModel(model.id, { coexist });
      setState('ready');
      return true;
    } catch (e) {
      setState('error');
      setError(String(e));
      return false;
    }
  }, [category, coexist]);

  return { state, progress, error, ensure };
}
```

### Persistent Storage (OPFS)

Models are automatically stored in Origin Private File System:

```typescript
import { OPFSStorage } from '@runanywhere/web';

// Models persist across sessions in OPFS automatically
// Use ModelManager for high-level management
// Use OPFSStorage for low-level access if needed
```

## Configuration

### Generation Options

```typescript
const options = {
  maxTokens: 512,
  temperature: 0.7,
  topP: 1.0,
  topK: 40,
  stopSequences: ['END'],
  systemPrompt: 'You are a helpful assistant.',
};

const result = await TextGeneration.generate(prompt, options);
```

### Performance Optimization

```typescript
// Use Web Workers for non-blocking VLM inference (required for VLM)
// TextGeneration already runs in the main WASM thread

// Enable WebGPU acceleration (if available)
// LlamaCPP auto-detects WebGPU at registration time
const mode = LlamaCPP.accelerationMode; // 'webgpu' | 'wasm' | null
```

## Browser Compatibility Check

```typescript
import { detectCapabilities } from '@runanywhere/web';

const caps = await detectCapabilities();

console.log('Cross-Origin Isolated:', caps.isCrossOriginIsolated);
console.log('SharedArrayBuffer:', caps.hasSharedArrayBuffer);
console.log('WebGPU:', caps.hasWebGPU);
console.log('OPFS:', caps.hasOPFS);

if (!caps.isCrossOriginIsolated) {
  console.warn('Add COOP/COEP headers for multi-threaded performance');
}
```

## Error Handling

```typescript
import { SDKError } from '@runanywhere/web';

try {
  await ModelManager.loadModel(modelId);
} catch (error) {
  if (error instanceof SDKError) {
    console.error('SDK Error:', error.message);
  } else if (error.message.includes('quota')) {
    console.error('Storage quota exceeded');
  } else if (error.message.includes('SharedArrayBuffer')) {
    console.error('Missing Cross-Origin headers');
  } else {
    console.error('Model load failed:', error);
  }
}
```

## Debugging

### Enable Logging

```typescript
import { SDKLogger, LogLevel } from '@runanywhere/web';

SDKLogger.enabled = true;
SDKLogger.level = LogLevel.Debug;

await RunAnywhere.initialize({
  environment: SDKEnvironment.Development,
  debug: true,
});
```

### Performance Monitoring

```typescript
const { stream, result: resultPromise } = TextGeneration.generateStream(prompt);
for await (const token of stream) { /* display */ }
const result = await resultPromise;

console.log('Tokens/sec:', result.tokensPerSecond);
console.log('Latency:', result.latencyMs);
console.log('Tokens used:', result.tokensUsed);
```

## Common Issues

### SharedArrayBuffer Not Available

Add Cross-Origin headers to your server:

```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: credentialless
```

In Vite, add to `server.headers` in `vite.config.ts`.

### WASM Files Not Found

Ensure WASM files are copied to your output directory. Use the `copyWasmPlugin()` pattern shown in Setup, and exclude the backend packages from pre-bundling:

```typescript
optimizeDeps: {
  exclude: ['@runanywhere/web-llamacpp', '@runanywhere/web-onnx'],
},
```

### Model Load Fails

```typescript
// Check storage quota
const estimate = await navigator.storage.estimate();
console.log('Used:', estimate.usage);
console.log('Quota:', estimate.quota);

if (estimate.usage / estimate.quota > 0.9) {
  // Clear old models via ModelManager
}
```

### Slow Generation

```typescript
// Use smaller quantized models (Q4_0 for web)
// Check if WebGPU is active:
console.log('Acceleration:', LlamaCPP.accelerationMode);

// Use streaming for better perceived performance
const { stream } = TextGeneration.generateStream(prompt);
for await (const token of stream) {
  // Display immediately
}
```

### VLM Worker Not Initializing

Ensure the worker is imported with the correct Vite syntax and `worker.format` is set:

```typescript
// vite.config.ts
worker: { format: 'es' }

// In your code
import vlmWorkerUrl from './workers/vlm-worker?worker&url';
VLMWorkerBridge.shared.workerUrl = vlmWorkerUrl;
```

## Resources

- Web SDK Source: [sdk/runanywhere-web/](https://github.com/RunanywhereAI/runanywhere-sdks/tree/main/sdk/runanywhere-web)
- Web Starter App: [web-starter-app](https://github.com/RunanywhereAI/web-starter-app) (Chat, Vision, Voice demo)
- Discord: [discord.gg/N359FBbDVd](https://discord.gg/N359FBbDVd)
