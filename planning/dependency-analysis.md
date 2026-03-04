# Podcast Extraction Dependency Map

**Source**: [stablemischief/Stable-Notebook](https://github.com/stablemischief/Stable-Notebook) (fork of [lfnovo/open-notebook](https://github.com/lfnovo/open-notebook))
**Generated**: 2026-03-04
**Purpose**: Exhaustive mapping of every component needed to extract podcast generation into a standalone application.

---

## Architecture Overview

The podcast system is a 5-stage async pipeline: **Outline (LLM) → Transcript (LLM) → User Review → TTS Audio → Audio Mixing (FFmpeg)**. It uses `podcast-creator` as the core engine, `surreal-commands` for async job orchestration, and Esperanto for multi-provider AI.

---

## 1. BACKEND MODELS (`open_notebook/podcasts/`)

| File | Class/Entity | Purpose |
|------|-------------|---------|
| `open_notebook/podcasts/models.py` | `EpisodeProfile(ObjectModel)` | Podcast format template (outline/transcript AI provider/model, briefing, segments) |
| `open_notebook/podcasts/models.py` | `SpeakerProfile(ObjectModel)` | Voice config (TTS provider/model, 1-4 speakers with voice_id/backstory/personality) |
| `open_notebook/podcasts/models.py` | `PodcastEpisode(ObjectModel)` | Generated episode with pipeline_state FSM, profile snapshots, transcript, outline, audio_file |
| `open_notebook/domain/base.py` | `ObjectModel` | **SHARED** - Base ORM class with save/delete/get/get_all (podcast models inherit this) |
| `open_notebook/domain/base.py` | `RecordModel` | **SHARED** - Singleton settings pattern (used by BasicModeSettings) |
| `open_notebook/domain/basic_mode_settings.py` | `BasicModeSettings(RecordModel)` | Simple Mode settings (notebook_id, default profile, suffixes, current_episode_id) |

**Transitive dependency**: `ObjectModel` depends on `open_notebook/database/repository.py` (repo_query, repo_create, repo_update, repo_delete, ensure_record_id)

---

## 2. BACKEND WORKFLOWS & COMMANDS (`commands/`)

| File | Command | Stages | Key Imports from `podcast-creator` |
|------|---------|--------|-----------------------------------|
| `commands/podcast_commands.py` | `generate_podcast` | 1-5 (full) | `configure`, `create_podcast` |
| `commands/generate_transcript_command.py` | `generate_transcript` | 1-3 (transcript only) | `configure`, `generate_outline_node`, `generate_transcript_node`, `load_speaker_config`, `PodcastState` |
| `commands/regenerate_audio_command.py` | `regenerate_audio` | 4-5 (audio only) | `configure`, `Dialogue`, `generate_all_audio_node`, `combine_audio_node`, `load_speaker_config`, `PodcastState` |

**Pipeline States FSM**: `generating_transcript` → `transcript_ready` → `generating_audio` → `completed` (with `transcript_failed` and `audio_failed` error branches)

**Each command depends on**:
- `surreal_commands` (CommandInput, CommandOutput, @command decorator, submit_command, get_command_status)
- `open_notebook.podcasts.models` (EpisodeProfile, SpeakerProfile, PodcastEpisode)
- `open_notebook.database.repository` (repo_query, ensure_record_id)
- `open_notebook.config` (DATA_FOLDER)
- `podcast_creator` (various nodes — see table above)

---

## 3. BACKEND API ROUTES & SERVICES

### Routers

| File | Prefix | Endpoints | Purpose |
|------|--------|-----------|---------|
| `api/routers/podcasts.py` | `/api/podcasts` | 11 endpoints | Episode CRUD, generation, transcript update, audio streaming, retry |
| `api/routers/episode_profiles.py` | `/api/episode-profiles` | 6 endpoints | Episode profile CRUD + duplicate |
| `api/routers/speaker_profiles.py` | `/api/speaker-profiles` | 6 endpoints | Speaker profile CRUD + duplicate |
| `api/routers/basic_mode.py` | `/api/basic-mode` + `/api/settings/basic-mode` | 3 endpoints | Simple Mode: PDF upload → transcript generation |
| `api/routers/commands.py` | `/api/commands` | 5 endpoints | Generic job status tracking (used by podcasts) |

### Services

| File | Class | Purpose |
|------|-------|---------|
| `api/podcast_service.py` | `PodcastService` | Submit generation jobs, list/get episodes, job status |
| `api/command_service.py` | `CommandService` | Generic surreal-commands wrapper for job submission |
| `api/podcast_api_service.py` | `PodcastAPIService` | HTTP client wrapper (legacy Streamlit) |
| `api/episode_profiles_service.py` | `EpisodeProfilesService` | HTTP client wrapper (legacy Streamlit) |

### Key Endpoint Details

| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/api/podcasts/generate` | Submit generation job |
| `GET` | `/api/podcasts/episodes` | List episodes with crash recovery reconciliation |
| `PUT` | `/api/podcasts/episodes/{id}/transcript` | Update transcript with speaker validation + backup |
| `GET` | `/api/podcasts/episodes/{id}/audio` | Stream audio (path-traversal protected) |
| `POST` | `/api/podcasts/episodes/{id}/regenerate-audio` | Regenerate audio from edited transcript |
| `POST` | `/api/podcasts/episodes/{id}/approve-audio` | Approve transcript → trigger audio |
| `POST` | `/api/podcasts/episodes/{id}/retry-transcript` | Retry failed transcript |
| `POST` | `/api/basic-mode/generate` | PDF upload → text extraction → transcript generation |

---

## 4. DATABASE SCHEMA (SurrealDB)

### Tables (Migration `7.surrealql`)

| Table | Fields | Indexes |
|-------|--------|---------|
| `episode_profile` | name, description, speaker_config, outline_provider, outline_model, transcript_provider, transcript_model, default_briefing, num_segments | `idx_episode_profile_name` (UNIQUE) |
| `speaker_profile` | name, description, tts_provider, tts_model, speakers[].{name,voice_id,backstory,personality} | `idx_speaker_profile_name` (UNIQUE) |
| `episode` | name, episode_profile (object), speaker_profile (object), briefing, content, transcript, outline, audio_file, command (record<command>), audio_version, pipeline_state | `idx_episode_profile`, `idx_episode_command` |

### Additional Migrations
- `14.surrealql`: Adds `audio_version` (int, default 1) to `episode`
- `15.surrealql`: Adds `pipeline_state` (string, default "completed") to `episode`
- Default seed data: 3 episode profiles + 3 speaker profiles

---

## 5. FRONTEND PAGES

| File | Route | Purpose |
|------|-------|---------|
| `frontend/src/app/(dashboard)/podcasts/page.tsx` | `/podcasts` | Main podcasts page with Episodes/Templates tabs |
| `frontend/src/app/(dashboard)/simple/page.tsx` | `/simple` | Simple Mode daily podcast studio |

---

## 6. FRONTEND COMPONENTS

### Podcast Components (`frontend/src/components/podcasts/`)

| File | Component | Purpose |
|------|-----------|---------|
| `EpisodesTab.tsx` | `EpisodesTab` | Episode list grouped by status with auto-refresh |
| `TemplatesTab.tsx` | `TemplatesTab` | Episode + speaker profile management panels |
| `EpisodeCard.tsx` | `EpisodeCard` | Full episode card: audio player, transcript viewer, status badge, actions |
| `TranscriptEditorDialog.tsx` | `TranscriptEditorDialog` | Edit transcript entries with speaker autocomplete, optional audio regen |
| `GeneratePodcastDialog.tsx` | `GeneratePodcastDialog` | Multi-notebook content selection, profile picker, generation form |
| `EpisodeProfilesPanel.tsx` | `EpisodeProfilesPanel` | CRUD panel for episode profiles |
| `SpeakerProfilesPanel.tsx` | `SpeakerProfilesPanel` | CRUD panel for speaker profiles with usage tracking |
| `forms/EpisodeProfileFormDialog.tsx` | `EpisodeProfileFormDialog` | Create/edit episode profile form with Zod validation |
| `forms/SpeakerProfileFormDialog.tsx` | `SpeakerProfileFormDialog` | Create/edit speaker profile form with dynamic 1-4 speakers |

### Simple Mode Components (`frontend/src/components/simple-mode/`)

| File | Component | Purpose |
|------|-----------|---------|
| `SimpleModeView.tsx` | `SimpleModeView` | Main container: PDF upload → transcript → audio (dark theme) |
| `SimpleEpisodeCard.tsx` | `SimpleEpisodeCard` | Compact episode card with progress steps and action buttons |
| `SimpleModeSettings.tsx` | `SimpleModeSettings` | Settings modal: notebook, profile defaults, suffixes |
| `PdfUploadZone.tsx` | `PdfUploadZone` | Drag-and-drop PDF upload zone |
| `ReplaceEpisodeWarning.tsx` | `ReplaceEpisodeWarning` | Confirmation dialog for replacing active episode |

---

## 7. FRONTEND HOOKS (`frontend/src/lib/hooks/`)

### `use-podcasts.ts` (18 hooks)
- `usePodcastEpisodes()` — List episodes with 15s auto-refresh
- `useDeletePodcastEpisode()` / `useRetryPodcastEpisode()`
- `useEpisodeProfiles()` / `useCreateEpisodeProfile()` / `useUpdateEpisodeProfile()` / `useDeleteEpisodeProfile()` / `useDuplicateEpisodeProfile()`
- `useSpeakerProfiles()` / `useCreateSpeakerProfile()` / `useUpdateSpeakerProfile()` / `useDeleteSpeakerProfile()` / `useDuplicateSpeakerProfile()`
- `useGeneratePodcast()` / `useUpdateEpisodeTranscript()` / `useRegenerateAudio()` / `useApproveTranscript()` / `useRetryTranscript()`

### `use-basic-mode.ts` (4 hooks)
- `useBasicModeSettings()` / `useUpdateBasicModeSettings()`
- `useBasicModeEpisode()` — 5s polling for active episodes
- `useBasicModeGenerate()`

---

## 8. FRONTEND API CLIENTS & TYPES

| File | Exports | Purpose |
|------|---------|---------|
| `frontend/src/lib/api/podcasts.ts` | `podcastsApi`, `resolvePodcastAssetUrl()` | 18 API methods for all podcast endpoints |
| `frontend/src/lib/api/basic-mode.ts` | `basicModeApi`, `BasicModeSettings`, `BasicModeGenerateResponse` | 3 API methods for Simple Mode |
| `frontend/src/lib/types/podcasts.ts` | All podcast TypeScript types | `PodcastEpisode`, `EpisodeProfile`, `SpeakerProfile`, `PipelineState`, status helpers |
| `frontend/src/lib/api/query-client.ts` | `QUERY_KEYS` | 6 podcast-related query keys |

### Zustand Store

| File | Store | Purpose |
|------|-------|---------|
| `frontend/src/lib/stores/mode-store.ts` | `useModeStore` | App mode ('simple'/'full'), persisted to localStorage |

---

## 9. INFRASTRUCTURE & EXTERNAL DEPENDENCIES

### Python Packages (podcast-specific)

| Package | Version | Purpose |
|---------|---------|---------|
| `podcast-creator` | >=0.11.2,<1 | Core 5-stage podcast generation engine |
| `surreal-commands` | >=1.3.1,<2 | Async job queue for podcast commands |
| `esperanto` | >=2.19.3,<3 | Multi-provider AI (LLM + TTS) |
| `aiofiles` | >=25.1.0 | Async file I/O for transcript writes |

### System Dependencies (Docker)
- **FFmpeg** — Required for audio mixing (Stage 5)

### TTS Providers (via Esperanto/podcast-creator)
- OpenAI TTS (tts-1, tts-1-hd, gpt-4o-mini-tts)
- ElevenLabs
- Speaches (local OpenAI-compatible)
- Google Cloud TTS

### File Storage

| Path | Purpose |
|------|---------|
| `./data/podcasts/episodes/{name}/` | Per-episode workspace |
| `./data/podcasts/episodes/{name}/clips/` | TTS audio clips |
| `./data/podcasts/episodes/{name}/transcript.json` | Editable transcript |
| `./data/podcasts/episodes/{name}/final_output.mp3` | Final mastered audio |

---

## 10. TRANSITIVE DEPENDENCIES (Shared with Non-Podcast Features)

These components are **used by podcasts but also used elsewhere**. They would need to be extracted or reimplemented:

### Must Extract (Core Framework)

| Component | Used By Podcasts For | Also Used By |
|-----------|---------------------|--------------|
| `open_notebook/domain/base.py` (ObjectModel, RecordModel) | Base class for all podcast models | ALL domain models (Notebook, Source, Note, ChatSession, Credential) |
| `open_notebook/database/repository.py` | All DB operations (repo_query, repo_create, etc.) | Everything in the app |
| `open_notebook/database/db.py` (AsyncSurreal connection) | Database connectivity | Everything |
| `open_notebook/config.py` (DATA_FOLDER) | File path configuration | All file operations |
| `open_notebook/exceptions.py` | Error types | All modules |

### Must Extract (AI System)

| Component | Used By Podcasts For | Also Used By |
|-----------|---------------------|--------------|
| `open_notebook/ai/models.py` (Model, DefaultModels) | TTS model selection | Chat, embeddings, transformations |
| `open_notebook/ai/key_provider.py` | API key provisioning | All AI operations |
| `open_notebook/domain/credential.py` (Credential) | Encrypted API key storage | All AI providers |
| Esperanto library | TTS + LLM model provisioning | Chat, ask, source processing |

### Must Extract (Content Pipeline — Simple Mode Only)

| Component | Used By Podcasts For | Also Used By |
|-----------|---------------------|--------------|
| `content_core` library | PDF text extraction (basic_mode.py) | Source ingestion |
| `open_notebook/domain/notebook.py` (Notebook, Source) | Content retrieval (get_context) | Notebooks, notes, chat, search |

### Frontend Shared Dependencies

| Component | Used By Podcasts | Also Used By |
|-----------|-----------------|--------------|
| `@/lib/api/client.ts` (apiClient) | All API calls | Everything |
| `@/lib/hooks/use-toast.ts` | Toast notifications | Everything |
| `@/lib/hooks/use-translation.ts` | i18n | Everything |
| `@/lib/hooks/use-notebooks.ts` | Notebook selection in GeneratePodcastDialog | Notebooks UI |
| `@/lib/hooks/use-models.ts` | Model selection in profile forms | Settings, model management |
| `@/lib/api/chat.ts` (chatApi.buildContext) | Context token counting | Chat feature |
| `@/lib/utils.ts` (cn) | Class composition | Everything |
| `@/lib/utils/error-handler.ts` | Error message mapping | Everything |
| All `@/components/ui/*` (shadcn) | UI primitives | Everything |
| `react-hook-form` + `zod` | Form validation | Other forms |
| `@tanstack/react-query` | Server state | Everything |
| `zustand` | Client state | Everything |
| `date-fns` | Date formatting | Everything |
| `lucide-react` | Icons | Everything |

---

## 11. PODCAST-ONLY FILES (Clean Extraction)

These files are **100% podcast-specific** and can be extracted directly:

### Backend
- `open_notebook/podcasts/` (entire directory)
- `commands/podcast_commands.py`
- `commands/generate_transcript_command.py`
- `commands/regenerate_audio_command.py`
- `api/routers/podcasts.py`
- `api/routers/episode_profiles.py`
- `api/routers/speaker_profiles.py`
- `api/routers/basic_mode.py`
- `api/podcast_service.py`
- `api/podcast_api_service.py`
- `api/episode_profiles_service.py`
- `open_notebook/database/migrations/7.surrealql` (podcast tables)
- `open_notebook/database/migrations/14.surrealql` (audio_version)
- `open_notebook/database/migrations/15.surrealql` (pipeline_state)

### Frontend
- `frontend/src/app/(dashboard)/podcasts/page.tsx`
- `frontend/src/app/(dashboard)/simple/page.tsx`
- `frontend/src/components/podcasts/` (entire directory)
- `frontend/src/components/simple-mode/` (entire directory)
- `frontend/src/lib/api/podcasts.ts`
- `frontend/src/lib/api/basic-mode.ts`
- `frontend/src/lib/hooks/use-podcasts.ts`
- `frontend/src/lib/hooks/use-basic-mode.ts`
- `frontend/src/lib/types/podcasts.ts`
- `frontend/src/lib/stores/mode-store.ts`

### Tests
- `tests/test_podcasts_api.py`
- `tests/test_regeneration_status.py`
- `tests/test_frontend_status_logic.py`

---

## 12. EXTRACTION SUMMARY

| Category | Podcast-Only Files | Shared (Need Reimpl.) | Total |
|----------|-------------------|----------------------|-------|
| Backend Models | 1 module (3 classes) | 2 (ObjectModel, RecordModel) | 5 classes |
| Backend Commands | 3 files | 0 | 3 |
| Backend API | 7 files (5 routers + 2 services) | 1 (command_service) | 8 |
| Database Migrations | 3 files | 0 | 3 |
| Frontend Pages | 2 | 0 | 2 |
| Frontend Components | ~14 files | 0 | ~14 |
| Frontend Hooks | 2 files (22 hooks) | 0 | 22 hooks |
| Frontend API Clients | 2 files | 1 (shared apiClient) | 3 |
| Frontend Types | 1 file | 0 | 1 |
| Frontend Stores | 1 file | 0 | 1 |
| External Packages | 2 (podcast-creator, surreal-commands) | 3 (esperanto, surrealdb, content_core) | 5 |
| System Deps | 1 (FFmpeg) | 0 | 1 |

---

## 13. KEY EXTRACTION DECISIONS (TBD — Planning Phase)

The following decisions should be made during the architecture planning phase:

### Tech Stack Questions

1. **Database**: Keep SurrealDB or switch to something lighter (SQLite, Postgres)?
   - SurrealDB provides graph relationships and built-in vector search, but the podcast pipeline may not need those capabilities
   - The `surreal-commands` job queue is tightly coupled to SurrealDB
   - If switching DB, the `ObjectModel` ORM layer needs full reimplementation

2. **Job Queue**: Keep `surreal-commands` or use something standard (Celery, Redis Queue, etc.)?
   - `surreal-commands` is SurrealDB-specific
   - Podcast generation is the primary async workload

3. **AI Provider Layer**: Keep Esperanto or simplify?
   - Esperanto provides multi-provider abstraction for LLM + TTS
   - For podcast-only, we might only need LLM (script) + TTS (audio) — could be simpler
   - `podcast-creator` library already uses Esperanto internally

4. **Frontend Framework**: Keep Next.js or simplify?
   - The podcast UI is relatively simple — could be a lighter framework
   - But Next.js provides SSR, routing, and the existing component library

5. **ORM Layer**: Reimplement `ObjectModel` or use standard ORM?
   - `ObjectModel` is a custom thin ORM over SurrealDB
   - If keeping SurrealDB, extract it; if switching DB, use SQLAlchemy/Prisma/etc.

### Architecture Questions

1. **Monolith vs Split**: Single service or separate API + frontend?
2. **Content Ingestion**: How do users provide source content?
   - Open Notebook has full notebook/source/note pipeline
   - Stable Podcast could accept: plain text, PDF upload, URL, or API input
3. **Authentication**: Simple password, OAuth, or API key?
4. **Deployment**: Docker Compose, single container, or cloud-native?

---

## 14. RECOMMENDED NEXT STEPS

1. **Decide tech stack** — Use this document to evaluate what the podcast pipeline actually requires
2. **Create architecture design** — Document the target architecture for Stable Podcast
3. **Build implementation plan** — Break the extraction into phases with clear milestones
4. **Phase 1: Foundation** — Set up project skeleton, database, ORM, config
5. **Phase 2: Core Pipeline** — Extract podcast-creator integration, command system, API routes
6. **Phase 3: Frontend** — Build the podcast UI (episode management, transcript editor, audio player)
7. **Phase 4: Simple Mode** — Add the focused daily studio experience
8. **Phase 5: Polish** — Testing, documentation, Docker deployment
