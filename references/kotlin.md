# RunAnywhere Kotlin SDK Reference

Complete guide for integrating RunAnywhere on-device AI into Android and JVM applications.

## Installation

### Gradle (build.gradle.kts)

```kotlin
dependencies {
    // Core SDK
    implementation("com.runanywhere.sdk:runanywhere-kotlin:0.1.4")

    // Optional: LLM support (llama.cpp backend) - ~34MB
    implementation("com.runanywhere.sdk:runanywhere-core-llamacpp:0.1.4")

    // Optional: STT/TTS/VAD support (ONNX backend) - ~25MB
    implementation("com.runanywhere.sdk:runanywhere-core-onnx:0.1.4")
}
```

## Requirements

- **Android**: API 24+ (Android 7.0+)
- **JVM**: Java 11+
- **Kotlin**: 1.9+
- **Memory**: 2GB minimum, 4GB+ recommended

## Quick Start

### 1. Initialize SDK

```kotlin
import com.runanywhere.sdk.public.RunAnywhere
import com.runanywhere.sdk.public.SDKEnvironment

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        // Initialize RunAnywhere (fast, ~1-5ms)
        RunAnywhere.initialize(
            apiKey = "your-api-key",    // Optional for development
            environment = SDKEnvironment.DEVELOPMENT
        )
    }
}
```

### 2. Register & Download Model

```kotlin
import com.runanywhere.sdk.public.RunAnywhere
import com.runanywhere.sdk.public.extensions.*
import com.runanywhere.sdk.core.types.InferenceFramework

// Register a model from HuggingFace
val modelInfo = RunAnywhere.registerModel(
    name = "Qwen 0.5B",
    url = "https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct-GGUF/resolve/main/qwen2.5-0.5b-instruct-q8_0.gguf",
    framework = InferenceFramework.LLAMA_CPP
)

// Download the model (observe progress)
RunAnywhere.downloadModel(modelInfo.id)
    .collect { progress ->
        println("Download: ${(progress.progress * 100).toInt()}%")
    }
```

### 3. Generate Text

```kotlin
// Load the model
RunAnywhere.loadLLMModel(modelInfo.id)

// Simple chat
val response = RunAnywhere.chat("What is machine learning?")
println(response)

// With full metrics
val result = RunAnywhere.generate(
    prompt = "Explain quantum computing",
    options = LLMGenerationOptions(
        maxTokens = 150,
        temperature = 0.7f
    )
)
println("Response: ${result.text}")
println("Tokens/sec: ${result.tokensPerSecond}")
println("Latency: ${result.latencyMs}ms")
```

## Advanced Usage

### Streaming Text Generation

```kotlin
// Stream tokens as they're generated
RunAnywhere.generateStream("Tell me a story about AI")
    .collect { token ->
        print(token) // Display in real-time
    }

// With metrics
val streamResult = RunAnywhere.generateStreamWithMetrics("Write a poem")
streamResult.stream.collect { token -> print(token) }
val metrics = streamResult.result.await()
println("\nSpeed: ${metrics.tokensPerSecond} tok/s")

// Cancel ongoing generation
RunAnywhere.cancelGeneration()
```

### Speech-to-Text (STT)

```kotlin
// Load an STT model
RunAnywhere.loadSTTModel("whisper-tiny")

// Transcribe audio
val text = RunAnywhere.transcribe(audioData)

// With options
val output = RunAnywhere.transcribeWithOptions(
    audioData = audioBytes,
    options = STTOptions(
        language = "en",
        enableTimestamps = true,
        enablePunctuation = true
    )
)
println("Text: ${output.text}")
println("Confidence: ${output.confidence}")

// Streaming transcription
RunAnywhere.transcribeStream(audioData) { partial ->
    println("Partial: ${partial.transcript}")
}
```

### Text-to-Speech (TTS)

```kotlin
// Load a TTS voice
RunAnywhere.loadTTSVoice("en-us-default")

// Simple speak (plays audio automatically)
RunAnywhere.speak("Hello, world!")

// Synthesize to bytes
val output = RunAnywhere.synthesize(
    text = "Welcome to RunAnywhere",
    options = TTSOptions(
        rate = 1.0f,
        pitch = 1.0f
    )
)

// Stream synthesis for long text
RunAnywhere.synthesizeStream(longText) { chunk ->
    audioPlayer.play(chunk)
}
```

### Voice Activity Detection (VAD)

```kotlin
// Detect speech in audio
val result = RunAnywhere.detectVoiceActivity(audioData)
println("Speech detected: ${result.hasSpeech}")
println("Confidence: ${result.confidence}")

// Configure VAD
RunAnywhere.configureVAD(VADConfiguration(
    threshold = 0.5f,
    minSpeechDurationMs = 250,
    minSilenceDurationMs = 300
))

// Stream VAD
RunAnywhere.streamVAD(audioSamplesFlow)
    .collect { result ->
        if (result.hasSpeech) println("Speaking...")
    }
```

### Voice Agent Pipeline

Complete VAD → STT → LLM → TTS orchestration:

```kotlin
// Configure voice agent
RunAnywhere.configureVoiceAgent(VoiceAgentConfiguration(
    sttModelId = "whisper-tiny",
    llmModelId = "qwen-0.5b",
    ttsVoiceId = "en-us-default"
))

// Start voice session
RunAnywhere.startVoiceSession()
    .collect { event ->
        when (event) {
            is VoiceSessionEvent.Listening -> println("Listening...")
            is VoiceSessionEvent.Transcribed -> println("You: ${event.text}")
            is VoiceSessionEvent.Thinking -> println("Thinking...")
            is VoiceSessionEvent.Responded -> println("AI: ${event.text}")
            is VoiceSessionEvent.Speaking -> println("Speaking...")
            is VoiceSessionEvent.Error -> println("Error: ${event.message}")
        }
    }

// Stop session
RunAnywhere.stopVoiceSession()
```

### Model Management

```kotlin
// List available models
val models = RunAnywhere.availableModels()
models.forEach { model ->
    println("${model.name} (${model.sizeBytes / 1_000_000}MB)")
}

// Check if model is downloaded
val isDownloaded = RunAnywhere.isModelDownloaded(modelId)

// Get model info
val modelInfo = RunAnywhere.getModelInfo(modelId)
println("Size: ${modelInfo.sizeBytes}")
println("Framework: ${modelInfo.framework}")

// Delete model
RunAnywhere.deleteModel(modelId)

// Get model download path
val path = RunAnywhere.getModelPath(modelId)
```

## Configuration

### Environment Modes

```kotlin
RunAnywhere.initialize(
    apiKey = "your-api-key",
    environment = SDKEnvironment.DEVELOPMENT  // or STAGING, PRODUCTION
)
```

| Environment       | Description                                     |
|-------------------|-------------------------------------------------|
| `DEVELOPMENT`     | Verbose logging, mock services, local analytics |
| `STAGING`         | Testing with real services                      |
| `PRODUCTION`      | Minimal logging, full authentication, telemetry |

### Generation Options

```kotlin
val options = LLMGenerationOptions(
    maxTokens = 100,
    temperature = 0.8f,
    topP = 1.0f,
    topK = 40,
    repeatPenalty = 1.1f,
    stopSequences = listOf("END", "\n\n"),
    systemPrompt = "You are a helpful assistant.",
    streamingEnabled = false
)
```

### STT Options

```kotlin
val sttOptions = STTOptions(
    language = "en",
    enableTimestamps = true,
    enablePunctuation = true,
    enableWordSegmentation = false,
    enableVAD = true
)
```

### TTS Options

```kotlin
val ttsOptions = TTSOptions(
    rate = 1.0f,      // 0.5 - 2.0
    pitch = 1.0f,     // 0.5 - 2.0
    volume = 1.0f     // 0.0 - 1.0
)
```

## Error Handling

### Common Errors

```kotlin
try {
    RunAnywhere.loadLLMModel(modelId)
} catch (e: ModelNotFoundException) {
    println("Model not found. Download it first.")
} catch (e: InsufficientMemoryException) {
    println("Not enough memory. Try a smaller model.")
} catch (e: InvalidModelException) {
    println("Corrupted model file. Re-download.")
} catch (e: GenerationException) {
    println("Text generation failed: ${e.message}")
} catch (e: Exception) {
    println("Unexpected error: ${e.message}")
}
```

### Error Types

- `ModelNotFoundException` - Model file doesn't exist
- `InsufficientMemoryException` - Not enough RAM
- `InvalidModelException` - Corrupted model
- `GenerationException` - LLM generation failed
- `TranscriptionException` - STT failed
- `SynthesisException` - TTS failed
- `NetworkException` - Download failed

## Best Practices

### Memory Management

```kotlin
// Check available memory before loading
fun canLoadModel(modelId: String): Boolean {
    val runtime = Runtime.getRuntime()
    val maxMemory = runtime.maxMemory()
    val usedMemory = runtime.totalMemory() - runtime.freeMemory()
    val availableMemory = maxMemory - usedMemory

    val modelSize = RunAnywhere.getModelInfo(modelId).sizeBytes
    return availableMemory > modelSize * 2  // 2x headroom
}

// Unload models when not in use
RunAnywhere.unloadModel()

// Use smaller models on low-memory devices
val activityManager = getSystemService(Context.ACTIVITY_SERVICE) as ActivityManager
val memoryInfo = ActivityManager.MemoryInfo()
activityManager.getMemoryInfo(memoryInfo)

val modelSize = when {
    memoryInfo.totalMem < 4_000_000_000 -> "smollm2-360m"  // < 4GB
    memoryInfo.totalMem < 8_000_000_000 -> "qwen-0.5b"     // < 8GB
    else -> "mistral-7b-q4"  // >= 8GB
}
```

### Background Processing

```kotlin
// Use coroutines for async operations
lifecycleScope.launch {
    try {
        val result = withContext(Dispatchers.Default) {
            RunAnywhere.generate(longPrompt)
        }
        withContext(Dispatchers.Main) {
            updateUI(result)
        }
    } catch (e: Exception) {
        handleError(e)
    }
}

// Cancel ongoing operations
val job = lifecycleScope.launch {
    RunAnywhere.generateStream(prompt).collect { token ->
        print(token)
    }
}
// Later: job.cancel()
```

### Model Lifecycle

```kotlin
class ChatViewModel : ViewModel() {
    private var modelLoaded = false

    suspend fun ensureModelLoaded(modelId: String) {
        if (!modelLoaded) {
            RunAnywhere.loadLLMModel(modelId)
            modelLoaded = true
        }
    }

    override fun onCleared() {
        super.onCleared()
        // Unload model when ViewModel is destroyed
        runBlocking {
            RunAnywhere.unloadModel()
        }
    }
}
```

### Performance Optimization

```kotlin
// Monitor performance metrics
val result = RunAnywhere.generate(prompt)
if (result.tokensPerSecond < 10) {
    Log.w("Performance", "Slow generation. Consider smaller model.")
}

// Batch processing
suspend fun processMultiplePrompts(prompts: List<String>): List<String> {
    return coroutineScope {
        prompts.map { prompt ->
            async {
                RunAnywhere.chat(prompt)
            }
        }.awaitAll()
    }
}

// Optimize for streaming UX
lifecycleScope.launch {
    val streamResult = RunAnywhere.generateStreamWithMetrics(prompt)
    streamResult.stream.collect { token ->
        // Update UI immediately for better UX
        textView.append(token)
    }
    val metrics = streamResult.result.await()
    Log.d("Metrics", "Generated at ${metrics.tokensPerSecond} tok/s")
}
```

### Permissions (Android)

```xml
<!-- AndroidManifest.xml -->

<!-- Required for model downloads -->
<uses-permission android:name="android.permission.INTERNET" />

<!-- Optional: for mic access (STT/Voice Agent) -->
<uses-permission android:name="android.permission.RECORD_AUDIO" />

<!-- Optional: for speaker access (TTS) -->
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
```

### ProGuard Rules

```proguard
# RunAnywhere SDK
-keep class com.runanywhere.sdk.** { *; }
-dontwarn com.runanywhere.sdk.**

# ONNX Runtime
-keep class ai.onnxruntime.** { *; }
-dontwarn ai.onnxruntime.**

# llama.cpp
-keep class com.llamacpp.** { *; }
-dontwarn com.llamacpp.**
```

## Debugging

### Enable Logging

```kotlin
// In Application.onCreate()
RunAnywhere.initialize(
    apiKey = key,
    environment = SDKEnvironment.DEVELOPMENT  // Enables verbose logging
)

// View logs
adb logcat | grep "RunAnywhere"
```

### Common Issues

**Model Download Fails**

```kotlin
// Retry with exponential backoff
suspend fun downloadWithRetry(modelId: String, maxRetries: Int = 3) {
    repeat(maxRetries) { attempt ->
        try {
            RunAnywhere.downloadModel(modelId).collect { progress ->
                println("Download: ${(progress.progress * 100).toInt()}%")
            }
            return
        } catch (e: Exception) {
            if (attempt == maxRetries - 1) throw e
            delay(2.0.pow(attempt).toLong() * 1000)
        }
    }
}
```

**Out of Memory**

```kotlin
// Check before loading
val activityManager = getSystemService(Context.ACTIVITY_SERVICE) as ActivityManager
val memoryInfo = ActivityManager.MemoryInfo()
activityManager.getMemoryInfo(memoryInfo)

if (memoryInfo.availMem < modelSize * 2) {
    // Not enough memory
    showError("Insufficient memory. Close other apps and try again.")
}
```

**Slow Generation**

```kotlin
// Use streaming for better UX
lifecycleScope.launch {
    RunAnywhere.generateStream(prompt).collect { token ->
        // Display tokens immediately
        textView.append(token)
    }
}
```

## Testing

### Unit Tests

```kotlin
class RunAnywhereTest {
    @Before
    fun setup() {
        RunAnywhere.initialize(
            apiKey = "test-key",
            environment = SDKEnvironment.DEVELOPMENT
        )
    }

    @Test
    fun testGeneration() = runBlocking {
        RunAnywhere.loadLLMModel("test-model")
        val result = RunAnywhere.chat("Hello")
        assertNotNull(result)
        assertTrue(result.isNotEmpty())
    }
}
```

### Integration Tests

```kotlin
@Test
fun testModelDownloadAndGeneration() = runTest {
    val modelId = RunAnywhere.registerModel(
        name = "Test Model",
        url = "https://example.com/model.gguf",
        framework = InferenceFramework.LLAMA_CPP
    ).id

    RunAnywhere.downloadModel(modelId).collect { /* progress */ }
    RunAnywhere.loadLLMModel(modelId)

    val response = RunAnywhere.chat("Test prompt")
    assertTrue(response.isNotEmpty())
}
```

## Resources

- Full Documentation: [docs.runanywhere.ai/kotlin](https://docs.runanywhere.ai/kotlin/introduction)
- Source Code: [sdk/runanywhere-kotlin/](https://github.com/RunanywhereAI/runanywhere-sdks/tree/main/sdk/runanywhere-kotlin)
- Example App: [examples/android/RunAnywhereAI/](https://github.com/RunanywhereAI/runanywhere-sdks/tree/main/examples/android/RunAnywhereAI)
- Discord: [discord.gg/N359FBbDVd](https://discord.gg/N359FBbDVd)
