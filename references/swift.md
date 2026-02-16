# RunAnywhere Swift SDK Reference

Complete guide for integrating RunAnywhere on-device AI into iOS, macOS, tvOS, and watchOS apps.

## Installation

### Swift Package Manager

Add to Xcode: **File > Add Package Dependencies...**

```
https://github.com/RunanywhereAI/runanywhere-sdks
```

Select products:
- **RunAnywhere** (required) - Core SDK
- **RunAnywhereLlamaCPP** - LLM text generation with GGUF models
- **RunAnywhereONNX** - STT/TTS/VAD via ONNX Runtime

### Package.swift

```swift
dependencies: [
    .package(url: "https://github.com/RunanywhereAI/runanywhere-sdks", from: "1.0.0")
],
targets: [
    .target(
        name: "YourApp",
        dependencies: [
            .product(name: "RunAnywhere", package: "runanywhere-sdks"),
            .product(name: "RunAnywhereLlamaCPP", package: "runanywhere-sdks"),
            .product(name: "RunAnywhereONNX", package: "runanywhere-sdks"),
        ]
    )
]
```

## Requirements

| Platform | Minimum Version |
|----------|-----------------|
| iOS      | 17.0+           |
| macOS    | 14.0+           |
| tvOS     | 17.0+           |
| watchOS  | 10.0+           |

- Swift 5.9+
- Xcode 15.2+
- 2GB RAM minimum (4GB+ recommended for larger models)

## Quick Start

### 1. Initialize SDK

```swift
import RunAnywhere
import LlamaCPPRuntime

@main
struct MyApp: App {
    init() {
        Task { @MainActor in
            // Register modules
            LlamaCPP.register()

            // Initialize SDK
            do {
                try RunAnywhere.initialize(
                    apiKey: "<YOUR_API_KEY>",
                    baseURL: "https://api.runanywhere.ai",
                    environment: .production
                )
            } catch {
                print("SDK initialization failed: \(error)")
            }
        }
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

### 2. Download and Load Model

```swift
// Download model with progress tracking
try await RunAnywhere.downloadModel("smollm2-360m")

// Load model
try await RunAnywhere.loadModel("smollm2-360m")

// Check if loaded
let isLoaded = await RunAnywhere.isModelLoaded
```

### 3. Generate Text

```swift
// Simple chat interface
let response = try await RunAnywhere.chat("What is the capital of France?")
print(response)  // "The capital of France is Paris."

// With options and metrics
let result = try await RunAnywhere.generate(
    "Explain quantum computing in simple terms",
    options: LLMGenerationOptions(
        maxTokens: 200,
        temperature: 0.7
    )
)
print("Response: \(result.text)")
print("Tokens used: \(result.tokensUsed)")
print("Speed: \(result.tokensPerSecond) tok/s")
```

## Advanced Usage

### Streaming Text Generation

```swift
let result = try await RunAnywhere.generateStream(
    "Write a short poem about AI",
    options: LLMGenerationOptions(maxTokens: 150)
)

for try await token in result.stream {
    print(token, terminator: "")
}

let metrics = try await result.result.value
print("\nGenerated \(metrics.tokensUsed) tokens at \(metrics.tokensPerSecond) tok/s")
```

### Speech-to-Text (STT)

```swift
import ONNXRuntime

// Register ONNX module for STT
ONNX.register()

// Download STT model
try await RunAnywhere.downloadModel("whisper-tiny")
try await RunAnywhere.loadSTTModel("whisper-tiny")

// Transcribe audio data
let audioData: Data = // ... your audio data
let result = try await RunAnywhere.transcribe(audioData)
print("Transcription: \(result.text)")
```

### Text-to-Speech (TTS)

```swift
// Download TTS model
try await RunAnywhere.downloadModel("piper-us-english")
try await RunAnywhere.loadTTSModel("piper-us-english")

// Generate speech
let audioData = try await RunAnywhere.synthesize(
    "Hello, this is RunAnywhere text to speech!",
    options: TTSOptions(
        rate: 1.0,
        pitch: 1.0,
        volume: 1.0
    )
)

// Play the audio
let player = try AVAudioPlayer(data: audioData)
player.play()
```

### Voice Agent Pipeline

Complete voice conversation flow with VAD → STT → LLM → TTS:

```swift
// Initialize voice agent
try await RunAnywhere.initializeVoiceAgent(
    sttModelId: "whisper-tiny",
    llmModelId: "smollm2-360m",
    ttsVoice: "piper-us-english"
)

// Process a voice turn
let audioData: Data = // ... your recorded audio
let result = try await RunAnywhere.processVoiceTurn(audioData)

print("User said: \(result.transcription)")
print("AI response: \(result.response)")
// result.audioResponse contains synthesized TTS audio

// Cleanup when done
await RunAnywhere.cleanupVoiceAgent()
```

### Structured Output Generation

Generate typed Swift objects using `Generatable` protocol:

```swift
struct Recipe: Codable, Generatable {
    let name: String
    let ingredients: [String]
    let steps: [String]
}

let recipe: Recipe = try await RunAnywhere.generate(
    "Give me a recipe for chocolate chip cookies"
)
print("Recipe: \(recipe.name)")
recipe.ingredients.forEach { print("- \($0)") }
```

### Event System

Subscribe to SDK events for reactive UI updates:

```swift
import Combine

class ViewModel: ObservableObject {
    @Published var downloadProgress: Double = 0
    @Published var generationStatus: String = ""

    private var cancellables = Set<AnyCancellable>()

    init() {
        // Subscribe to model download progress
        RunAnywhere.eventBus.subscribe(to: ModelDownloadProgressEvent.self) { event in
            self.downloadProgress = event.progress
        }.store(in: &cancellables)

        // Subscribe to generation events
        RunAnywhere.eventBus.subscribe(to: GenerationStartedEvent.self) { event in
            self.generationStatus = "Generating..."
        }.store(in: &cancellables)

        RunAnywhere.eventBus.subscribe(to: GenerationCompletedEvent.self) { event in
            self.generationStatus = "Completed"
        }.store(in: &cancellables)
    }
}
```

## Configuration

### Environment Modes

```swift
try RunAnywhere.initialize(
    apiKey: "<YOUR_API_KEY>",
    baseURL: "https://api.runanywhere.ai",
    environment: .production  // .development, .staging, or .production
)
```

| Environment     | Description                                      |
|-----------------|--------------------------------------------------|
| `.development`  | Verbose logging, mock services, local analytics  |
| `.staging`      | Testing with real services                       |
| `.production`   | Minimal logging, full authentication, telemetry  |

### Generation Options

```swift
let options = LLMGenerationOptions(
    maxTokens: 100,
    temperature: 0.8,
    topP: 1.0,
    stopSequences: ["END"],
    streamingEnabled: false,
    preferredFramework: .llamaCpp,
    systemPrompt: "You are a helpful assistant."
)
```

## Error Handling

### Common Errors

```swift
do {
    try await RunAnywhere.loadModel("my-model")
} catch RunAnywhereError.modelNotFound {
    print("Model not found. Download it first.")
} catch RunAnywhereError.insufficientMemory {
    print("Not enough memory. Try a smaller model.")
} catch RunAnywhereError.invalidConfiguration {
    print("Invalid configuration. Check your settings.")
} catch {
    print("Unexpected error: \(error)")
}
```

### Error Types

- `modelNotFound` - Model file doesn't exist
- `insufficientMemory` - Not enough RAM for model
- `invalidConfiguration` - Invalid SDK setup
- `networkError` - Download failed
- `invalidModel` - Corrupted model file
- `generationError` - Text generation failed
- `transcriptionError` - STT failed
- `synthesisError` - TTS failed

## Best Practices

### Memory Management

```swift
// Use smaller quantized models for memory-constrained devices
let deviceMemory = ProcessInfo.processInfo.physicalMemory
let modelSize: String

if deviceMemory < 4_000_000_000 {  // < 4GB
    modelSize = "smollm2-360m"  // ~400MB
} else if deviceMemory < 8_000_000_000 {  // < 8GB
    modelSize = "llama-3.2-1b-instruct-q4"  // ~1GB
} else {
    modelSize = "mistral-7b-instruct-q4"  // ~4GB
}

try await RunAnywhere.loadModel(modelSize)
```

### Background Processing

```swift
// Keep SDK responsive during long operations
Task.detached(priority: .userInitiated) {
    do {
        let result = try await RunAnywhere.generate(longPrompt)
        await MainActor.run {
            self.updateUI(with: result)
        }
    } catch {
        await MainActor.run {
            self.handleError(error)
        }
    }
}
```

### Model Lifecycle

```swift
// Unload models when not in use to free memory
await RunAnywhere.unloadModel()

// Load on-demand
func generateResponse(_ prompt: String) async throws -> String {
    if !await RunAnywhere.isModelLoaded {
        try await RunAnywhere.loadModel("smollm2-360m")
    }
    return try await RunAnywhere.chat(prompt)
}
```

### Performance Optimization

```swift
// Use Metal GPU acceleration (default on Apple Silicon)
// No configuration needed - automatically enabled

// Monitor performance
let result = try await RunAnywhere.generate(prompt)
if result.tokensPerSecond < 10 {
    print("Warning: Slow generation. Consider smaller model.")
}

// Batch processing for multiple requests
let prompts = ["Question 1", "Question 2", "Question 3"]
try await withThrowingTaskGroup(of: String.self) { group in
    for prompt in prompts {
        group.addTask {
            try await RunAnywhere.chat(prompt)
        }
    }

    for try await response in group {
        print(response)
    }
}
```

## Debugging

### Logging

```swift
// Enable verbose logging in development
try RunAnywhere.initialize(
    apiKey: key,
    baseURL: baseURL,
    environment: .development
)

// View logs with Pulse (optional integration)
import Pulse
let logger = NetworkLogger()
```

### Device Logs

```bash
# Simulator logs
log stream --predicate 'subsystem CONTAINS "com.runanywhere"' --info --debug

# Physical device logs
idevicesyslog | grep "com.runanywhere"
```

## Common Issues

### Model Download Fails

```swift
// Retry with exponential backoff
func downloadWithRetry(modelID: String, maxRetries: Int = 3) async throws {
    for attempt in 0..<maxRetries {
        do {
            try await RunAnywhere.downloadModel(modelID)
            return
        } catch {
            if attempt == maxRetries - 1 { throw error }
            try await Task.sleep(nanoseconds: UInt64(pow(2.0, Double(attempt)) * 1_000_000_000))
        }
    }
}
```

### Out of Memory

```swift
// Check available memory before loading
func canLoadModel(_ modelID: String) async -> Bool {
    let modelSize = await RunAnywhere.getModelSize(modelID)
    let availableMemory = ProcessInfo.processInfo.physicalMemory
    return availableMemory > modelSize * 2  // 2x headroom
}
```

### Slow Generation

```swift
// Use streaming for better UX
let result = try await RunAnywhere.generateStream(prompt)
for try await token in result.stream {
    // Display tokens as they arrive
    updateUI(with: token)
}
```

## Resources

- Full Documentation: [docs.runanywhere.ai/swift](https://docs.runanywhere.ai/swift/introduction)
- Source Code: [sdk/runanywhere-swift/](https://github.com/RunanywhereAI/runanywhere-sdks/tree/main/sdk/runanywhere-swift)
- Example App: [examples/ios/RunAnywhereAI/](https://github.com/RunanywhereAI/runanywhere-sdks/tree/main/examples/ios/RunAnywhereAI)
- Discord: [discord.gg/N359FBbDVd](https://discord.gg/N359FBbDVd)
