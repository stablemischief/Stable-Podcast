# Skill: TTS Provider API Reference

## Purpose

Quick-reference for all text-to-speech providers relevant to podcast generation. Covers API endpoints, voices, parameters, pricing, rate limits, and Python SDK usage for OpenAI, ElevenLabs, Google Cloud TTS, and Speaches (local).

---

## 1. OpenAI TTS

### Models

| Model | Description | Notes |
|---|---|---|
| `tts-1` | Standard quality, low latency | 6 classic voices |
| `tts-1-hd` | Higher quality | 6 classic voices |
| `gpt-4o-mini-tts` | Latest, steerable | All 13 voices; `instructions` param |

### Voices

| Voice | Availability |
|---|---|
| alloy, echo, fable, onyx, nova, shimmer | All models (classic 6) |
| ash, ballad, coral, sage, verse, marin, cedar | `gpt-4o-mini-tts` only |

### Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `model` | string | Yes | `tts-1`, `tts-1-hd`, or `gpt-4o-mini-tts` |
| `input` | string | Yes | Text to synthesize (max 4096 chars) |
| `voice` | string | Yes | Voice identifier |
| `response_format` | string | No | `mp3` (default), `opus`, `aac`, `flac`, `wav`, `pcm` |
| `speed` | float | No | `0.25` to `4.0`, default `1.0` |
| `instructions` | string | No | **gpt-4o-mini-tts only.** Controls tone, accent, emotion |

### Pricing

| Model | Price |
|---|---|
| `tts-1` | $15.00 / 1M characters |
| `tts-1-hd` | $30.00 / 1M characters |
| `gpt-4o-mini-tts` | $0.60/1M input tokens + $12.00/1M audio output tokens |

### Python SDK

```python
from openai import OpenAI
client = OpenAI()

response = client.audio.speech.create(
    model="gpt-4o-mini-tts",
    voice="coral",
    input="Hello, welcome to our podcast!",
    response_format="mp3",
    instructions="Speak in a warm, conversational tone.",
)
response.stream_to_file("output.mp3")

# Streaming
with client.audio.speech.with_streaming_response.create(
    model="gpt-4o-mini-tts", voice="coral",
    input="Streaming example.", response_format="pcm",
) as response:
    for chunk in response.iter_bytes(chunk_size=1024):
        pass  # process chunks
```

### Gotchas
- Max 4096 chars per request — split longer content
- `instructions` only works on `gpt-4o-mini-tts`
- `speed` has NO effect on `gpt-4o-mini-tts` — use `instructions` instead

---

## 2. ElevenLabs

### Models

| Model ID | Latency | Languages | Max Chars |
|---|---|---|---|
| `eleven_v3` | Standard | 70+ | 5,000 |
| `eleven_multilingual_v2` | Standard | 29 | 10,000 |
| `eleven_flash_v2_5` | ~75ms | 32 | 40,000 |
| `eleven_turbo_v2_5` | ~250ms | 32 | 40,000 |

### Key Premade Voices

| Voice | Voice ID | Use Case |
|---|---|---|
| Rachel | `21m00Tcm4TlvDq8ikWAM` | Narration |
| Adam | `pNInz6obpgDQGcFmaJgB` | Narration |
| Josh | `TxGEqnHWrfWFTfGW9XjX` | Narration |
| Brian | `nPczCjzI2devNBz1zQrb` | Narration |
| George | `JBFqnCBsd6RMkjVDRZzb` | Narration |
| Matilda | `XrExE9yKIg1WjnnlVkGX` | Audiobook |
| Daniel | `onwK4e9ZLuTAKqWW03F9` | News |
| Charlie | `IKne3meq5aSn9XLyUdCD` | Conversational |

### Audio Formats

`mp3_44100_128` (default), `mp3_22050_32`, `mp3_44100_64`, `mp3_44100_96`, `mp3_44100_192`, `pcm_16000`, `pcm_22050`, `pcm_24000`, `pcm_44100`

### Voice Settings

| Parameter | Range | Description |
|---|---|---|
| `stability` | 0.0 - 1.0 | Lower = more expressive |
| `similarity_boost` | 0.0 - 1.0 | Match original voice |
| `style` | 0.0 - 1.0 | Style exaggeration (increases latency) |
| `use_speaker_boost` | bool | Enhance similarity |
| `speed` | float | Speech rate |

### Additional Parameters
- `previous_text` / `next_text` — for multi-chunk continuity
- `seed` — 0-4294967295 for deterministic output
- `language_code` — ISO 639-1 to force language

### Pricing

| Plan | Monthly | Included Chars |
|---|---|---|
| Free | $0 | 10,000 |
| Starter | ~$5 | 30,000 |
| Creator | ~$11 | 100,000 |
| Pro | ~$82.50 | 500,000 |
| Scale | ~$275 | 2,000,000 |

### Python SDK

```python
from elevenlabs.client import ElevenLabs
from elevenlabs import VoiceSettings

client = ElevenLabs(api_key="your-key")  # or ELEVEN_API_KEY env var

audio = client.text_to_speech.convert(
    text="Welcome to our podcast.",
    voice_id="21m00Tcm4TlvDq8ikWAM",
    model_id="eleven_multilingual_v2",
    output_format="mp3_44100_128",
    voice_settings=VoiceSettings(
        stability=0.5, similarity_boost=0.75,
        style=0.0, use_speaker_boost=True,
    ),
)
with open("output.mp3", "wb") as f:
    for chunk in audio:
        f.write(chunk)

# List voices
voices = client.voices.get_all()
for v in voices.voices:
    print(f"{v.name}: {v.voice_id}")
```

### Gotchas
- Voice IDs are opaque strings — look up via API
- Max chars differ by model (5K for v3, 40K for flash)
- `previous_text`/`next_text` crucial for multi-chunk prosody
- Flash/Turbo cost 0.5 credits/char; Multilingual v2 costs 1 credit/char

---

## 3. Google Cloud TTS

### Voice Types

| Type | Quality | Price/1M chars |
|---|---|---|
| Standard | Basic | $4.00 |
| WaveNet | Premium | $16.00 |
| Neural2 | Premium | $16.00 |
| Studio | Premium+ | $30-160 |
| Chirp 3 HD | Premium+ | ~$30 |

### Voice Naming: `{lang}-{VoiceType}-{Letter}`
Examples: `en-US-Standard-A`, `en-US-Wavenet-D`, `en-US-Neural2-A`, `en-US-Studio-O`

### SSML Support
`<speak>`, `<break>`, `<emphasis>`, `<prosody>`, `<say-as>`, `<sub>`, `<audio>`, `<par>`, `<seq>`

### Parameters

| Parameter | Description |
|---|---|
| `input.text` / `input.ssml` | Text or SSML content |
| `voice.languageCode` | BCP-47 code (e.g., `en-US`) |
| `voice.name` | Specific voice name |
| `audioConfig.audioEncoding` | `MP3`, `LINEAR16`, `OGG_OPUS`, `MULAW` |
| `audioConfig.speakingRate` | 0.25 to 4.0 |
| `audioConfig.pitch` | -20.0 to 20.0 semitones |

### Rate Limits
- 1,000 RPM per project (increasable)
- Max 5,000 chars per request

### Free Tier
- 4M Standard chars/month
- 1M WaveNet/Neural2 chars/month

### Python SDK

```python
from google.cloud import texttospeech

client = texttospeech.TextToSpeechClient()

synthesis_input = texttospeech.SynthesisInput(text="Welcome to our podcast.")
voice = texttospeech.VoiceSelectionParams(
    language_code="en-US", name="en-US-Neural2-D",
)
audio_config = texttospeech.AudioConfig(
    audio_encoding=texttospeech.AudioEncoding.MP3,
)
response = client.synthesize_speech(
    input=synthesis_input, voice=voice, audio_config=audio_config,
)
with open("output.mp3", "wb") as f:
    f.write(response.audio_content)
```

### Gotchas
- Auth setup more involved (GCP project, service account, `GOOGLE_APPLICATION_CREDENTIALS`)
- SSML markup characters count toward billing
- Studio voices limited to a few languages

---

## 4. Speaches (Local OpenAI-Compatible)

### Overview
Self-hosted, OpenAI-compatible TTS via **Kokoro** (82M params, #1 on TTS Arena) and **Piper** backends. MIT license, no API costs, no rate limits.

### Setup

```bash
# Docker (GPU)
docker compose -f compose.cuda.yaml up

# Docker (CPU)
docker compose -f compose.cpu.yaml up

# Download models
uvx speaches-cli model download speaches-ai/Kokoro-82M-v1.0-ONNX
```

### Voices (Kokoro)
`af_alloy`, `af_aoede`, `af_bella`, `af_heart`, `af_jessica`, `af_nicole`, `af_nova`, `af_river`, `af_sarah`, `af_sky` (26 total)

Naming: `a`=American, `b`=British, `f`=female, `m`=male

### Supported Formats
`mp3`, `wav` only (no opus, no aac)

### Python SDK (uses standard OpenAI SDK)

```python
from openai import OpenAI

client = OpenAI(
    api_key="not-needed-but-required",
    base_url="http://localhost:8000/v1",
)
response = client.audio.speech.create(
    model="speaches-ai/Kokoro-82M-v1.0-ONNX",
    voice="af_heart",
    input="Hello from local TTS!",
    response_format="mp3",
)
response.stream_to_file("output.mp3")
```

### Gotchas
- OpenAI SDKs require a non-empty API key (use any dummy string)
- GPU recommended for production
- Only 8 languages (vs 29-99+ for cloud providers)

---

## Quick Comparison

| Feature | OpenAI | ElevenLabs | Google Cloud | Speaches |
|---|---|---|---|---|
| Best Model | `gpt-4o-mini-tts` | `eleven_v3` | Neural2/Chirp 3 | Kokoro-82M |
| Voices | 13 | 40+ premade | 380+ | ~26 |
| Languages | 99+ | 29-70+ | 75+ | 8 |
| Max Input | 4,096 chars | 5K-40K chars | 5,000 chars | No limit |
| Free Tier | None | 10K chars/mo | 4M Standard/mo | Unlimited |
| Cheapest | $15/1M chars | ~$120/1M chars | $4/1M chars | Free |
| Voice Cloning | No | Yes | No | No |
| SSML | No | No | Yes | No |
| Steerability | `instructions` | `voice_settings` | SSML/prosody | `speed` only |
| Self-Hosted | No | No | No | Yes |
| Python SDK | `openai` | `elevenlabs` | `google-cloud-texttospeech` | `openai` |
