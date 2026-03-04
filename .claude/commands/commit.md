---
description: "Create a well-structured git commit with meaningful message"
---

# Commit: Create Structured Git Commit

## INPUT

**Files to Commit:** $ARGUMENTS (optional - if not provided, commits all changes)

**Context:**
- Recent implementation work (ideally just ran `/execute`)
- Understanding of what changed and why
- Knowledge of project's commit message conventions

## PROCESS

Create a meaningful, well-structured git commit that clearly communicates what changed and why.

### Step 1: Analyze Changes

**Check current git status:**
```bash
git status
```

**Review all changes:**
```bash
# See staged changes
git diff --cached

# See unstaged changes
git diff

# See both
git diff HEAD
```

**Understand the scope:**
- What files were added?
- What files were modified?
- What files were deleted?
- Are there any untracked files that should be included?

### Step 2: Research Commit Message Style

**Check recent commits for style:**
```bash
git log -10 --oneline
git log -5 --format="%h %s"
```

**Look for patterns:**
- Do commits use conventional commits? (feat:, fix:, docs:, etc.)
- Are they imperative mood? ("Add feature" vs "Added feature")
- Do they reference issues/tickets?
- Are they short and descriptive or detailed?
- Do they use emojis?

### Step 3: Categorize the Change

**Determine change type:**
- **feat**: New feature or capability
- **fix**: Bug fix
- **docs**: Documentation changes only
- **style**: Code style/formatting (no logic change)
- **refactor**: Code restructuring (no behavior change)
- **perf**: Performance improvement
- **test**: Adding or updating tests
- **chore**: Build process, dependencies, tooling
- **ci**: CI/CD configuration changes

### Step 4: Stage Files

**Stage specific files** (if not already staged):
```bash
# If specific files provided as arguments
git add [file1] [file2] [file3]

# OR if committing all changes
git add -A

# OR selectively based on what makes sense
git add [relevant files for this logical change]
```

**Verify staging:**
```bash
git status
git diff --cached --stat
```

### Step 5: Draft Commit Message

**Structure:**

```
[type]([optional scope]): [short description]

[optional body - explain what and why, not how]

[optional footer - breaking changes, issue references]

Co-Authored-By: Claude <noreply@anthropic.com>
```

**Guidelines:**
- **Subject line** (first line):
  - Keep under 72 characters
  - Use imperative mood ("Add" not "Added" or "Adds")
  - Don't end with a period
  - Be specific but concise

- **Body** (optional, after blank line):
  - Explain WHAT changed and WHY
  - Don't explain HOW (code shows that)
  - Wrap at 72 characters
  - Use bullet points if multiple changes

- **Footer** (optional):
  - Note breaking changes: `BREAKING CHANGE: description`
  - Reference issues: `Closes #123` or `Fixes #456`
  - Always include co-author tag for AI assistance

**Examples:**

```
feat(auth): add JWT token refresh mechanism

Implement automatic token refresh to improve user experience
by preventing unexpected logouts during active sessions.

- Add refresh token endpoint
- Implement token rotation logic
- Update authentication middleware

Co-Authored-By: Claude <noreply@anthropic.com>
```

```
fix: resolve race condition in user session cleanup

Sessions were occasionally being cleaned up while still in use,
causing intermittent 401 errors for active users.

Fixes #234

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Step 6: Create Commit

**Important:** Use a heredoc for proper formatting:

```bash
git commit -m "$(cat <<'EOF'
feat(scope): short description

Longer explanation if needed.
Can span multiple lines.

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

**Verify commit was created:**
```bash
git log -1 --format="%H %s"
git show --stat HEAD
```

### Step 7: Final Checks

**Review the commit:**
```bash
# See the commit details
git show HEAD

# See files changed
git show --stat HEAD

# See the diff
git diff HEAD~1 HEAD
```

**Run post-commit validation** (if project has it):
```bash
# Any post-commit hooks or checks
[project-specific validation]
```

## OUTPUT

**Commit Summary Report:**

```markdown
# Commit Created Successfully

## Commit Details
- **Hash:** [commit hash]
- **Type:** [feat/fix/docs/etc.]
- **Scope:** [if applicable]
- **Message:** [commit subject line]

## Files Changed
[List from git show --stat]

## Full Commit Message

```
[Show the complete commit message]
```

## Verification
✅ Commit created successfully
✅ All changed files included
✅ Message follows project conventions
✅ Co-author tag included

## Next Steps
- Review with `git show HEAD`
- Push to remote when ready with `git push`
- Create pull request if needed
```

## Important Notes

**DO NOT:**
- ❌ Use `--no-verify` flag (skips hooks)
- ❌ Force push to main/master
- ❌ Commit sensitive data (.env, credentials, API keys)
- ❌ Create empty commits (unless intentional with `--allow-empty`)
- ❌ Amend commits that have been pushed (unless you know what you're doing)
- ❌ Commit generated files (build artifacts, node_modules, etc.)

**DO:**
- ✅ Review changes before committing
- ✅ Write clear, descriptive messages
- ✅ Follow project's commit conventions
- ✅ Include co-author tag for AI-assisted work
- ✅ Keep commits atomic (one logical change per commit)
- ✅ Test before committing

## Handling Commit Failures

**If pre-commit hooks fail:**
1. Read the error message carefully
2. Fix the issues (linting, tests, etc.)
3. Re-stage the fixes: `git add [fixed files]`
4. Create a NEW commit (don't amend!)
5. The hook failure means the commit DID NOT happen

**If commit message is rejected:**
1. Check project's commit message conventions
2. Adjust message format
3. Try again with corrected message

## Success Criteria

- [ ] All relevant changes are staged
- [ ] Commit message clearly describes what changed
- [ ] Commit message explains why it changed
- [ ] Message follows project conventions
- [ ] Co-author tag included
- [ ] No sensitive data committed
- [ ] Pre-commit hooks passed (if any)
- [ ] Commit verified with `git show HEAD`

---

**After committing, you can push to remote or create a pull request as needed.**
