# SmolChatAndroidLib

A lightweight Android library for running GGUF LLMs on-device using **llama.cpp** via native JNI bindings. Forked from [Shubham Panchal's SmolChat-Android](https://github.com/shubham0204/SmolChat-Android) and published as a reusable library via Jitpack.

📦 Licensed under **Apache-2.0**  
⚡ Optimized for ARM64 with runtime CPU feature detection (FP16, DotProd, SVE, I8MM)  
💬 Streaming & non-streaming inference support  
🧠 Configurable chat templates and context management

---

## Features

- **GGUF Support:** Load and run standard GGUF models directly on Android
- **Smart Native Selection:** Automatically detects CPU features at runtime and loads the optimal native library (`v8_4`, `v8_2`, `v7a`, or fallback)
- **Streaming API:** Get responses as a Kotlin `Flow<String>` for real-time token streaming
- **Chat History:** Manage user/system/assistant messages with optional persistence
- **Jinja2 Templates:** Full support for Jinja2 chat templates (with sensible defaults)
- **Configurable Inference:** Control temperature, minP, threads, context size, mmap/mlock settings
- **Benchmarking:** Built-in model benchmarking utility (`benchModel`)
- **Coroutine-Friendly:** `load()` is a suspending function for easy integration

---

## Installation

### Step 1: Add Jitpack repository

In your root `settings.gradle` (or `build.gradle`):

```groovy
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        mavenCentral()
        maven { url 'https://jitpack.io' }
    }
}
```

### Step 2: Add the dependency

In your module's `build.gradle`:

```groovy
dependencies {
    implementation 'com.github.woheller69:SmolChatAndroidLib:V1.5'
}
```

---

## Quick Start

```kotlin
class MainActivity : AppCompatActivity() {

    private val smolLM = SmolLM()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val modelFile = File(getExternalFilesDir(null), "model.gguf")
        
        lifecycleScope.launch(Dispatchers.IO) {
            try {
                smolLM.load(modelFile.absolutePath, SmolLM.InferenceParams(contextSize = 2048))
                
                val response = smolLM.getResponse("Hello!")  // Runs on IO thread
                
                withContext(Dispatchers.Main) {
                    textView.text = response  // UI update on Main
                }
            } catch (e: Exception) {
                withContext(Dispatchers.Main) {
                    textView.text = "Error: ${e.message}"
                }
            }
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        smolLM.close()
    }
}

```

### Non-Streaming Responses

```kotlin
// Returns complete response as a String (blocking on IO thread)
val response = smolLM.getResponse("What is Android?")
```

### Streaming Responses

```kotlin
// Emits tokens one by one as Flow<String>
lifecycleScope.launch {
    smolLM.getResponseAsFlow("Your prompt here")
        .collect { token ->
            // Update UI with each token
            textView.append(token)
        }
}
```

Always call `close()` in `onDestroy()` to release native resources.


## API Reference

### SmolLM

| Method | Description |
|--------|-------------|
| `suspend fun load(modelPath, params)` | Load a GGUF model. Returns when model is ready. |
| `fun getResponse(query: String): String` | Get complete response (blocking). |
| `fun getResponseAsFlow(query: String): Flow<String>` | Stream tokens as they're generated. Emits `[EOG]` at end. |
| `fun addUserMessage(message: String)` | Add user message to chat history. |
| `fun addSystemPrompt(prompt: String)` | Set system prompt. |
| `fun addAssistantMessage(message: String)` | Add assistant response to history. |
| `fun getResponseGenerationSpeed(): Float` | Get tokens/sec from last inference. |
| `fun getContextLengthUsed(): Int` | Get current context window usage. |
| `fun benchModel(pp, tg, pl, nr): String` | Run benchmark and return timings. |
| `fun close()` | Unload model and release native resources. |

### InferenceParams

```kotlin
data class InferenceParams(
    val minP: Float = 0.1f,           // Minimum token probability
    val temperature: Float = 0.8f,     // Sampling temperature
    val storeChats: Boolean = true,    // Persist chat history
    val contextSize: Long? = null,     // Override model context size
    val chatTemplate: String? = null,  // Jinja2 template override
    val numThreads: Int = 4,           // CPU threads for inference
    val useMmap: Boolean = true,       // Memory-map the model file
    val useMlock: Boolean = false      // Lock model in RAM
)
```

## Project Structure

This library is extracted from the [SmolChat-Android](https://github.com/shubham0204/SmolChat-Android) monorepo:

- **smollm module**: Core inference engine wrapping llama.cpp
  - `SmolLM.kt` — Kotlin API for JNI bindings
  - `llama_inference.cpp` — C++ inference logic
  - `smollm.cpp` — JNI binding layer
- Supports all standard GGUF formats and Hugging Face model architectures

---

## License

This project is licensed under the Apache License 2.0 — see [LICENSE](LICENSE) for details.

**Upstream Attribution:**  
Original work by [Shubham Panchal](https://github.com/shubham0204).  
Powered by [llama.cpp](https://github.com/ggerganov/llama.cpp).

---
