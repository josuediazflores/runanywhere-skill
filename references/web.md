# RunAnywhere Web SDK Reference

Complete guide for integrating RunAnywhere on-device AI into web applications via WebAssembly.

## Installation

```bash
npm install @runanywhere/web
```

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

**Vite:**

```typescript
// vite.config.ts
export default defineConfig({
  assetsInclude: ['**/*.wasm'],
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

## Quick Start

### 1. Initialize SDK

```typescript
import { RunAnywhere } from '@runanywhere/web';

await RunAnywhere.initialize({
  environment: 'development',
  debug: true
});
```

### 2. Text Generation (LLM)

```typescript
import { TextGeneration } from '@runanywhere/web';

// Load a GGUF model
await TextGeneration.loadModel(
  '/models/qwen2.5-0.5b-instruct-q4_0.gguf',
  'qwen2.5-0.5b'
);

// Generate
const result = await TextGeneration.generate('Explain quantum computing briefly.');
console.log(result.text);

// Stream tokens
for await (const token of TextGeneration.generateStream('Write a haiku about code.')) {
  process.stdout.write(token);
}
```

### 3. Speech-to-Text (STT)

```typescript
import { STT, STTModelType } from '@runanywhere/web';

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
import { TTS } from '@runanywhere/web';

await TTS.loadModel({
  modelId: 'piper-us-english',
  modelPath: '/models/piper-model.onnx',
  tokensPath: '/models/tokens.txt'
});

const audioData = await TTS.synthesize('Hello from RunAnywhere!');
// audioData is Float32Array PCM audio
```

## Advanced Usage

### Voice Activity Detection (VAD)

```typescript
import { VAD } from '@runanywhere/web';

await VAD.loadModel('/models/silero-vad.onnx');

// Detect speech in audio
const result = await VAD.detect(audioFloat32Array);
console.log('Speech detected:', result.hasSpeech);

// Callback-based detection
VAD.onSpeechStart = () => console.log('Speech started');
VAD.onSpeechEnd = () => console.log('Speech ended');
```

### Vision Language Models (VLM)

```typescript
import { VLM } from '@runanywhere/web';

// Load VLM model
await VLM.loadModel('/models/qwen2-vl-2b.gguf');

// Generate from image + text
const result = await VLM.generate({
  prompt: 'What do you see in this image?',
  image: imageRGBData  // RGB pixel data
});
console.log(result.text);
```

### Tool Calling

```typescript
import { TextGeneration } from '@runanywhere/web';

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

### Structured Output

```typescript
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
  llmModel: 'qwen2.5-0.5b',
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

### Download Models

```typescript
import { ModelDownloader } from '@runanywhere/web';

await ModelDownloader.download(
  'https://huggingface.co/.../model.gguf',
  'my-model',
  (progress) => {
    console.log(`${(progress * 100).toFixed(1)}%`);
  }
);
```

### Persistent Storage (OPFS)

Models are automatically stored in Origin Private File System:

```typescript
import { ModelStorage } from '@runanywhere/web';

// List stored models
const models = await ModelStorage.list();

// Get model path
const path = await ModelStorage.getPath('my-model');

// Delete model
await ModelStorage.delete('my-model');

// Clear all models
await ModelStorage.clear();
```

## Configuration

### Generation Options

```typescript
const options = {
  maxTokens: 100,
  temperature: 0.8,
  topP: 1.0,
  topK: 40,
  stopSequences: ['END'],
  systemPrompt: 'You are a helpful assistant.',
  streamingEnabled: true
};

const result = await TextGeneration.generate(prompt, options);
```

### Performance Optimization

```typescript
// Use Web Workers for non-blocking inference
await TextGeneration.loadModel(modelPath, modelId, {
  useWorker: true  // Runs in dedicated Web Worker
});

// Enable WebGPU acceleration (if available)
await TextGeneration.loadModel(modelPath, modelId, {
  useGPU: true  // Falls back to WASM if unavailable
});
```

## Error Handling

```typescript
try {
  await TextGeneration.loadModel(modelPath, modelId);
} catch (error) {
  if (error.message.includes('quota')) {
    console.error('Storage quota exceeded');
  } else if (error.message.includes('SharedArrayBuffer')) {
    console.error('Missing Cross-Origin headers');
  } else {
    console.error('Model load failed:', error);
  }
}
```

## Browser Compatibility Check

```typescript
import { BrowserCapabilities } from '@runanywhere/web';

const caps = await BrowserCapabilities.check();

console.log('WebAssembly:', caps.hasWasm);
console.log('SharedArrayBuffer:', caps.hasSharedArrayBuffer);
console.log('WebGPU:', caps.hasWebGPU);
console.log('OPFS:', caps.hasOPFS);

if (!caps.hasWasm) {
  throw new Error('Browser not supported');
}
```

## Debugging

### Enable Logging

```typescript
await RunAnywhere.initialize({
  environment: 'development',
  debug: true,
  logLevel: 'verbose'
});
```

### Performance Monitoring

```typescript
const result = await TextGeneration.generate(prompt);
console.log('Tokens/sec:', result.tokensPerSecond);
console.log('Latency:', result.latencyMs);
console.log('Memory:', result.memoryUsed);
```

## Common Issues

### SharedArrayBuffer Not Available

Add Cross-Origin headers:

```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: credentialless
```

### Model Load Fails

```typescript
// Check storage quota
const estimate = await navigator.storage.estimate();
console.log('Used:', estimate.usage);
console.log('Quota:', estimate.quota);

if (estimate.usage / estimate.quota > 0.9) {
  // Clear old models
  await ModelStorage.clear();
}
```

### Slow Generation

```typescript
// Use smaller quantized models
// q4_0: ~3.5 bits per weight (fastest, lowest quality)
// q5_0: ~4.5 bits per weight (balanced)
// q8_0: ~8 bits per weight (slower, higher quality)

// Enable GPU acceleration
await TextGeneration.loadModel(modelPath, modelId, {
  useGPU: true
});

// Use streaming for better UX
for await (const token of TextGeneration.generateStream(prompt)) {
  // Display immediately
}
```

## Resources

- Source Code: [sdk/runanywhere-web/](https://github.com/RunanywhereAI/runanywhere-sdks/tree/main/sdk/runanywhere-web)
- Example App: [examples/web/RunAnywhereAI/](https://github.com/RunanywhereAI/runanywhere-sdks/tree/main/examples/web/RunAnywhereAI)
- Discord: [discord.gg/N359FBbDVd](https://discord.gg/N359FBbDVd)
