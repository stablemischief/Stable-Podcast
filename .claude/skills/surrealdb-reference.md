# Skill: SurrealDB Python Driver Reference

## Purpose

Reference for SurrealDB's Python async driver, covering connection management, CRUD operations, and query patterns used in podcast data storage (episodes, profiles, pipeline state).

## Installation

```bash
pip install surrealdb
```

## Connection

```python
from surrealdb import AsyncSurreal

# Context manager (recommended)
async with AsyncSurreal("ws://localhost:8000/rpc") as db:
    await db.signin({"user": "root", "pass": "root"})
    await db.use("namespace", "database")
    # ... operations

# Manual lifecycle
db = AsyncSurreal("ws://localhost:8000/rpc")
await db.connect()
await db.signin({"user": "root", "pass": "root"})
await db.use("namespace", "database")
# ... operations
await db.close()
```

### Connection URL Formats

```
ws://localhost:8000/rpc          # WebSocket (development)
wss://cloud.surrealdb.com/rpc   # Secure WebSocket (production)
http://localhost:8000            # HTTP
https://cloud.surrealdb.com     # HTTPS
```

## CRUD Operations

### Create

```python
# Create with auto-generated ID
result = await db.create("podcast_episode", {
    "name": "episode_001",
    "pipeline_state": "generating_transcript",
    "created_at": "2026-03-04T12:00:00Z",
})

# Create with specific ID
result = await db.create("podcast_episode:ep001", {
    "name": "episode_001",
    "pipeline_state": "generating_transcript",
})
```

### Select (Read)

```python
# Get all records from table
episodes = await db.select("podcast_episode")

# Get specific record by ID
episode = await db.select("podcast_episode:ep001")
```

### Update (Full Replace)

```python
# Replace entire record
result = await db.update("podcast_episode:ep001", {
    "name": "episode_001",
    "pipeline_state": "completed",
    "audio_file": "file:///path/to/audio.mp3",
})
```

### Merge (Partial Update)

```python
# Update specific fields only
result = await db.merge("podcast_episode:ep001", {
    "pipeline_state": "completed",
    "audio_version": 2,
})
```

### Delete

```python
# Delete specific record
await db.delete("podcast_episode:ep001")

# Delete all records in table
await db.delete("podcast_episode")
```

## Query (SurrealQL)

```python
# Simple query
result = await db.query("SELECT * FROM podcast_episode WHERE pipeline_state = 'completed'")

# Parameterized query (prevents injection)
result = await db.query(
    "SELECT * FROM podcast_episode WHERE name = $name",
    {"name": "episode_001"}
)

# Complex queries
result = await db.query("""
    SELECT *,
        ->generated_by->speaker_profile.* AS speakers
    FROM podcast_episode
    WHERE pipeline_state IN ['completed', 'transcript_ready']
    ORDER BY created_at DESC
    LIMIT 10
""")
```

### Query Result Format

```python
# result is a list of query results (one per statement)
# Each element has: status, result, time
records = result[0]["result"]  # List of matching records
```

## SurrealQL Quick Reference

### Data Types

```sql
-- Strings, numbers, booleans
SET $name = "episode";
SET $count = 42;
SET $active = true;

-- Datetime
SET $now = time::now();
SET $created = <datetime>"2026-03-04T12:00:00Z";

-- Arrays and objects
SET $tags = ["ai", "podcast"];
SET $config = { provider: "openai", model: "gpt-4" };

-- Record IDs (table:id)
SET $episode = podcast_episode:ep001;
SET $profile = speaker_profile:default;
```

### Table Operations

```sql
-- Define table with schema
DEFINE TABLE podcast_episode SCHEMAFULL;
DEFINE FIELD name ON podcast_episode TYPE string;
DEFINE FIELD pipeline_state ON podcast_episode TYPE string
    ASSERT $value IN ["generating_transcript", "transcript_ready", "generating_audio", "completed", "transcript_failed", "audio_failed"];
DEFINE FIELD audio_file ON podcast_episode TYPE option<string>;
DEFINE FIELD audio_version ON podcast_episode TYPE option<int> DEFAULT 1;
DEFINE FIELD transcript ON podcast_episode TYPE option<object>;
DEFINE FIELD outline ON podcast_episode TYPE option<object>;
DEFINE FIELD created_at ON podcast_episode TYPE datetime DEFAULT time::now();

-- Indexes
DEFINE INDEX idx_pipeline_state ON podcast_episode FIELDS pipeline_state;
DEFINE INDEX idx_name ON podcast_episode FIELDS name UNIQUE;
```

### Migrations

```sql
-- Migration files are plain .surrealql files
-- Execute via CLI or programmatically

-- Example: Add audio_version field
DEFINE FIELD audio_version ON podcast_episode TYPE option<int> DEFAULT 1;
UPDATE podcast_episode SET audio_version = 1 WHERE audio_version IS NONE;
```

## Open Notebook ORM Pattern (ObjectModel)

The Open Notebook project wraps SurrealDB with a thin ORM. When extracting to Stable-Podcast, understand these patterns:

### ObjectModel Base Class

```python
class ObjectModel:
    """
    Base class providing save(), delete(), get(), get_all().
    - save() calls repo_create() or repo_update() depending on whether id exists
    - All fields are serialized to/from SurrealDB records
    - Record IDs use table:id format (e.g., "podcast_episode:abc123")
    """

    id: Optional[str] = None
    table_name: str  # Set by subclass

    async def save(self):
        if self.id:
            await repo_update(self.table_name, self.id, self.to_dict())
        else:
            result = await repo_create(self.table_name, self.to_dict())
            self.id = result["id"]

    async def delete(self):
        await repo_delete(self.table_name, self.id)

    @classmethod
    async def get(cls, record_id: str):
        data = await repo_query(f"SELECT * FROM {record_id}")
        return cls.from_dict(data[0]) if data else None

    @classmethod
    async def get_all(cls):
        data = await repo_query(f"SELECT * FROM {cls.table_name}")
        return [cls.from_dict(d) for d in data]
```

### Repository Functions

```python
async def repo_query(query: str, params: dict = None) -> list:
    """Execute SurrealQL query and return results."""

async def repo_create(table: str, data: dict) -> dict:
    """Create a new record."""

async def repo_update(table: str, record_id: str, data: dict) -> dict:
    """Update an existing record."""

async def repo_delete(table: str, record_id: str):
    """Delete a record."""

def ensure_record_id(record_id: str) -> str:
    """Ensure ID has table:id format."""
```

## Docker Setup

```bash
# Development
docker run --rm -p 8000:8000 \
    surrealdb/surrealdb:latest start \
    --user root --pass root \
    file:/data/surreal.db

# Docker Compose
services:
  surrealdb:
    image: surrealdb/surrealdb:latest
    command: start --user root --pass root file:/data/surreal.db
    ports:
      - "8000:8000"
    volumes:
      - surreal_data:/data
```

## Key Considerations for Stable-Podcast

1. **Decision Point:** Keep SurrealDB or switch to simpler DB (SQLite, PostgreSQL)?
   - SurrealDB is powerful but adds operational complexity
   - For podcast-only use case, graph capabilities may be overkill
   - If keeping SurrealDB, the ORM layer can be extracted as-is

2. **Pipeline State FSM:** The `pipeline_state` field tracks podcast generation progress:
   ```
   generating_transcript → transcript_ready → generating_audio → completed
                        → transcript_failed              → audio_failed
   ```

3. **Record ID format:** SurrealDB uses `table:id` format (e.g., `podcast_episode:abc123`). The `ensure_record_id()` helper normalizes this.

4. **Denormalized snapshots:** PodcastEpisode stores full copies of EpisodeProfile and SpeakerProfile at generation time. This means episodes are self-contained but profile changes don't retroactively update old episodes.

5. **surreal-commands dependency:** The async job queue (`surreal-commands` library) is tightly coupled to SurrealDB. If switching databases, the entire job queue mechanism needs reimplementation.
