# Stable Podcast - Root CLAUDE.md

This file provides architectural guidance for contributors working on Stable Podcast at the project level.

## Project Overview

**Stable Podcast** is a standalone podcast generation tool extracted from the [Open Notebook](https://github.com/lfnovo/open-notebook) project. It provides a focused, purpose-built application for AI-powered podcast creation — from content ingestion through script generation, transcript editing, TTS audio synthesis, and final audio mixing.

**Key Values**: Simplicity, focused functionality, multi-provider AI support, self-hosted option, open-source transparency.

**Origin**: Extracted from [stablemischief/Stable-Notebook](https://github.com/stablemischief/Stable-Notebook), which is a fork of Open Notebook. See `planning/` for the full extraction dependency analysis.

---

## Architecture

> **NOTE**: Architecture and tech stack are TBD — to be defined during the planning phase.
> See `planning/dependency-analysis.md` for the full dependency map from the source project.
> The planning phase will determine whether we keep the same stack (Next.js + FastAPI + SurrealDB)
> or simplify based on what the podcast pipeline actually requires.

---

## Code Style & Conventions

### Python (Backend)
- **Variables/Functions:** snake_case
- **Classes:** PascalCase
- **Constants:** UPPER_SNAKE_CASE
- **Files:** snake_case
- **Type hints:** Required on all function signatures (Pydantic v2 enforced)
- **Async:** All I/O functions must be async (`async def` + `await`)
- **Imports:** Standard library -> third-party -> local, separated by blank lines

### TypeScript (Frontend)
- **Variables/Functions:** camelCase
- **Components:** PascalCase
- **Files:** kebab-case for pages/routes, PascalCase for components
- **Props:** Interface with PascalCase name (e.g., `EpisodeCardProps`)
- **i18n:** All user-facing strings must use translation keys

---

## Logging Rules

**Library:** Loguru (Python backend)

**Import:** `from loguru import logger`

**Log Levels:**
- `logger.debug()` — Detailed diagnostic info (development only)
- `logger.info()` — Key operations: API calls, workflow steps, model provisioning
- `logger.warning()` — Recoverable issues: fallback models, retry attempts
- `logger.error()` — Failures requiring attention: DB errors, provider failures
- `logger.exception()` — In except blocks only (includes traceback)

**DO NOT:**
- Log sensitive data (API keys, credentials, user content)
- Use print() statements — always use logger
- Log inside tight loops without rate limiting

---

## Available Commands (PIV Loop Workflow)

**Core PIV Loop** — the primary development workflow: PRIME -> PLAN -> EXECUTE -> COMMIT

| Command | Description |
|---------|-------------|
| `/core_piv_loop/prime` | Load project context at the start of every session |
| `/core_piv_loop/plan` | Create structured implementation plan for a feature |
| `/core_piv_loop/execute` | Implement a plan step-by-step with validation |
| `/commit` | Create well-structured git commit |
| `/end-to-end-feature` | Chain all 4 core commands for autonomous feature development |
| `/create-prd` | Generate a Product Requirements Document from conversation |

**GitHub Bug Fix** — investigate and fix issues from GitHub:

| Command | Description |
|---------|-------------|
| `/github_bug_fix/rca` | Root cause analysis for a GitHub issue |
| `/github_bug_fix/implement-fix` | Implement fix based on RCA document |

**Validation** — quality assurance and process improvement:

| Command | Description |
|---------|-------------|
| `/validation/code-review` | Technical code review for quality and bugs |
| `/validation/code-review-fix` | Fix issues found in code review |
| `/validation/execution-report` | Generate implementation report after execution |
| `/validation/system-review` | Meta-analysis of plan vs implementation for process improvement |

**Standard Workflow:**
1. `/core_piv_loop/prime` — Load context (start of every session)
2. `/core_piv_loop/plan "feature description"` — Plan the feature
3. Review the plan in `.agents/plans/`
4. `/core_piv_loop/execute .agents/plans/[feature].md` — Implement
5. `/validation/code-review` — Review changes
6. `/commit` — Commit when satisfied

---

## Project Status

**Phase**: Planning
**Current Focus**: Defining architecture, tech stack, and extraction strategy from Open Notebook

---

**Last Updated**: March 2026
