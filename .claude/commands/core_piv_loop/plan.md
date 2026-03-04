---
description: "Create comprehensive feature plan with deep codebase analysis and research"
---

# Plan: Create Implementation Plan for a Feature

## INPUT

**Feature Request:** $ARGUMENTS

**Required Context:**
- Global rules (CLAUDE.md) - loaded automatically
- Product Requirements (PRD.md) - if exists
- Recent priming output - run `/prime` first if not done recently
- Task-specific reference guides - will be identified during planning

## PROCESS

Transform a feature request into a **comprehensive implementation plan** through systematic codebase analysis, external research, and strategic planning.

**Core Principle**: We do NOT write code in this phase. Our goal is to create a context-rich implementation plan that enables one-pass implementation success for AI agents.

**Key Philosophy**: Context is King. The plan must contain ALL information needed for implementation - patterns, mandatory reading, documentation, validation commands - so the execution agent succeeds on the first attempt.

### Phase 1: Feature Understanding

**Deep Feature Analysis:**

- Extract the core problem being solved
- Identify user value and business impact
- Determine feature type: New Capability/Enhancement/Refactor/Bug Fix
- Assess complexity: Low/Medium/High
- Map affected systems and components

**Create or Refine User Story:**

```
As a <type of user>
I want to <action/goal>
So that <benefit/value>
```

**Clarify Ambiguities:**
- If requirements are unclear, ask the user to clarify before continuing
- Get specific implementation preferences (libraries, approaches, patterns)
- Resolve architectural decisions before proceeding

### Phase 2: Codebase Intelligence Gathering

**1. Project Structure Analysis**

- Detect primary language(s), frameworks, and runtime versions
- Map directory structure and architectural patterns
- Identify service/component boundaries and integration points
- Locate configuration files
- Find environment setup and build processes

**2. Pattern Recognition**

Search for similar implementations in the codebase:
- Identify coding conventions (naming patterns, file organization)
- Extract error handling approaches
- Document logging patterns and standards
- Find common patterns for the feature's domain
- Document anti-patterns to avoid
- Reference CLAUDE.md for project-specific rules

**3. Dependency Analysis**

- Catalog external libraries relevant to feature
- Understand how libraries are integrated
- Find relevant documentation in docs/, .agents/reference/, or ai-wiki
- Note library versions and compatibility requirements

**4. Testing Patterns**

- Identify test framework and structure
- Find similar test examples for reference
- Understand test organization (unit vs integration)
- Note coverage requirements and testing standards

**5. Integration Points**

- Identify existing files that need updates
- Determine new files that need creation and their locations
- Map router/API registration patterns
- Understand database/model patterns if applicable
- Identify authentication/authorization patterns if relevant

### Phase 3: External Research & Documentation

**Documentation Gathering:**

- Research latest library versions and best practices
- Find official documentation with specific section anchors
- Locate implementation examples and tutorials
- Identify common gotchas and known issues
- Check for breaking changes and migration guides

**Technology Best Practices:**

- Research current best practices for the technology stack
- Find relevant blog posts, guides, or case studies
- Identify performance optimization patterns
- Document security considerations

**Compile Research References:**

```markdown
## Relevant Documentation

- [Library Official Docs](https://example.com/docs#section)
  - Specific feature implementation guide
  - Why: Needed for X functionality
- [Framework Guide](https://example.com/guide#integration)
  - Integration patterns section
  - Why: Shows how to connect components
```

### Phase 4: Deep Strategic Thinking

**Critical Questions:**

- How does this feature fit into the existing architecture?
- What are the critical dependencies and order of operations?
- What could go wrong? (Edge cases, race conditions, errors)
- How will this be tested comprehensively?
- What performance implications exist?
- Are there security considerations?
- How maintainable is this approach?

**Design Decisions:**

- Choose between alternative approaches with clear rationale
- Design for extensibility and future modifications
- Plan for backward compatibility if needed
- Consider scalability implications

**PRD Validation (if PRD exists):**
- Read `.agents/PRD.md` or `PRD.md` if it exists
- Verify plan preserves architectural patterns defined in PRD
- Validate against any architectural principles or design constraints

### Phase 5: Plan Structure Generation

Create a comprehensive plan document following the template below.

## OUTPUT

**Output Location:** `.agents/plans/{kebab-case-feature-name}.md`

Create the `.agents/plans/` directory if it doesn't exist.

**Plan Template:**

```markdown
# Feature: <feature-name>

**IMPORTANT:** Validate documentation, codebase patterns, and task sanity before implementing!

Pay special attention to naming of existing utils, types, and models. Import from the right files.

## Feature Description

<Detailed description of the feature, its purpose, and value to users>

## User Story

As a <type of user>
I want to <action/goal>
So that <benefit/value>

## Problem Statement

<Clearly define the specific problem or opportunity this feature addresses>

## Solution Statement

<Describe the proposed solution approach and how it solves the problem>

## Feature Metadata

**Feature Type**: [New Capability/Enhancement/Refactor/Bug Fix]
**Estimated Complexity**: [Low/Medium/High]
**Primary Systems Affected**: [List of main components/services]
**Dependencies**: [External libraries or services required]

---

## CONTEXT REFERENCES

### Relevant Codebase Files

**IMPORTANT: READ THESE FILES BEFORE IMPLEMENTING!**

- `path/to/file.ext` (lines X-Y) - Why: [Specific reason]
- `path/to/another.ext` - Why: [Pattern to follow/understand]

### New Files to Create

- `path/to/new_file.ext` - [Purpose and what it will contain]
- `tests/path/to/test_file.ext` - [Test coverage for new functionality]

### Relevant Documentation

**READ THESE BEFORE IMPLEMENTING!**

- [Documentation Title](https://url.com#section)
  - Specific section: [Section name]
  - Why: [Why this is needed]

### Patterns to Follow

**Naming Conventions:**
```[language]
// Example from codebase
[actual code showing naming pattern]
```

**Error Handling:**
```[language]
// Example from codebase
[actual error handling pattern]
```

**Logging Pattern:**
```[language]
// Example from codebase
[actual logging pattern]
```

**Other Relevant Patterns:**
[Any other patterns specific to this feature]

---

## IMPLEMENTATION PLAN

### Phase 1: Foundation

<Describe foundational work needed>

**Tasks:**
- [Foundational task 1]
- [Foundational task 2]

### Phase 2: Core Implementation

<Describe main implementation work>

**Tasks:**
- [Core task 1]
- [Core task 2]

### Phase 3: Integration

<Describe how feature integrates with existing functionality>

**Tasks:**
- [Integration task 1]
- [Integration task 2]

### Phase 4: Testing & Validation

<Describe testing approach>

**Tasks:**
- [Testing task 1]
- [Testing task 2]

---

## STEP-BY-STEP TASKS

**IMPORTANT:** Execute every task in order, top to bottom. Each task is atomic and independently testable.

### Task Format

Use information-dense keywords:
- **CREATE**: New files or components
- **UPDATE**: Modify existing files
- **ADD**: Insert new functionality into existing code
- **REMOVE**: Delete deprecated code
- **REFACTOR**: Restructure without changing behavior
- **MIRROR**: Copy pattern from elsewhere in codebase

### Task 1: [ACTION] [target]

- **IMPLEMENT**: [Specific implementation detail]
- **PATTERN**: [Reference to existing pattern - file:line]
- **IMPORTS**: [Required imports and dependencies]
- **GOTCHA**: [Known issues or constraints to avoid]
- **VALIDATE**: `[executable validation command]`

### Task 2: [ACTION] [target]

[Continue with all tasks in dependency order...]

---

## TESTING STRATEGY

### Unit Tests

[Scope and requirements based on project standards]

Design unit tests with fixtures and assertions following existing approaches.

### Integration Tests

[Scope and requirements based on project standards]

### Edge Cases

[List specific edge cases that must be tested for this feature]

---

## VALIDATION COMMANDS

Execute every command to ensure zero regressions and 100% feature correctness.

### Level 1: Syntax & Style

```bash
[Project-specific linting command]
[Project-specific formatting command]
[Project-specific type checking command]
```

**Expected**: All commands pass with exit code 0

### Level 2: Unit Tests

```bash
[Project-specific unit test command]
[Project-specific coverage command]
```

**Expected**: All tests pass, coverage meets requirements

### Level 3: Integration Tests

```bash
[Project-specific integration test command]
```

**Expected**: All integration tests pass

### Level 4: Manual Validation

[Feature-specific manual testing steps:]
1. [Manual test step 1]
2. [Manual test step 2]

---

## ACCEPTANCE CRITERIA

- [ ] Feature implements all specified functionality
- [ ] All validation commands pass with zero errors
- [ ] Unit test coverage meets requirements
- [ ] Integration tests verify end-to-end workflows
- [ ] Code follows project conventions and patterns
- [ ] No regressions in existing functionality
- [ ] Documentation is updated (if applicable)
- [ ] Performance meets requirements (if applicable)
- [ ] Security considerations addressed (if applicable)

---

## COMPLETION CHECKLIST

- [ ] All tasks completed in order
- [ ] Each task validation passed immediately
- [ ] All validation commands executed successfully
- [ ] Full test suite passes (unit + integration)
- [ ] No linting errors
- [ ] No formatting errors
- [ ] No type checking errors
- [ ] Build succeeds (if applicable)
- [ ] All acceptance criteria met
- [ ] Code reviewed for quality and maintainability

---

## NOTES

[Additional context, design decisions, trade-offs, or important considerations]

```

## Quality Criteria

### Context Completeness ✓
- [ ] All necessary patterns identified and documented
- [ ] External library usage documented with links
- [ ] Integration points clearly mapped
- [ ] Gotchas and anti-patterns captured
- [ ] Every task has executable validation command

### Implementation Ready ✓
- [ ] Another developer could execute without additional context
- [ ] Tasks ordered by dependency (can execute top-to-bottom)
- [ ] Each task is atomic and independently testable
- [ ] Pattern references include specific file:line numbers

### Pattern Consistency ✓
- [ ] Tasks follow existing codebase conventions
- [ ] New patterns justified with clear rationale
- [ ] No reinvention of existing patterns or utils
- [ ] Testing approach matches project standards

### Information Density ✓
- [ ] No generic references (all specific and actionable)
- [ ] URLs include section anchors when applicable
- [ ] Task descriptions use codebase keywords
- [ ] Validation commands are executable (non-interactive)

## Success Metrics

**One-Pass Implementation**: Execution agent can complete feature without additional research or clarification

**Validation Complete**: Every task has at least one working validation command

**Context Rich**: Plan passes "No Prior Knowledge Test" - someone unfamiliar with codebase can implement using only Plan content

## Report Summary

After creating the plan, provide:

1. **Summary**: Brief overview of feature and approach
2. **Plan Location**: Full path to created plan file
3. **Complexity Assessment**: Low/Medium/High
4. **Key Risks**: Implementation risks or considerations
5. **Confidence Score**: X/10 that execution will succeed on first attempt

---

**Next Steps:** Review the plan, make any necessary adjustments, then run `/execute [plan-file]` to implement.
