---
description: Prime agent with codebase understanding
---

# Prime: Load Project Context

## Objective

Build comprehensive understanding of the codebase by analyzing structure, documentation, and key files. This command should be run at the start of any feature implementation to catch the AI agent up to speed on the project.

## Process

### 1. Analyze Project Structure

List all tracked files:
```bash
git ls-files
```

Show directory structure (if tree is available):
```bash
# On Linux/Mac with tree installed
tree -L 3 -I 'node_modules|__pycache__|.git|dist|build|venv|.venv'

# Alternative using find
find . -type d -not -path '*/node_modules/*' -not -path '*/__pycache__/*' -not -path '*/.git/*' -not -path '*/dist/*' -not -path '*/build/*' -not -path '*/.venv/*' | head -50
```

### 2. Read Core Documentation

Read these files in order:

1. **Global Rules:** Read `CLAUDE.md` or `agents.md` (most important)
2. **Product Requirements:** Read `.agents/PRD.md` or `PRD.md` (if exists)
3. **Project README:** Read `README.md`
4. **Architecture Docs:** Read any files in `docs/` or `.agents/docs/` that describe architecture

### 3. Identify and Read Key Files

Based on the structure and technology, identify and read:

**Configuration Files:**
- Python: `pyproject.toml`, `requirements.txt`, `setup.py`
- JavaScript/TypeScript: `package.json`, `tsconfig.json`
- Go: `go.mod`
- Rust: `Cargo.toml`
- Java: `pom.xml`, `build.gradle`

**Main Entry Points:**
- Python: `main.py`, `app.py`, `__main__.py`, `src/main.py`
- JavaScript/TypeScript: `index.ts`, `main.ts`, `src/index.ts`, `app.ts`
- Go: `main.go`
- Rust: `src/main.rs`

**Core Application Files:**
- API routers/controllers
- Database models/schemas
- Core service files
- Shared utilities

**Task-Specific Guides (if relevant):**
- Read guides in `.agents/reference/` or `reference/` that apply to the current task

### 4. Understand Current State

Check recent development activity:

```bash
# Recent commits
git log -10 --oneline

# Current branch
git branch --show-current

# Working directory status
git status
```

## Output Report

Provide a concise summary structured as follows:

---

### Project Overview
- **Purpose:** [What this application does]
- **Type:** [Web API, CLI tool, library, web app, etc.]
- **Current Version:** [If available from package files]
- **Status:** [Active development, stable, etc.]

### Tech Stack
- **Primary Language(s):** [Language and version]
- **Framework(s):** [Main frameworks]
- **Database:** [Database technology if applicable]
- **Build Tool:** [npm, uv, cargo, etc.]
- **Package Manager:** [How dependencies are managed]
- **Testing Framework:** [How tests are run]

### Architecture
- **Overall Pattern:** [MVC, vertical slice, microservices, etc.]
- **Project Structure:** [Key directories and their purposes]
  ```
  [show main directory structure]
  ```
- **Key Components:** [Main modules and their responsibilities]
- **Integration Points:** [How different parts connect]

### Core Principles & Conventions
- **Code Style:** [Naming conventions, formatting]
- **Type Safety:** [Level of type enforcement]
- **Error Handling:** [How errors are managed]
- **Logging:** [Logging approach and standards]
- **Testing:** [Testing philosophy and coverage]

### Current State
- **Active Branch:** [Current git branch]
- **Recent Development Focus:** [Based on recent commits]
  - [Most recent work areas]
  - [Patterns observed]
- **Pending Changes:** [If any uncommitted work]

### Key Observations
- [Important patterns noticed]
- [Potential concerns or areas of focus]
- [Recommendations for working with this codebase]

---

**Make this summary easy to scan - use bullet points and clear headers. Keep it to 1-2 pages when possible.**

## Success Criteria

After running this command, you should be able to:
- [ ] Explain what this project does at a high level
- [ ] Know which files to modify for common tasks
- [ ] Understand the coding conventions and patterns used
- [ ] Know how to run, test, and build the project
- [ ] Identify the main architectural components and how they relate

## Notes

- This command should be run in a **fresh conversation** before starting any feature work
- If the project is very large, focus on the areas relevant to your current task
- Update this command if you discover it's missing critical context for your projects
- Save the output report for reference during the implementation phase
