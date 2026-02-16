# RunAnywhere Flutter SDK Reference

Complete guide for integrating RunAnywhere on-device AI into Flutter applications.

## Installation

Add to `pubspec.yaml`:

**Core + LlamaCpp (LLM):**

```yaml
dependencies:
  runanywhere: ^0.15.11
  runanywhere_llamacpp: ^0.15.11
```

**Core + ONNX (STT/TTS/VAD):**

```yaml
dependencies:
  runanywhere: ^0.15.11
  runanywhere_onnx: ^0.15.11
```

**All Backends:**

```yaml
dependencies:
  runanywhere: ^0.15.11
  runanywhere_llamacpp: ^0.15.11
  runanywhere_onnx: ^0.15.11
```

Then run:

```bash
flutter pub get
```

## Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **Flutter** | 3.10.0+ | 3.24.0+ |
| **Dart** | 3.0.0+ | 3.5.0+ |
| **iOS** | 14.0+ | 15.0+ |
| **Android** | API 24 (7.0) | API 28+ |
| **RAM** | 2GB | 4GB+ for larger models |

## Platform Setup

### iOS Setup (Required)

Update `ios/Podfile`:

```ruby
# Set minimum iOS version to 14.0
platform :ios, '14.0'

target 'Runner' do
  # REQUIRED: Add static linkage
  use_frameworks! :linkage => :static

  flutter_install_all_ios_pods File.dirname(File.realpath(__FILE__))
end

post_install do |installer|
  installer.pods_project.targets.each do |target|
    flutter_additional_ios_build_settings(target)
    target.build_configurations.each do |config|
      config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '14.0'
    end
  end
end
```

Update `ios/Runner/Info.plist`:

```xml
<key>NSMicrophoneUsageDescription</key>
<string>This app needs microphone access for speech recognition</string>
```

Run pod install:

```bash
cd ios && pod install && cd ..
```

### Android Setup

Add to `android/app/src/main/AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.RECORD_AUDIO" />
```

## Quick Start

### 1. Initialize SDK

```dart
import 'package:runanywhere/runanywhere.dart';
import 'package:runanywhere_llamacpp/runanywhere_llamacpp.dart';
import 'package:runanywhere_onnx/runanywhere_onnx.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // Initialize SDK
  await RunAnywhere.initialize();

  // Register modules
  await LlamaCpp.register();
  await ONNX.register();

  runApp(MyApp());
}
```

### 2. Download & Load Model

```dart
// Download with progress tracking
await RunAnywhere.downloadModel(
  'smollm2-360m',
  onProgress: (progress) {
    print('Download: ${(progress * 100).toStringAsFixed(1)}%');
  },
);

// Load model
await RunAnywhere.loadModel('smollm2-360m');

// Check if loaded
final isLoaded = await RunAnywhere.isModelLoaded();
print('Model loaded: $isLoaded');
```

### 3. Generate Text

```dart
// Simple chat
final response = await RunAnywhere.chat('What is the capital of France?');
print(response);

// With options
final result = await RunAnywhere.generate(
  'Explain quantum computing in simple terms',
  options: LLMGenerationOptions(
    maxTokens: 200,
    temperature: 0.7,
    systemPrompt: 'You are a helpful assistant.',
  ),
);

print('Response: ${result.text}');
print('Tokens/sec: ${result.tokensPerSecond}');
```

## Advanced Usage

### Streaming Text Generation

```dart
// Basic streaming
await for (final token in RunAnywhere.generateStream('Tell me a story')) {
  print(token);
}

// With metrics
final streamResult = await RunAnywhere.generateStreamWithMetrics(
  'Write a poem about AI'
);

await for (final token in streamResult.stream) {
  print(token);
}

final metrics = await streamResult.metrics;
print('Speed: ${metrics.tokensPerSecond} tok/s');
```

### Speech-to-Text (STT)

```dart
// Load STT model
await RunAnywhere.loadSTTModel('whisper-tiny');

// Transcribe audio
final text = await RunAnywhere.transcribe(audioData);
print('Transcription: $text');

// With options
final result = await RunAnywhere.transcribeWithOptions(
  audioData,
  options: STTOptions(
    language: 'en',
    enableTimestamps: true,
  ),
);

print('Text: ${result.text}');
print('Confidence: ${result.confidence}');
```

### Text-to-Speech (TTS)

```dart
// Load TTS voice
await RunAnywhere.loadTTSVoice('en-us-default');

// Synthesize speech
final audioData = await RunAnywhere.synthesize(
  'Hello from RunAnywhere!',
  options: TTSOptions(
    rate: 1.0,
    pitch: 1.0,
  ),
);

// Play audio (use audioplayers package)
final player = AudioPlayer();
await player.playBytes(audioData);
```

### Voice Agent Pipeline

```dart
// Configure voice agent
await RunAnywhere.configureVoiceAgent(
  VoiceAgentConfig(
    sttModelId: 'whisper-tiny',
    llmModelId: 'smollm2-360m',
    ttsVoiceId: 'en-us-default',
  ),
);

// Start voice session
final stream = RunAnywhere.startVoiceSession();

await for (final event in stream) {
  if (event is VoiceSessionListening) {
    print('Listening...');
  } else if (event is VoiceSessionTranscribed) {
    print('User: ${event.text}');
  } else if (event is VoiceSessionResponded) {
    print('AI: ${event.text}');
  } else if (event is VoiceSessionSpeaking) {
    print('Speaking...');
  } else if (event is VoiceSessionError) {
    print('Error: ${event.message}');
  }
}

// Stop session
await RunAnywhere.stopVoiceSession();
```

## Configuration

### Environment Modes

```dart
await RunAnywhere.initialize(
  environment: SDKEnvironment.development,  // development, staging, production
);
```

### Generation Options

```dart
final options = LLMGenerationOptions(
  maxTokens: 100,
  temperature: 0.8,
  topP: 1.0,
  topK: 40,
  repeatPenalty: 1.1,
  stopSequences: ['END'],
  systemPrompt: 'You are a helpful assistant.',
);

final result = await RunAnywhere.generate(prompt, options: options);
```

## Error Handling

```dart
try {
  await RunAnywhere.loadModel(modelId);
} on ModelNotFoundException {
  print('Model not found. Download it first.');
} on InsufficientMemoryException {
  print('Not enough memory. Try a smaller model.');
} on InvalidModelException {
  print('Corrupted model file. Re-download.');
} catch (e) {
  print('Unexpected error: $e');
}
```

## Best Practices

### State Management

```dart
import 'package:flutter/material.dart';

class LLMProvider with ChangeNotifier {
  bool _isLoaded = false;
  bool _isGenerating = false;

  bool get isLoaded => _isLoaded;
  bool get isGenerating => _isGenerating;

  Future<void> loadModel(String modelId) async {
    await RunAnywhere.loadModel(modelId);
    _isLoaded = true;
    notifyListeners();
  }

  Future<String> generate(String prompt) async {
    _isGenerating = true;
    notifyListeners();

    try {
      final response = await RunAnywhere.chat(prompt);
      return response;
    } finally {
      _isGenerating = false;
      notifyListeners();
    }
  }

  void dispose() {
    RunAnywhere.unloadModel();
    super.dispose();
  }
}

// Usage with Provider
class ChatScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Consumer<LLMProvider>(
      builder: (context, llm, child) {
        if (!llm.isLoaded) {
          return CircularProgressIndicator();
        }

        return ChatUI(
          onSend: (message) => llm.generate(message),
          loading: llm.isGenerating,
        );
      },
    );
  }
}
```

### Memory Management

```dart
// Check available memory
import 'dart:io';

Future<bool> canLoadModel(String modelId) async {
  final modelSize = await RunAnywhere.getModelSize(modelId);

  if (Platform.isAndroid) {
    // Android memory check (requires permission)
    final memInfo = await getMemoryInfo();
    return memInfo.availableMemory > modelSize * 2;
  } else if (Platform.isIOS) {
    // iOS memory check
    final memInfo = await getMemoryInfo();
    return memInfo.availableMemory > modelSize * 2;
  }

  return true;
}

// Unload models when not in use
@override
void dispose() {
  RunAnywhere.unloadModel();
  super.dispose();
}
```

### Background Processing

```dart
import 'dart:isolate';

// Run generation in isolate for heavy workloads
Future<String> generateInIsolate(String prompt) async {
  final receivePort = ReceivePort();

  await Isolate.spawn(_generateWorker, {
    'sendPort': receivePort.sendPort,
    'prompt': prompt,
  });

  return await receivePort.first as String;
}

void _generateWorker(Map<String, dynamic> args) async {
  final sendPort = args['sendPort'] as SendPort;
  final prompt = args['prompt'] as String;

  final response = await RunAnywhere.chat(prompt);
  sendPort.send(response);
}
```

## Permissions

### Request Microphone Permission

```dart
import 'package:permission_handler/permission_handler.dart';

Future<bool> requestMicrophonePermission() async {
  final status = await Permission.microphone.request();
  return status.isGranted;
}

// Usage
if (await requestMicrophonePermission()) {
  // Start voice recording
} else {
  // Show permission denied message
}
```

## Debugging

### Enable Logging

```dart
await RunAnywhere.initialize(
  environment: SDKEnvironment.development,  // Verbose logging
  debug: true,
);
```

### Performance Monitoring

```dart
final result = await RunAnywhere.generate(prompt);
print('Tokens/sec: ${result.tokensPerSecond}');
print('Latency: ${result.latencyMs}ms');

if (result.tokensPerSecond < 5) {
  print('Warning: Slow generation. Consider smaller model.');
}
```

### Native Logs

```bash
# iOS
flutter run --verbose | grep "RunAnywhere"

# Android
flutter run --verbose | grep "RunAnywhere"
adb logcat | grep "RunAnywhere"
```

## Common Issues

### Model Download Fails

```dart
// Retry with exponential backoff
Future<void> downloadWithRetry(String modelId, {int maxRetries = 3}) async {
  for (var i = 0; i < maxRetries; i++) {
    try {
      await RunAnywhere.downloadModel(modelId);
      return;
    } catch (e) {
      if (i == maxRetries - 1) rethrow;
      await Future.delayed(Duration(seconds: pow(2, i).toInt()));
    }
  }
}
```

### Out of Memory

```dart
// Check before loading
final modelSize = await RunAnywhere.getModelSize(modelId);
final availableMemory = await getAvailableMemory();

if (availableMemory < modelSize * 2) {
  throw Exception('Insufficient memory');
}
```

### Slow Generation

```dart
// Use streaming for better UX
String generatedText = '';

await for (final token in RunAnywhere.generateStream(prompt)) {
  setState(() {
    generatedText += token;
  });
}
```

### iOS Build Errors

Ensure static linkage in Podfile:

```ruby
use_frameworks! :linkage => :static
```

Run pod install after changes:

```bash
cd ios && pod install && cd ..
```

## Testing

### Unit Tests

```dart
import 'package:flutter_test/flutter_test.dart';

void main() {
  setUp(() async {
    await RunAnywhere.initialize();
  });

  test('Generate text', () async {
    await RunAnywhere.loadModel('test-model');
    final response = await RunAnywhere.chat('Hello');

    expect(response, isNotEmpty);
  });
}
```

### Widget Tests

```dart
testWidgets('Chat screen generates response', (tester) async {
  await tester.pumpWidget(MyApp());

  // Enter message
  await tester.enterText(find.byType(TextField), 'Hello');
  await tester.tap(find.byIcon(Icons.send));

  // Wait for generation
  await tester.pumpAndSettle();

  // Verify response appears
  expect(find.textContaining('Hello'), findsOneWidget);
});
```

## Resources

- Full Documentation: [docs.runanywhere.ai/flutter](https://docs.runanywhere.ai/flutter/introduction)
- Source Code: [sdk/runanywhere-flutter/](https://github.com/RunanywhereAI/runanywhere-sdks/tree/main/sdk/runanywhere-flutter)
- Example App: [examples/flutter/RunAnywhereAI/](https://github.com/RunanywhereAI/runanywhere-sdks/tree/main/examples/flutter/RunAnywhereAI)
- Discord: [discord.gg/N359FBbDVd](https://discord.gg/N359FBbDVd)
