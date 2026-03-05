# Skill: podcast-creator Library Reference

## Purpose

Comprehensive API reference for the `podcast-creator` library (v0.12.0) by lfnovo. This library powers the podcast generation pipeline — it orchestrates outline generation, transcript creation, TTS audio synthesis, and audio mixing via a LangGraph workflow.

## Installation

```bash
pip install podcast-creator       # Core library
pip install podcast-creator[ui]   # With Streamlit web UI
```

## Environment Variables

| Variable | Purpose |
|---|---|
| `OPENAI_API_KEY` | OpenAI LLM and TTS |
| `ANTHROPIC_API_KEY` | Anthropic Claude models |
| `ELEVENLABS_API_KEY` | ElevenLabs TTS voices |
| `GOOGLE_API_KEY` | Google AI/TTS |
| `GROQ_API_KEY` | Groq inference |
| `XAI_API_KEY` | xAI models |
| `DEEPSEEK_API_KEY` | DeepSeek models |
| `MISTRAL_API_KEY` | Mistral models |
| `PERPLEXITY_API_KEY` | Perplexity models |
| `OPENROUTER_API_KEY` | OpenRouter gateway |
| `OLLAMA_BASE_URL` | Ollama local endpoint (default: `http://127.0.0.1:11434`) |
| `TTS_BATCH_SIZE` | Concurrent TTS clips per batch (default: `5`) |
| `PODCAST_RETRY_MAX_ATTEMPTS` | Max retry attempts (default: `3`) |
| `PODCAST_RETRY_WAIT_MULTIPLIER` | Exponential backoff multiplier in seconds (default: `5`) |
| `PODCAST_RETRY_WAIT_MAX` | Maximum wait between retries in seconds (default: `30`) |

## Top-Level Exports

```python
from podcast_creator import (
    # Configuration
    configure, get_config, PodcastConfig,
    # Data models
    Segment, Outline, Dialogue, Transcript,
    # Utility functions
    combine_audio_files, extract_text_content,
    parse_thinking_content, clean_thinking_content,
    # LangGraph workflow
    create_podcast, podcast_graph, PodcastState,
    # Speaker models
    Speaker, SpeakerProfile, SpeakerConfig, load_speaker_config,
    # Episode models
    EpisodeProfile, EpisodeConfig, load_episode_config,
    # Language utilities
    resolve_language_name,
)
```

## Main Entry Point: `create_podcast()`

```python
async def create_podcast(
    content: Union[str, List[str]],          # Source text or list of text pieces
    briefing: Optional[str] = None,          # Instructions for tone/style
    episode_name: Optional[str] = None,      # Required: episode identifier
    output_dir: Optional[str] = None,        # Required: output directory path
    speaker_config: Optional[str] = None,    # Speaker profile name
    outline_provider: Optional[str] = None,  # LLM provider for outline (default: "openai")
    outline_model: Optional[str] = None,     # LLM model for outline (default: "gpt-4o-mini")
    transcript_provider: Optional[str] = None,  # LLM provider for transcript (default: "anthropic")
    transcript_model: Optional[str] = None,  # LLM model for transcript (default: "claude-3-5-sonnet-latest")
    num_segments: Optional[int] = None,      # Number of outline segments (default: 3)
    episode_profile: Optional[str] = None,   # Episode profile name for defaults
    briefing_suffix: Optional[str] = None,   # Appended to profile's default_briefing
    outline_config: Optional[Dict] = None,   # Extra config for outline LLM
    transcript_config: Optional[Dict] = None, # Extra config for transcript LLM
    retry_max_attempts: Optional[int] = None,
    retry_wait_multiplier: Optional[int] = None,
    language: Optional[str] = None,          # Language code (e.g., "pt", "es")
) -> Dict
```

**Returns:**
```python
{
    "outline": Outline,
    "transcript": List[Dialogue],
    "final_output_file_path": Path,
    "audio_clips_count": int,
    "output_dir": Path,
}
```

**Output files created:**
- `{output_dir}/outline.json`
- `{output_dir}/transcript.json`
- `{output_dir}/clips/0000.mp3, 0001.mp3, ...`
- `{output_dir}/audio/{episode_name}.mp3`

## Data Models

### Segment
```python
class Segment(BaseModel):
    name: str
    description: str
    size: Literal["short", "medium", "long"]  # Determines turns: 3/6/10
```

### Outline
```python
class Outline(BaseModel):
    segments: list[Segment]
```

### Dialogue
```python
class Dialogue(BaseModel):
    speaker: str    # Speaker name (validated non-empty)
    dialogue: str   # The spoken text
```

### Transcript
```python
class Transcript(BaseModel):
    transcript: list[Dialogue]
```

## Speaker System

### Speaker
```python
class Speaker(BaseModel):
    name: str
    voice_id: str
    backstory: str
    personality: str
    tts_provider: Optional[str] = None    # Per-speaker override
    tts_model: Optional[str] = None       # Per-speaker override
    tts_config: Optional[Dict] = None     # Per-speaker override
```

### SpeakerProfile
```python
class SpeakerProfile(BaseModel):
    tts_provider: str                     # Default TTS provider
    tts_model: str                        # Default TTS model
    speakers: List[Speaker]               # 1-4 speakers (validated)
    tts_config: Optional[Dict] = None

    def get_speaker_names() -> List[str]
    def get_voice_mapping() -> Dict[str, str]   # {name: voice_id}
    def get_speaker_by_name(name: str) -> Speaker
```

### SpeakerConfig
```python
class SpeakerConfig(BaseModel):
    profiles: Dict[str, SpeakerProfile]

    def get_profile(profile_name: str) -> SpeakerProfile
    def list_profiles() -> List[str]
    @classmethod
    def load_from_file(config_path: Union[Path, str]) -> "SpeakerConfig"
```

### load_speaker_config() Priority Cascade
1. Configured via `configure("speakers_config", {...})`
2. Configured file path via `configure("speakers_config", "/path/to/file.json")`
3. `./speakers_config.json` in working directory
4. Bundled defaults (4 built-in profiles)

**Built-in profiles:** `ai_researchers` (2 speakers, ElevenLabs), `business_analysts` (3), `solo_expert` (1, OpenAI TTS), `diverse_panel` (4, ElevenLabs)

## Episode Profile System

### EpisodeProfile
```python
class EpisodeProfile(BaseModel):
    speaker_config: str
    outline_provider: str = "openai"
    outline_model: str = "gpt-4o-mini"
    transcript_provider: str = "anthropic"
    transcript_model: str = "claude-3-5-sonnet-latest"
    default_briefing: str = ""
    num_segments: int = 3               # Validated: 1-10
    language: Optional[str] = None
    outline_config: Optional[Dict] = None
    transcript_config: Optional[Dict] = None
```

**Built-in profiles:** `tech_discussion` (4 segments), `solo_expert` (3), `business_analysis` (4), `diverse_panel` (5)

## Configuration System

```python
from podcast_creator import configure

# Single key-value
configure("prompts_dir", "/path/to/prompts")
configure("speakers_config", "/path/to/speakers.json")
configure("episode_config", "/path/to/episodes.json")

# Multiple settings
configure({
    "prompts_dir": "/path/to/prompts",
    "speakers_config": "/path/to/speakers.json"
})

# Inline speaker configs (bypasses file system)
configure("speakers_config", {
    "profiles": {
        "my_speakers": {
            "tts_provider": "openai",
            "tts_model": "gpt-4o-mini-tts",
            "speakers": [...]
        }
    }
})

# Inline templates
configure("templates", {
    "outline": "Your custom Jinja2 template...",
    "transcript": "Your custom Jinja2 template..."
})
```

### Template Resolution Priority
1. Inline templates set via `configure("templates", {...})`
2. Configured prompts directory: `{prompts_dir}/podcast/{name}.jinja`
3. Working directory: `./prompts/podcast/{name}.jinja`
4. Bundled defaults from package resources

## Pipeline Stages (LangGraph Workflow)

```
START → generate_outline → generate_transcript → generate_all_audio → combine_audio → END
```

### Stage 1: generate_outline_node
- Creates LLM via `AIFactory.create_language(provider, model).to_langchain()`
- Renders outline Jinja2 template with: `briefing`, `num_segments`, `context`, `speakers`, `language`
- Parses response via `PydanticOutputParser(Outline)`
- Handles `<think>` tags from reasoning models

### Stage 2: generate_transcript_node
- Iterates over each segment **sequentially** (for context coherence)
- Renders transcript template with: `briefing`, `outline`, `context`, `segment`, `is_final`, `turns`, `speakers`, `transcript` (accumulated)
- Turns derived from segment size: `short`=3, `medium`=6, `long`=10
- Validates speaker names against profile

### Stage 3: generate_all_audio_node
- Processes dialogue in batches of `TTS_BATCH_SIZE` (default: 5)
- Within each batch, clips generated **concurrently** via `asyncio.gather()`
- 1-second inter-batch delay for rate limiting
- Per-speaker TTS overrides resolved
- Audio saved as `{output_dir}/clips/{index:04d}.mp3`

### Stage 4: combine_audio_node
- Reads all `.mp3` files from `{output_dir}/clips/` in sorted order
- Uses `moviepy.AudioFileClip` + `concatenate_audioclips()` to join
- Writes final output as `{output_dir}/audio/{episode_name}.mp3`

## Direct Node Access (Advanced Patterns)

### Two-Stage Transcript Generation (skip audio)
```python
from podcast_creator.nodes import generate_outline_node, generate_transcript_node
from podcast_creator.state import PodcastState

state = PodcastState(
    content=content, briefing=briefing, num_segments=3,
    outline=None, transcript=[], audio_clips=[],
    final_output_file_path=None,
    output_dir=output_dir, episode_name=name,
    speaker_profile=speaker_profile,
)

config = {"configurable": {
    "outline_provider": "openai", "outline_model": "gpt-4o-mini",
    "transcript_provider": "anthropic", "transcript_model": "claude-3-5-sonnet-latest",
}}

outline_result = await generate_outline_node(state, config)
state["outline"] = outline_result["outline"]

transcript_result = await generate_transcript_node(state, config)
state["transcript"] = transcript_result["transcript"]
```

### Audio Regeneration from Edited Transcript
```python
from podcast_creator import Dialogue
from podcast_creator.nodes import generate_all_audio_node, combine_audio_node

transcript = [Dialogue(**entry) for entry in json.load(open(path))]

state = PodcastState(
    content="", briefing="", num_segments=0,
    outline=None, transcript=transcript,
    audio_clips=[], final_output_file_path=None,
    output_dir=output_dir, episode_name=name,
    speaker_profile=speaker_profile,
)

audio_result = await generate_all_audio_node(state, config={})
state["audio_clips"] = audio_result["audio_clips"]

combine_result = await combine_audio_node(state, config={})
```

## Integration with Esperanto (AI Factory)

**LLM creation:**
```python
model = AIFactory.create_language(
    provider, model_name,
    config={"max_tokens": 3000, "structured": {"type": "json"}, ...}
).to_langchain()
```

**TTS creation:**
```python
tts = AIFactory.create_text_to_speech(
    provider, model_name,
    api_key=None, base_url=None, **tts_config
)
await tts.agenerate_speech(text=..., voice=..., output_file=...)
```

## Retry and Error Handling

- Uses `tenacity` with exponential backoff
- **Non-retryable:** `ValueError`, `TypeError`, `KeyError`, `FileNotFoundError`, HTTP 4xx (except 429)
- **Retryable:** HTTP 429, 5xx, network timeouts
- Config priority: `create_podcast()` params > env vars > defaults

## Key Gotchas

1. **Async-only API**: `create_podcast()` requires `await` or `asyncio.run()`
2. **Singleton ConfigurationManager**: `configure()` affects all subsequent calls globally
3. **Speaker count limit**: 1-4 speakers per profile
4. **Segment count limit**: 1-10 segments per episode
5. **Sequential transcript**: Segments processed sequentially for context coherence
6. **MoviePy dependency**: Uses `imageio-ffmpeg` (~80MB) for audio concatenation only
7. **No streaming**: Pipeline runs to completion
8. **Template `format_instructions`**: Custom templates must include `{{ format_instructions }}`
9. **Working directory sensitivity**: Without `configure()`, looks for configs in `cwd`
10. **Output always MP3**: No format configuration options
11. **`<think>` tag stripping**: Handles reasoning traces from DeepSeek etc.
12. **Module-level imports trigger graph compilation**: Importing the library produces log output

## Version History

| Version | Key Changes |
|---|---|
| 0.12.0 | Multilingual support via language codes |
| 0.11.0 | Automatic retry with exponential backoff |
| 0.10.0 | Per-speaker TTS provider overrides |
| 0.9.0 | Simplified proxy config to env vars |
| 0.7.0 | Configurable TTS batch size, CLI enhancements |
| 0.4.0 | Streamlit made optional |
| 0.3.0 | Initial public release |
