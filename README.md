# RunAnywhere AI Skill for Claude Code

A comprehensive skill for integrating [RunAnywhere](https://github.com/RunanywhereAI/runanywhere-sdks) on-device AI into your applications across all supported platforms.

## Overview

This skill enables Claude Code to assist with integrating RunAnywhere's privacy-first, on-device AI capabilities into Swift (iOS/macOS), Kotlin (Android), Web (WebAssembly), React Native, and Flutter applications.

### What is RunAnywhere?

RunAnywhere lets you add AI features to your app that run entirely on-device:
- **LLM Text Generation** - Run Llama, Mistral, Qwen, SmolLM locally via llama.cpp
- **Speech-to-Text** - Whisper-based transcription
- **Text-to-Speech** - Neural voice synthesis via Piper
- **Voice Agent Pipeline** - Complete VAD → STT → LLM → TTS orchestration
- **Vision Language Models** - Visual understanding (iOS/Web only)

All processing happens locally—no cloud, no latency, no data leaves the device.

## Features

### Comprehensive Platform Coverage

-  **Swift (iOS/macOS)** - Complete integration guide with Swift Package Manager setup
-  **Kotlin (Android)** - Full Gradle configuration and usage examples
-  **Web (Browser)** - WebAssembly setup with bundler configuration
-  **React Native** - Cross-platform mobile integration
-  **Flutter** - Dart/Flutter implementation guide

### Deep Integration Guidance

- **Installation & Setup** - Platform-specific dependency management
- **Model Selection** - Device-appropriate model recommendations with quantization guidance
- **Complete API Reference** - All SDK methods with working code examples
- **Error Handling** - Common issues and solutions for each platform
- **Performance Optimization** - Memory management, streaming patterns, best practices
- **Voice Agent Pipelines** - Complete STT → LLM → TTS workflows

### Progressive Disclosure Design

The skill uses a three-tier information architecture:
1. **SKILL.md** - Core workflow and platform selection (~270 lines)
2. **Platform Guides** - Detailed references loaded on-demand (swift.md, kotlin.md, etc.)
3. **Model Guide** - Comprehensive model selection and optimization guidance

This minimizes context usage while providing complete information when needed.

## Installation

### For Claude Code Users

1. Download the packaged skill: `runanywhere-ai.skill`
2. Install via Claude Code CLI:
   ```bash
   claude-code install runanywhere-ai.skill
   ```
3. The skill will automatically trigger when working with:
   - On-device AI features
   - Local LLM inference
   - RunAnywhere SDK integration
   - GGUF models or llama.cpp
   - Offline AI processing

### From Source

```bash
# Clone the repository
git clone https://github.com/josuediazflores/runanywhere-skill.git

# Install from directory
claude-code install runanywhere-skill/
```

## Structure

```
runanywhere-ai/
├── SKILL.md                          # Main skill file with core workflow
├── references/                       # Platform-specific detailed guides
│   ├── swift.md                     # iOS/macOS integration (781 lines)
│   ├── kotlin.md                    # Android integration (573 lines)
│   ├── web.md                       # Browser/WebAssembly guide (795 lines)
│   ├── react-native.md              # React Native guide (743 lines)
│   ├── flutter.md                   # Flutter guide (800 lines)
│   └── models.md                    # Model selection guide (450 lines)
└── README.md                        # This file
```

## Usage Examples

### Example 1: Integrate LLM into Swift iOS App

```
User: "Help me integrate RunAnywhere's LLM into my Swift iOS app"

Claude:
1. Reads swift.md reference
2. Provides Swift Package Manager setup
3. Shows SDK initialization with LlamaCPP registration
4. Demonstrates model download and loading
5. Provides text generation examples
6. Suggests streaming for better UX
```

### Example 2: Set Up Speech-to-Text in React Native

```
User: "Set up speech-to-text in my React Native app"

Claude:
1. Reads react-native.md reference
2. Shows npm installation for @runanywhere/core and @runanywhere/onnx
3. Demonstrates ONNX module registration
4. Shows Whisper model setup
5. Provides transcription examples with progress tracking
```

### Example 3: Build Complete Voice Assistant on Android

```
User: "Build a voice assistant pipeline on Android"

Claude:
1. Reads kotlin.md reference
2. Shows Gradle dependencies for LlamaCPP and ONNX
3. Demonstrates voice agent configuration
4. Shows voice session handling with events
5. Provides complete implementation example
```

## Key Concepts Covered

### Quantization

Models are compressed using quantization:
- **Q4_0**: Smallest size, fastest, lower quality (~3.5 bits/weight)
- **Q5_K_M**: Balanced size and quality (~5.5 bits/weight)
- **Q8_0**: Largest size, best quality, slower (~8 bits/weight)

### Model Formats

- **LLM**: GGUF format (via llama.cpp)
- **STT**: ONNX format (Whisper models)
- **TTS**: ONNX format (Piper voices)
- **VAD**: ONNX format (Silero VAD)

### Memory Requirements

Rule of thumb: Device RAM should be 2× model size
- SmolLM2 360M (~400MB) → Need 800MB+ RAM
- Llama 3.2 1B (~1GB) → Need 2GB+ RAM
- Mistral 7B (~4GB) → Need 8GB+ RAM

## Development & Testing

### Skill Creation Process

This skill was created following the [Claude Code Skill Creator](https://github.com/anthropics/claude-code/tree/main/skills/skill-creator) guidelines:

1. **Understanding Phase** - Analyzed RunAnywhere SDKs across all platforms
2. **Planning Phase** - Identified reusable resources (platform guides, model selection)
3. **Implementation Phase** - Created comprehensive reference documentation
4. **Validation Phase** - Audited all APIs against official RunAnywhere documentation
5. **Iteration Phase** - Fixed discrepancies and removed misaligned components

### Quality Assurance

All API methods and code examples were verified against:
- [Official RunAnywhere Repository](https://github.com/RunanywhereAI/runanywhere-sdks)
- Platform-specific README files
- CLAUDE.md repository guidelines

**Corrections Made:**
- Fixed Swift API method names (transcribe, synthesize, voice agent)
- Removed misaligned download script (SDKs have built-in download)
- Verified all platform APIs match official documentation

## Accuracy & Maintenance

### Source Documentation

All information is derived from official sources:
- [RunAnywhere GitHub Repository](https://github.com/RunanywhereAI/runanywhere-sdks)
- [Swift SDK Documentation](https://docs.runanywhere.ai/swift/introduction)
- [Kotlin SDK Documentation](https://docs.runanywhere.ai/kotlin/introduction)
- [React Native Documentation](https://docs.runanywhere.ai/react-native/introduction)
- [Flutter Documentation](https://docs.runanywhere.ai/flutter/introduction)

### Keeping Up-to-Date

As RunAnywhere SDKs evolve, this skill should be updated:
1. Monitor [RunAnywhere Releases](https://github.com/RunanywhereAI/runanywhere-sdks/releases)
2. Check for API changes in platform-specific READMEs
3. Update reference files as needed
4. Re-validate against official documentation

## Contributing

Contributions are welcome! To improve this skill:

1. **Report Issues** - Found incorrect API usage? [Open an issue](https://github.com/josuediazflores/runanywhere-skill/issues)
2. **Update Documentation** - SDK changed? Submit a pull request
3. **Add Examples** - Have useful patterns? Share them

### Validation Checklist

Before submitting updates:
- [ ] Verify APIs against [official RunAnywhere docs](https://github.com/RunanywhereAI/runanywhere-sdks)
- [ ] Test code examples when possible
- [ ] Keep platform guides under 1000 lines (progressive disclosure)
- [ ] Maintain consistent formatting and structure
- [ ] Update README.md if structure changes

## License

This skill is licensed under Apache 2.0, matching the RunAnywhere SDK license.

## Resources

### Official RunAnywhere Resources

- **Website**: [runanywhere.ai](https://runanywhere.ai)
- **Documentation**: [docs.runanywhere.ai](https://docs.runanywhere.ai)
- **GitHub**: [github.com/RunanywhereAI/runanywhere-sdks](https://github.com/RunanywhereAI/runanywhere-sdks)
- **Discord**: [discord.gg/N359FBbDVd](https://discord.gg/N359FBbDVd)

### Example Applications

- iOS: [examples/ios/RunAnywhereAI](https://github.com/RunanywhereAI/runanywhere-sdks/tree/main/examples/ios/RunAnywhereAI)
- Android: [examples/android/RunAnywhereAI](https://github.com/RunanywhereAI/runanywhere-sdks/tree/main/examples/android/RunAnywhereAI)
- Web: [examples/web/RunAnywhereAI](https://github.com/RunanywhereAI/runanywhere-sdks/tree/main/examples/web/RunAnywhereAI)
- React Native: [examples/react-native/RunAnywhereAI](https://github.com/RunanywhereAI/runanywhere-sdks/tree/main/examples/react-native/RunAnywhereAI)
- Flutter: [examples/flutter/RunAnywhereAI](https://github.com/RunanywhereAI/runanywhere-sdks/tree/main/examples/flutter/RunAnywhereAI)

### Claude Code Resources

- **Skill Creator Guide**: [claude-code/skills/skill-creator](https://github.com/anthropics/claude-code/tree/main/skills/skill-creator)
- **Claude Code Documentation**: [claude.ai/code](https://claude.ai/code)

## Support

For issues specific to this skill:
- Open an issue on [GitHub Issues](https://github.com/josuediazflores/runanywhere-skill/issues)

For RunAnywhere SDK questions:
- Join [RunAnywhere Discord](https://discord.gg/N359FBbDVd)
- Open an issue on [RunAnywhere GitHub](https://github.com/RunanywhereAI/runanywhere-sdks/issues)

---

**Built with** ❤️ **using [Claude Code](https://claude.ai/code)**
