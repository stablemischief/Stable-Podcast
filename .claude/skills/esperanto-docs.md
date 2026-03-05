# Skill: Esperanto AI Library Reference

## Purpose

Comprehensive API reference for the `esperanto` library (v2.19.4) by lfnovo. Esperanto is a unified interface for 16+ AI model providers across five capability domains: LLM, Embeddings, Reranking, STT, and TTS. It is the AI provider abstraction layer used by `podcast-creator`.

## Installation

```bash
pip install esperanto                     # Core
pip install "esperanto[transformers]"     # HuggingFace local models
pip install "esperanto[validation]"       # JSON schema validation
```

**Requires:** Python >= 3.10 and < 3.14
**Core Dependencies:** `pydantic>=2.0.0`, `httpx>=0.28.0` (no vendor SDKs)

## Provider Capability Matrix

| Provider | Env Variable | LLM | Embedding | Reranking | STT | TTS |
|---|---|---|---|---|---|---|
| OpenAI | `OPENAI_API_KEY` | Yes | Yes | -- | Yes | Yes |
| Anthropic | `ANTHROPIC_API_KEY` | Yes | -- | -- | -- | -- |
| Google GenAI | `GOOGLE_API_KEY` / `GEMINI_API_KEY` | Yes | Yes | -- | Yes | Yes |
| Groq | `GROQ_API_KEY` | Yes | -- | -- | Yes | -- |
| Mistral | `MISTRAL_API_KEY` | Yes | Yes | -- | -- | -- |
| DeepSeek | `DEEPSEEK_API_KEY` | Yes | -- | -- | -- | -- |
| xAI | `XAI_API_KEY` | Yes | -- | -- | -- | -- |
| Perplexity | `PERPLEXITY_API_KEY` | Yes | -- | -- | -- | -- |
| OpenRouter | `OPENROUTER_API_KEY` | Yes | -- | -- | -- | -- |
| Ollama | `OLLAMA_BASE_URL` | Yes | Yes | -- | -- | -- |
| Azure OpenAI | `AZURE_OPENAI_API_KEY` + `_ENDPOINT` + `_API_VERSION` | Yes | Yes | -- | Yes | Yes |
| Vertex AI | `GOOGLE_APPLICATION_CREDENTIALS` + `VERTEX_PROJECT` + `VERTEX_LOCATION` | -- | Yes | -- | -- | Yes |
| ElevenLabs | `ELEVENLABS_API_KEY` | -- | -- | -- | Yes | Yes |
| Jina | `JINA_API_KEY` | -- | Yes | Yes | -- | -- |
| Voyage | `VOYAGE_API_KEY` | -- | Yes | Yes | -- | -- |
| Transformers | `HF_TOKEN` | -- | Yes | Yes | -- | -- |
| OpenAI-Compatible | `OPENAI_COMPATIBLE_BASE_URL` + `_API_KEY` | Yes | Yes | -- | Yes | Yes |

## The AIFactory — Central Entry Point

```python
from esperanto.factory import AIFactory

# Language Model
model = AIFactory.create_language("openai", "gpt-4")
model = AIFactory.create_language("anthropic", "claude-3-5-sonnet-20241022")
model = AIFactory.create_language("google", "gemini-2.0-flash")
model = AIFactory.create_language("groq", "llama-3.3-70b-versatile")
model = AIFactory.create_language("ollama", "llama3.1")
model = AIFactory.create_language("openai-compatible", "model", config={"base_url": "http://localhost:1234/v1"})

# Embedding
embedder = AIFactory.create_embedding("openai", "text-embedding-3-small")
embedder = AIFactory.create_embedding("jina", "jina-embeddings-v3")

# Reranker
reranker = AIFactory.create_reranker("jina", "jina-reranker-v2-base-multilingual")

# Speech-to-Text
transcriber = AIFactory.create_speech_to_text("openai", "whisper-1")
transcriber = AIFactory.create_speech_to_text("groq", "whisper-large-v3")

# Text-to-Speech
speaker = AIFactory.create_text_to_speech("openai", "tts-1")
speaker = AIFactory.create_text_to_speech("elevenlabs", "eleven_multilingual_v2")
```

### Factory Signature Pattern

```python
AIFactory.create_<capability>(
    provider: str,          # "openai", "anthropic", etc.
    model_name: str,        # Model identifier
    config: dict = None,    # Optional configuration overrides
    **kwargs
)
```

### Model Discovery

```python
models = AIFactory.get_provider_models("openai")
models = AIFactory.get_provider_models("openai", model_type="language")
providers = AIFactory.get_available_providers()  # Dict by capability type
```

## Language Models (LLM)

### Methods

```python
response = model.chat_complete(messages, **kwargs)        # Sync
response = await model.achat_complete(messages, **kwargs) # Async
```

### Message Format (OpenAI-compatible)

```python
messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Hello!"},
]
```

### Configuration Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `temperature` | float | 1.0 | Randomness (0.0 - 2.0) |
| `max_tokens` | int | Model default | Maximum response length |
| `top_p` | float | 1.0 | Nucleus sampling |
| `streaming` | bool | False | Token-by-token delivery |
| `structured` | dict | None | JSON output: `{"type": "json"}` |
| `timeout` | float | 60.0 | Request timeout (seconds) |
| `base_url` | str | Provider default | Custom endpoint |
| `api_key` | str | Env variable | API key override |

**Priority:** Per-request kwargs > config dict > provider defaults > env vars

### Response Object (ChatResponse)

```python
response.content                          # Shortcut to text
response.choices[0].message.content       # Full path
response.choices[0].message.role          # "assistant"
response.usage.prompt_tokens              # int
response.usage.completion_tokens          # int
response.usage.total_tokens              # int
response.model                            # str
```

### Streaming

```python
model = AIFactory.create_language("openai", "gpt-4", config={"streaming": True})
for chunk in model.chat_complete(messages):
    print(chunk.choices[0].delta.content, end="")

# Async
async for chunk in await model.achat_complete(messages):
    print(chunk.choices[0].delta.content, end="")
```

### Structured Output (JSON Mode)

```python
model = AIFactory.create_language("openai", "gpt-4", config={"structured": {"type": "json"}})
response = model.chat_complete([{"role": "user", "content": "List 3 colors as JSON"}])
# response.content is valid JSON
```

### Tool Calling

```python
tools = [{
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "Get weather",
        "parameters": {
            "type": "object",
            "properties": {"location": {"type": "string"}},
            "required": ["location"]
        }
    }
}]
response = model.chat_complete(messages, tools=tools)
```

### Reasoning Trace Handling

```python
response = model.chat_complete(messages)
message = response.choices[0].message
message.thinking           # Extracted from <think> tags (or None)
message.cleaned_content    # Response with <think> tags removed
```

### LangChain Integration

```python
langchain_model = model.to_langchain()
# Usable in LangChain chains, agents, RAG pipelines
```

## Text-to-Speech (TTS)

### Methods

```python
audio_bytes = speaker.generate_speech(text, voice=None, **kwargs)         # Sync
audio_bytes = await speaker.agenerate_speech(text, voice=None, **kwargs)  # Async
```

### Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `text` | str | Required | Text to convert |
| `voice` | str | Provider default | Voice identifier |
| `timeout` | float | 300.0 | Request timeout |
| `speed` | float | 1.0 | Speech rate (0.25 - 4.0) |
| `response_format` | str | "mp3" | mp3, opus, aac, flac, wav, pcm |

### Voices by Provider

- **OpenAI:** alloy, echo, fable, onyx, nova, shimmer
- **ElevenLabs:** Custom voice IDs (see TTS provider reference skill)
- **Google/Vertex:** Language-regional variants (e.g., `en-US-Neural2-A`)

### ElevenLabs-Specific Config

| Parameter | Range | Description |
|---|---|---|
| `stability` | 0.0 - 1.0 | Voice consistency |
| `similarity_boost` | 0.0 - 1.0 | Voice resemblance |
| `style` | 0.0 - 1.0 | Style exaggeration |
| `use_speaker_boost` | bool | Enhance clarity |

### Usage

```python
speaker = AIFactory.create_text_to_speech("openai", "tts-1")
audio = speaker.generate_speech("Hello world!", voice="nova")
with open("output.mp3", "wb") as f:
    f.write(audio)
```

## Embeddings

```python
embedder = AIFactory.create_embedding("openai", "text-embedding-3-small")
response = embedder.embed(["Sample text"])
response.data[0].embedding    # list[float]
response.usage.total_tokens   # int
```

### Task Types
`RETRIEVAL_QUERY`, `RETRIEVAL_DOCUMENT`, `SIMILARITY`, `CLASSIFICATION`, `CLUSTERING`, `CODE_RETRIEVAL`, `QUESTION_ANSWERING`, `FACT_VERIFICATION`

## Reranking

```python
reranker = AIFactory.create_reranker("jina", "jina-reranker-v2-base-multilingual")
results = reranker.rerank("query", documents, top_k=5)
for r in results:
    r.document, r.relevance_score, r.index
```

## Speech-to-Text (STT)

```python
transcriber = AIFactory.create_speech_to_text("groq", "whisper-large-v3")
result = transcriber.transcribe("interview.mp3", language="en")
print(result.text)
```

## Timeout Configuration

| Capability | Default | Env Variable |
|---|---|---|
| LLM | 60s | `ESPERANTO_LLM_TIMEOUT` |
| Embeddings | 60s | `ESPERANTO_EMBEDDING_TIMEOUT` |
| Reranking | 60s | `ESPERANTO_RERANKER_TIMEOUT` |
| STT | 300s | `ESPERANTO_STT_TIMEOUT` |
| TTS | 300s | `ESPERANTO_TTS_TIMEOUT` |

Valid range: 1 - 3600 seconds.

## Resource Management

```python
# Context managers (recommended)
with AIFactory.create_language("openai", "gpt-4") as model:
    response = model.chat_complete(messages)

async with AIFactory.create_language("openai", "gpt-4") as model:
    response = await model.achat_complete(messages)

# Manual cleanup
model.close()          # sync
await model.aclose()   # async
```

## Provider-Specific Notes

- **Anthropic:** `max_tokens` is **required**; 200K context window
- **Google GenAI:** 2M token context with Gemini; native task types for embeddings; accepts `GOOGLE_API_KEY` or `GEMINI_API_KEY`
- **Groq:** Ultra-fast LPU inference; LLM + STT only
- **Ollama:** Fully local, no API key; default 128K context
- **OpenAI-Compatible:** Universal adapter for any OpenAI-API-compatible endpoint (LM Studio, vLLM, Speaches, etc.)
- **Azure OpenAI:** `model_name` = your **deployment name**, not model ID; supports modality-specific endpoint/key overrides
- **ElevenLabs:** Premium TTS with voice cloning and emotional control; STT with auto language detection
- **Perplexity:** LLM with built-in web search; unique params: `search_domain_filter`, `search_recency_filter`

## Credential Priority Order

1. Direct parameters in code (highest)
2. Capability-specific env vars (e.g., `AZURE_OPENAI_API_KEY_LLM`)
3. Generic env vars (e.g., `AZURE_OPENAI_API_KEY`)
4. Provider defaults (lowest)

## How podcast-creator Uses Esperanto

1. **Outline Generation:** `AIFactory.create_language(provider, model, config={...}).to_langchain()`
2. **Transcript Generation:** Same pattern, different provider/model
3. **Audio Synthesis:** `AIFactory.create_text_to_speech(provider, model, **tts_config)` → `await tts.agenerate_speech(text, voice, output_file)`

Per-speaker TTS overrides allow mixing providers within a single podcast (e.g., some speakers on ElevenLabs, others on OpenAI).
