# RunAnywhere React Native SDK Reference

Complete guide for integrating RunAnywhere on-device AI into React Native applications.

## Installation

### Full Installation

```bash
npm install @runanywhere/core @runanywhere/llamacpp @runanywhere/onnx
# or
yarn add @runanywhere/core @runanywhere/llamacpp @runanywhere/onnx
```

### Minimal Installation (LLM Only)

```bash
npm install @runanywhere/core @runanywhere/llamacpp
```

### Platform Setup

**iOS:**

```bash
cd ios && pod install && cd ..
```

**Android:** No additional setup required.

## Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **React Native** | 0.71+ | 0.74+ |
| **iOS** | 15.1+ | 17.0+ |
| **Android** | API 24 (7.0+) | API 28+ |
| **Node.js** | 18+ | 20+ |
| **RAM** | 3GB | 6GB+ for 7B models |

## Quick Start

### 1. Initialize SDK

```typescript
import { RunAnywhere, SDKEnvironment } from '@runanywhere/core';
import { LlamaCPP } from '@runanywhere/llamacpp';
import { ONNX, ModelArtifactType } from '@runanywhere/onnx';

// Initialize SDK
await RunAnywhere.initialize({
  environment: SDKEnvironment.Development,
});

// Register LlamaCpp module
LlamaCPP.register();
await LlamaCPP.addModel({
  id: 'smollm2-360m-q8_0',
  name: 'SmolLM2 360M Q8_0',
  url: 'https://huggingface.co/prithivMLmods/SmolLM2-360M-GGUF/resolve/main/SmolLM2-360M.Q8_0.gguf',
  memoryRequirement: 500_000_000,
});

// Register ONNX module for STT/TTS
ONNX.register();
```

### 2. Download & Load Model

```typescript
// Download with progress tracking
await RunAnywhere.downloadModel('smollm2-360m-q8_0', (progress) => {
  console.log(`Download: ${(progress.progress * 100).toFixed(1)}%`);
});

// Load model
const modelInfo = await RunAnywhere.getModelInfo('smollm2-360m-q8_0');
if (modelInfo?.localPath) {
  await RunAnywhere.loadModel(modelInfo.localPath);
}

// Check if loaded
const isLoaded = await RunAnywhere.isModelLoaded();
console.log('Model loaded:', isLoaded);
```

### 3. Generate Text

```typescript
// Simple chat
const response = await RunAnywhere.chat('What is the capital of France?');
console.log(response);

// With options
const result = await RunAnywhere.generate(
  'Explain quantum computing in simple terms',
  {
    maxTokens: 200,
    temperature: 0.7,
    systemPrompt: 'You are a helpful assistant.',
  }
);

console.log('Response:', result.text);
console.log('Tokens/sec:', result.tokensPerSecond);
```

## Advanced Usage

### Streaming Text Generation

```typescript
// Basic streaming
for await (const token of RunAnywhere.generateStream('Tell me a story')) {
  console.log(token);
}

// With metrics
const streamResult = await RunAnywhere.generateStreamWithMetrics(
  'Write a poem about AI'
);

for await (const token of streamResult.stream) {
  console.log(token);
}

const metrics = await streamResult.metrics;
console.log('Speed:', metrics.tokensPerSecond);
```

### Speech-to-Text (STT)

```typescript
// Add STT model
await ONNX.addModel({
  id: 'whisper-tiny',
  name: 'Whisper Tiny',
  url: 'https://github.com/RunanywhereAI/sherpa-onnx/releases/download/runanywhere-models-v1/sherpa-onnx-whisper-tiny.en.tar.gz',
  modality: ModelCategory.SpeechRecognition,
  artifactType: ModelArtifactType.TarGzArchive,
});

// Download and load
await RunAnywhere.downloadModel('whisper-tiny');
const modelInfo = await RunAnywhere.getModelInfo('whisper-tiny');
await RunAnywhere.loadSTTModel(modelInfo.localPath);

// Transcribe audio
const text = await RunAnywhere.transcribe(audioData);
console.log('Transcription:', text);
```

### Text-to-Speech (TTS)

```typescript
// Add TTS model
await ONNX.addModel({
  id: 'piper-en-us',
  name: 'Piper English US',
  url: 'https://github.com/RunanywhereAI/sherpa-onnx/releases/download/runanywhere-models-v1/piper-en-us.tar.gz',
  modality: ModelCategory.SpeechSynthesis,
  artifactType: ModelArtifactType.TarGzArchive,
});

// Download and load
await RunAnywhere.downloadModel('piper-en-us');
const modelInfo = await RunAnywhere.getModelInfo('piper-en-us');
await RunAnywhere.loadTTSModel(modelInfo.localPath);

// Synthesize speech
const audioData = await RunAnywhere.synthesize('Hello from RunAnywhere!');
```

### Voice Agent Pipeline

```typescript
// Initialize voice agent
const agent = await RunAnywhere.createVoiceAgent({
  llmModelId: 'smollm2-360m-q8_0',
  sttModelId: 'whisper-tiny',
  ttsModelId: 'piper-en-us',
});

// Start listening
await agent.start();

// Handle events
agent.on('transcription', (text) => {
  console.log('User:', text);
});

agent.on('response', (text) => {
  console.log('AI:', text);
});

agent.on('audio', (audioData) => {
  // Play audio
});

// Stop
await agent.stop();
```

## Configuration

### Environment Modes

```typescript
await RunAnywhere.initialize({
  environment: SDKEnvironment.Development  // Development, Staging, Production
});
```

### Generation Options

```typescript
const options = {
  maxTokens: 100,
  temperature: 0.8,
  topP: 1.0,
  topK: 40,
  repeatPenalty: 1.1,
  stopSequences: ['END'],
  systemPrompt: 'You are a helpful assistant.',
};

const result = await RunAnywhere.generate(prompt, options);
```

## Error Handling

```typescript
import { RunAnywhereError } from '@runanywhere/core';

try {
  await RunAnywhere.loadModel(modelPath);
} catch (error) {
  if (error instanceof RunAnywhereError) {
    switch (error.code) {
      case 'MODEL_NOT_FOUND':
        console.error('Model not found');
        break;
      case 'INSUFFICIENT_MEMORY':
        console.error('Not enough memory');
        break;
      case 'INVALID_MODEL':
        console.error('Corrupted model file');
        break;
      default:
        console.error('Error:', error.message);
    }
  }
}
```

## Best Practices

### Memory Management

```typescript
import { Platform, NativeModules } from 'react-native';

// Check available memory (iOS)
if (Platform.OS === 'ios') {
  const { MemoryInfo } = NativeModules;
  const memoryInfo = await MemoryInfo.getAvailableMemory();

  if (memoryInfo.available < 1_000_000_000) {  // < 1GB
    console.warn('Low memory. Use smaller model.');
  }
}

// Unload models when not in use
await RunAnywhere.unloadModel();
```

### React Hooks

```typescript
import { useState, useEffect } from 'react';

function useLLM() {
  const [isLoaded, setIsLoaded] = useState(false);
  const [isGenerating, setIsGenerating] = useState(false);

  useEffect(() => {
    async function loadModel() {
      await RunAnywhere.loadModel(modelPath);
      setIsLoaded(true);
    }
    loadModel();

    return () => {
      RunAnywhere.unloadModel();
    };
  }, []);

  const generate = async (prompt: string) => {
    setIsGenerating(true);
    try {
      const response = await RunAnywhere.chat(prompt);
      return response;
    } finally {
      setIsGenerating(false);
    }
  };

  return { isLoaded, isGenerating, generate };
}

// Usage
function ChatScreen() {
  const { isLoaded, isGenerating, generate } = useLLM();

  const handleSend = async (message: string) => {
    const response = await generate(message);
    console.log(response);
  };

  if (!isLoaded) return <Text>Loading model...</Text>;

  return <ChatUI onSend={handleSend} loading={isGenerating} />;
}
```

### Background Processing

```typescript
// Use async tasks for long operations
import { InteractionManager } from 'react-native';

InteractionManager.runAfterInteractions(async () => {
  const result = await RunAnywhere.generate(longPrompt);
  updateUI(result);
});
```

## Permissions

### iOS (Info.plist)

```xml
<key>NSMicrophoneUsageDescription</key>
<string>This app needs microphone access for speech recognition</string>
```

### Android (AndroidManifest.xml)

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
```

Request runtime permissions:

```typescript
import { PermissionsAndroid, Platform } from 'react-native';

async function requestMicrophonePermission() {
  if (Platform.OS === 'android') {
    const granted = await PermissionsAndroid.request(
      PermissionsAndroid.PERMISSIONS.RECORD_AUDIO
    );
    return granted === PermissionsAndroid.RESULTS.GRANTED;
  }
  return true;  // iOS handles via Info.plist
}
```

## Debugging

### Enable Logging

```typescript
await RunAnywhere.initialize({
  environment: SDKEnvironment.Development,  // Verbose logging
  debug: true,
});
```

### Performance Monitoring

```typescript
const result = await RunAnywhere.generate(prompt);
console.log('Tokens/sec:', result.tokensPerSecond);
console.log('Latency:', result.latencyMs);
console.log('Memory:', result.memoryUsed);

if (result.tokensPerSecond < 5) {
  console.warn('Slow generation. Consider smaller model.');
}
```

### Native Logs

```bash
# iOS
npx react-native log-ios | grep "RunAnywhere"

# Android
npx react-native log-android | grep "RunAnywhere"
```

## Common Issues

### Model Download Fails

```typescript
// Retry with exponential backoff
async function downloadWithRetry(modelId: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      await RunAnywhere.downloadModel(modelId);
      return;
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
    }
  }
}
```

### Out of Memory

```typescript
// Check before loading
const modelInfo = await RunAnywhere.getModelInfo(modelId);
const deviceMemory = await getDeviceMemory();

if (deviceMemory < modelInfo.memoryRequirement * 2) {
  console.error('Insufficient memory');
  // Use smaller model
}
```

### Slow Generation

```typescript
// Use streaming for better UX
for await (const token of RunAnywhere.generateStream(prompt)) {
  // Update UI immediately
  setText(prev => prev + token);
}
```

## Resources

- Full Documentation: [docs.runanywhere.ai/react-native](https://docs.runanywhere.ai/react-native/introduction)
- Source Code: [sdk/runanywhere-react-native/](https://github.com/RunanywhereAI/runanywhere-sdks/tree/main/sdk/runanywhere-react-native)
- Example App: [examples/react-native/RunAnywhereAI/](https://github.com/RunanywhereAI/runanywhere-sdks/tree/main/examples/react-native/RunAnywhereAI)
- Discord: [discord.gg/N359FBbDVd](https://discord.gg/N359FBbDVd)
