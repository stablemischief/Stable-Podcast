# Skill: Extract Component from Open Notebook

## Purpose

Guide the extraction of podcast-related code from the Open Notebook (Stable-Notebook) source repository into Stable-Podcast. This skill provides the file mapping, dependency chain, and adaptation patterns needed for clean extraction.

## Source Repository

- **Repo**: stablemischief/Stable-Notebook (fork of lfnovo/open-notebook)
- **Local Path**: /Users/jw-dev/Developer/GitHub/adw-orchestrator/apps/open-notebook
- **Branch**: main (HEAD at 4c64b5e)

## Extraction Categories

### Category A: Podcast-Only Files (Direct Copy + Adapt)

These files are 100% podcast-specific. Copy them and adapt imports/paths for the new project structure.

**Backend Models:**
```
open_notebook/podcasts/           → stable_podcast/models/
  models.py                       → stable_podcast/models/podcast.py
    - EpisodeProfile(ObjectModel)   → Podcast format template
    - SpeakerProfile(ObjectModel)   → Voice configuration
    - PodcastEpisode(ObjectModel)   → Generated episode with FSM
```

**Backend Commands (Pipeline):**
```
commands/podcast_commands.py              → stable_podcast/commands/generate.py
commands/generate_transcript_command.py   → stable_podcast/commands/transcript.py
commands/regenerate_audio_command.py      → stable_podcast/commands/audio.py
```

**Backend API Routes:**
```
api/routers/podcasts.py            → stable_podcast/api/routers/podcasts.py
api/routers/episode_profiles.py    → stable_podcast/api/routers/episode_profiles.py
api/routers/speaker_profiles.py    → stable_podcast/api/routers/speaker_profiles.py
api/routers/basic_mode.py          → stable_podcast/api/routers/basic_mode.py
```

**Backend Services:**
```
api/podcast_service.py             → stable_podcast/api/services/podcast_service.py
api/command_service.py             → stable_podcast/api/services/command_service.py
```

**Database Migrations:**
```
open_notebook/database/migrations/7.surrealql   → migrations/001_podcast_tables.surrealql
open_notebook/database/migrations/14.surrealql  → migrations/002_audio_version.surrealql
open_notebook/database/migrations/15.surrealql  → migrations/003_pipeline_state.surrealql
```

**Frontend Pages:**
```
frontend/src/app/(dashboard)/podcasts/page.tsx  → frontend/src/app/podcasts/page.tsx
frontend/src/app/(dashboard)/simple/page.tsx    → frontend/src/app/simple/page.tsx
```

**Frontend Components:**
```
frontend/src/components/podcasts/     → frontend/src/components/podcasts/
  EpisodesTab.tsx
  TemplatesTab.tsx
  EpisodeCard.tsx
  TranscriptEditorDialog.tsx
  GeneratePodcastDialog.tsx
  EpisodeProfilesPanel.tsx
  SpeakerProfilesPanel.tsx
  forms/EpisodeProfileFormDialog.tsx
  forms/SpeakerProfileFormDialog.tsx

frontend/src/components/simple-mode/  → frontend/src/components/simple-mode/
  SimpleModeView.tsx
  SimpleEpisodeCard.tsx
  SimpleModeSettings.tsx
  PdfUploadZone.tsx
  ReplaceEpisodeWarning.tsx
```

**Frontend Hooks, API Clients, Types:**
```
frontend/src/lib/hooks/use-podcasts.ts   → frontend/src/lib/hooks/use-podcasts.ts
frontend/src/lib/hooks/use-basic-mode.ts → frontend/src/lib/hooks/use-basic-mode.ts
frontend/src/lib/api/podcasts.ts         → frontend/src/lib/api/podcasts.ts
frontend/src/lib/api/basic-mode.ts       → frontend/src/lib/api/basic-mode.ts
frontend/src/lib/types/podcasts.ts       → frontend/src/lib/types/podcasts.ts
frontend/src/lib/stores/mode-store.ts    → frontend/src/lib/stores/mode-store.ts
```

**Tests:**
```
tests/test_podcasts_api.py            → tests/test_podcasts_api.py
tests/test_regeneration_status.py     → tests/test_regeneration_status.py
tests/test_frontend_status_logic.py   → tests/test_frontend_status_logic.py
```

### Category B: Shared Infrastructure (Must Reimplement or Extract)

These components are used by podcasts AND other Open Notebook features. They need to be extracted and potentially simplified.

**ORM Layer (Critical — everything depends on this):**
```
open_notebook/domain/base.py
  - ObjectModel: Base class with save(), delete(), get(), get_all()
  - RecordModel: Singleton settings pattern
  → Decision: Extract minimal version or replace with SQLAlchemy/Prisma

open_notebook/database/repository.py
  - repo_query(), repo_create(), repo_update(), repo_delete(), ensure_record_id()
  → Decision: Extract if keeping SurrealDB, replace if switching DB

open_notebook/database/db.py
  - AsyncSurreal connection singleton
  → Decision: Extract if keeping SurrealDB
```

**AI System:**
```
open_notebook/ai/models.py
  - Model, DefaultModels — model registry and selection
  → Extract subset needed for LLM + TTS only

open_notebook/ai/key_provider.py
  - API key provisioning from credentials
  → Simplify for podcast-only use case

open_notebook/domain/credential.py
  - Credential model with encryption
  → Extract or simplify (env vars may be sufficient)
```

**Configuration:**
```
open_notebook/config.py
  - DATA_FOLDER and other settings
  → Create simplified config for Stable-Podcast

open_notebook/exceptions.py
  - Custom exception types
  → Create podcast-specific exceptions
```

**Content Pipeline (Simple Mode Only):**
```
content_core library
  - PDF text extraction
  → Keep as dependency if Simple Mode is included

open_notebook/domain/notebook.py
  - Notebook, Source models (used for content retrieval)
  → Simplify: may only need a generic "content source" abstraction
```

**BasicModeSettings:**
```
open_notebook/domain/basic_mode_settings.py
  - BasicModeSettings(RecordModel)
  → Extract directly (podcast-specific, just uses shared base class)
```

### Category C: Frontend Shared Components (Bring Along)

These are shared UI utilities that podcast components import. Include them as-is:

```
frontend/src/lib/api/client.ts          → API client (axios/fetch wrapper)
frontend/src/lib/utils.ts               → cn() class helper
frontend/src/lib/utils/error-handler.ts  → Error message mapping
frontend/src/lib/hooks/use-toast.ts      → Toast notifications
frontend/src/lib/hooks/use-translation.ts → i18n system
frontend/src/lib/api/query-client.ts     → TanStack Query client + QUERY_KEYS
frontend/src/components/ui/*             → Shadcn/ui components (install fresh)
```

## Extraction Process

### Step 1: Read the Source File
```bash
# Read the file from Open Notebook
cat /Users/jw-dev/Developer/GitHub/adw-orchestrator/apps/open-notebook/{source_path}
```

### Step 2: Identify Import Dependencies
Look at every `import` and `from ... import` statement. Classify each as:
- **Podcast-only** → will exist in new project
- **Shared infrastructure** → needs extraction decision
- **External library** → add to dependencies
- **Standard library** → no action needed

### Step 3: Adapt the Code
1. Update import paths from `open_notebook.` to `stable_podcast.`
2. Update `api.` imports to `stable_podcast.api.`
3. Replace any notebook/source/note references with podcast-appropriate abstractions
4. Remove any non-podcast functionality
5. Preserve all error handling, logging, and security patterns

### Step 4: Validate
- All imports resolve
- Type hints are correct
- No references to removed functionality
- Tests pass

## Common Import Transformations

```python
# Old → New
from open_notebook.podcasts.models import ...    → from stable_podcast.models.podcast import ...
from open_notebook.domain.base import ...        → from stable_podcast.models.base import ...
from open_notebook.database.repository import ... → from stable_podcast.database.repository import ...
from open_notebook.config import DATA_FOLDER     → from stable_podcast.config import DATA_FOLDER
from open_notebook.ai.models import ...          → from stable_podcast.ai.models import ...
```

## Key Gotchas

1. **ObjectModel.save()** calls `repo_create()` or `repo_update()` internally — it's not a simple data class
2. **PodcastEpisode** stores full snapshots of EpisodeProfile and SpeakerProfile at generation time (denormalized)
3. **Pipeline state FSM** has specific transitions — don't break the state machine
4. **Audio file paths** use DATA_FOLDER — ensure this is configured in the new project
5. **surreal-commands** is tightly coupled to SurrealDB — if switching DB, the job queue needs full reimplementation
6. **Frontend GeneratePodcastDialog** imports `use-notebooks` hook for content selection — this will need a replacement content input mechanism
7. **Frontend uses i18n** — translation keys must be carried over or the UI will show raw keys
