# Skill: MCP Servers for Podcast Generation

## Purpose

Evaluation of Model Context Protocol (MCP) servers relevant to a podcast generation tool, with installation instructions and integration recommendations.

---

## 1. ElevenLabs MCP (Official) — Priority: VERY HIGH

**What it does:** Text-to-speech generation, voice cloning, audio transcription, voice design, and audio isolation via ElevenLabs API.

**Repository:** https://github.com/elevenlabs/elevenlabs-mcp

**Installation:**
```bash
# Install uv if needed
curl -LsSf https://astral.sh/uv/install.sh | sh

# Or via pip
pip install elevenlabs-mcp
```

**Claude Desktop Config:**
```json
{
  "mcpServers": {
    "ElevenLabs": {
      "command": "uvx",
      "args": ["elevenlabs-mcp"],
      "env": {
        "ELEVENLABS_API_KEY": "<your-api-key>"
      }
    }
  }
}
```

**Options:**
- `ELEVENLABS_MCP_BASE_PATH` — output directory (default: ~/Desktop)
- Output modes: `files`, `resources` (base64), or `both`

**Why it matters:** Core TTS capability for generating podcast audio. High-quality voices with cloning support. Free tier: 10K credits/month.

---

## 2. SurrealMCP (Official) — Priority: HIGH

**What it does:** Secure database access for storing/querying podcast metadata, scripts, generation state. Supports stdio, HTTP, Unix socket transports with auth, rate limiting, and OpenTelemetry.

**Repository:** https://github.com/surrealdb/surrealmcp
**Website:** https://surrealdb.com/mcp

**Installation:**
```bash
# Docker
docker run --rm -i --pull always surrealdb/surrealmcp:latest start

# From source (Rust/Cargo)
cargo install --path .
```

**Environment Variables:**
```
SURREALDB_URL, SURREALDB_NS, SURREALDB_DB, SURREALDB_USER, SURREALDB_PASS
```

**Why it matters:** If using SurrealDB for podcast data, this enables the AI agent to directly query and manage episode metadata, scripts, and pipeline state.

**Alternatives:**
- [nsxdavid/surrealdb-mcp-server](https://github.com/nsxdavid/surrealdb-mcp-server) (community)
- [lfnovo/surreal-mcp](https://github.com/lfnovo/surreal-mcp) (same author as podcast-creator)

---

## 3. FFmpeg Video/Audio MCP — Priority: HIGH

**What it does:** 30+ FFmpeg-powered tools for professional media editing: audio format conversion, silence removal, concatenation, bitrate/sample rate adjustment, codec switching, audio extraction, watermarking, subtitle burning.

**Repository:** https://github.com/misbahsy/video-audio-mcp

**Requirements:** Python 3.8+, FFmpeg installed

**Claude Desktop Config:**
```json
{
  "mcpServers": {
    "VideoAudioServer": {
      "command": "uv",
      "args": ["--directory", "/path/to/video-audio-mcp", "run", "server.py"]
    }
  }
}
```

**Why it matters:** Post-production audio processing — concatenating TTS segments, adding intro/outro music, silence removal, normalization, and format export.

---

## 4. Multi-Engine TTS MCP (mcp-tts) — Priority: HIGH

**What it does:** TTS with four backends: macOS `say` (free/offline), ElevenLabs, Google Gemini (30 voices), OpenAI (10 voices with speed control + instructions).

**Repository:** https://github.com/blacktop/mcp-tts

**Installation:**
```bash
go install github.com/blacktop/mcp-tts@latest
```

**Claude Code CLI:**
```bash
claude mcp add say -- mcp-tts
```

**Environment Variables:**
```
OPENAI_API_KEY, ELEVENLABS_API_KEY, GOOGLE_AI_API_KEY, MCP_TTS_OUTPUT_DIR
```

**Why it matters:** Compare voices across providers before settling on one. Mix providers for different characters. Sequential playback prevents concurrent speech.

---

## 5. FFmpeg MCP Lite — Priority: MODERATE

**What it does:** Lightweight FFmpeg server for audio extraction, format conversion, compression. Built on FastMCP.

**Repository:** https://github.com/kevinwatt/ffmpeg-mcp-lite

**Installation:**
```bash
uvx ffmpeg-mcp-lite
```

**Why it matters:** Simpler alternative to the full video-audio-mcp if only basic audio operations are needed.

---

## 6. Podcast Transcriber MCP — Priority: MODERATE

**What it does:** RSS feed parsing, episode listing with pagination, audio transcription via OpenAI Whisper with intelligent chunking (full episodes, not just 60s).

**Repository:** https://github.com/dingkwang/podcast-transcriber-mcp

**Installation:**
```bash
git clone https://github.com/dingkwang/podcast-transcriber-mcp
cd podcast-transcriber-mcp && npm install && npm run build
```

**Why it matters:** Analyze existing podcasts for style/pacing reference. Reverse direction from generation but useful for content research.

---

## 7. Filesystem MCP (Official) — Priority: MODERATE

**What it does:** Secure file operations (read, write, list, move, search) with directory access controls.

**Repository:** https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem

**Installation:**
```bash
npx @modelcontextprotocol/server-filesystem /path/to/allowed/dir
```

**Why it matters:** Utility server for managing generated podcast files on disk.

---

## 8. RSS Feed MCP Servers — Priority: LOW-MODERATE

- **[feed-mcp](https://mcpservers.org/servers/richardwooding/feed-mcp)** — Read RSS/Atom/JSON feeds
- **[mcp_rss](https://github.com/buhe/mcp_rss)** — RSS feed interaction
- **[rss-mcp](https://github.com/veithly/rss-mcp)** — TypeScript RSS/Atom parser with RSSHub

**Why it matters:** Content ingestion from feeds for news-style podcasts, or RSS feed generation for publishing.

---

## 9. Cloud Storage MCP Servers — Priority: LOW-MODERATE

- **[gcloud-mcp](https://github.com/googleapis/gcloud-mcp)** — Google Cloud Storage
- **[aws-ow-s3-mcp](https://mcpservers.org/servers/OpenWorkspace-o1/aws-ow-s3-mcp)** — AWS S3
- **[gdrive-mcp-server](https://mcpservers.org/servers/felores/gdrive-mcp-server)** — Google Drive

**Why it matters:** Store generated podcasts in cloud storage for distribution/hosting.

---

## Recommendation Summary

| MCP Server | Relevance | Integrate? | Priority |
|---|---|---|---|
| **ElevenLabs MCP** | Very High | **Yes** — core TTS | 1 |
| **SurrealMCP** | High | **Yes** — if using SurrealDB | 2 |
| **video-audio-mcp (FFmpeg)** | High | **Yes** — audio post-processing | 3 |
| **mcp-tts (multi-engine)** | High | **Yes** — multi-provider TTS | 4 |
| **Filesystem MCP** | Moderate | Maybe | 5 |
| **ffmpeg-mcp-lite** | Moderate | Maybe — lighter alternative | 6 |
| **podcast-transcriber-mcp** | Moderate | Maybe — reference analysis | 7 |
| **RSS Feed servers** | Low-Moderate | Maybe — content ingestion | 8 |
| **Cloud Storage servers** | Low-Moderate | Maybe — hosting/distribution | 9 |

## Integration Decision

MCP servers are configured per-agent, not per-project. During the planning phase, decide which MCP servers should be recommended in the project documentation vs. which should be configured in the development environment. The top 4 (ElevenLabs, SurrealMCP, FFmpeg, mcp-tts) should be evaluated as first-class integrations.
