---
description: "Execute a feature plan step-by-step with validation"
---

# Execute: Implement a Feature Plan

## INPUT

**Plan File:** $ARGUMENTS

**Required:**
- Path to a plan file (e.g., `.agents/plans/add-authentication.md`)
- The plan must have been created using `/plan` command
- Fresh conversation recommended (or at least recent `/prime`)

## PROCESS

Implement a feature by executing the plan document step-by-step with continuous validation.

### Step 1: Read and Validate Plan

**Read the entire plan file:**
- Understand the feature description and goals
- Review all context references
- Identify files to read and documentation to reference
- Note validation commands for each task

**Validate plan structure:**
- Ensure all required sections are present
- Check that tasks are clear and actionable
- Verify validation commands are executable

### Step 2: Load Required Context

**Read all referenced files:**
- Read every file listed in "Relevant Codebase Files"
- Understand patterns and conventions from these files
- Note integration points

**Review referenced documentation:**
- Skim external documentation links (don't deep dive unless needed)
- Understand key concepts from documentation
- Note any version-specific considerations

**Load task-specific guides:**
- If plan references guides in `.agents/reference/` or `reference/`, read them
- Understand specialized patterns for this type of work

### Step 3: Execute Tasks Sequentially

**For each task in the STEP-BY-STEP TASKS section:**

1. **Read the task carefully**
   - Understand what needs to be done
   - Note the pattern references
   - Check for gotchas
   - Identify validation command

2. **Implement the task**
   - Follow existing patterns exactly
   - Import from correct files
   - Use project naming conventions
   - Add appropriate logging
   - Handle errors according to project standards

3. **Validate immediately**
   - Run the validation command specified in the task
   - Fix any errors before proceeding
   - Do NOT move to next task until this one passes

4. **Report progress**
   - Briefly describe what was implemented
   - Confirm validation passed
   - Note any deviations from plan (with justification)

**Important Rules:**
- Execute tasks in order (top to bottom)
- Never skip a task
- Never skip validation
- Fix errors immediately before proceeding
- If blocked, stop and ask for guidance

### Step 4: Run All Validation Commands

After all tasks are complete, run the full validation suite:

**Level 1: Syntax & Style**
```bash
[Run all linting/formatting/type-checking commands from plan]
```

**Level 2: Unit Tests**
```bash
[Run unit tests as specified in plan]
```

**Level 3: Integration Tests**
```bash
[Run integration tests as specified in plan]
```

**Level 4: Manual Validation**
- Follow manual testing steps from plan
- Test the feature as a user would
- Verify all acceptance criteria are met

### Step 5: Final Verification

**Check acceptance criteria:**
- Go through each item in the ACCEPTANCE CRITERIA section
- Verify each one is met
- Document any that are not met and why

**Review changes:**
- List all files created or modified
- Confirm changes align with plan
- Check for any unintended side effects

## OUTPUT

**Implementation Summary Report:**

```markdown
# Implementation Complete: [Feature Name]

## Summary
[Brief description of what was implemented]

## Files Changed

### Created
- `path/to/new_file.ext` - [Purpose]

### Modified
- `path/to/existing_file.ext` - [What changed]

## Validation Results

### ✅ All Validation Levels Passed
- Level 1 (Syntax & Style): PASS
- Level 2 (Unit Tests): PASS
- Level 3 (Integration Tests): PASS
- Level 4 (Manual Validation): PASS

### Test Coverage
- Unit tests: [X tests added/modified]
- Integration tests: [X tests added/modified]
- Coverage: [percentage if available]

## Acceptance Criteria Status

- [✅] Criterion 1
- [✅] Criterion 2
- [❌] Criterion 3 - [Reason if not met]

## Deviations from Plan

[List any intentional deviations from the plan with justification]
OR
No deviations - plan was followed exactly.

## Known Issues

[List any known issues or limitations]
OR
None - all functionality working as expected.

## Next Steps

1. Review the implementation
2. Test the feature manually
3. Create a commit with `/commit`
4. [Any other next steps]
```

## Success Criteria

- [ ] All tasks in plan executed in order
- [ ] All task-level validations passed
- [ ] All validation commands passed (all 4 levels)
- [ ] All acceptance criteria met (or documented why not)
- [ ] No errors in console/logs
- [ ] Feature works as described in user story

## Common Pitfalls to Avoid

1. **Skipping validation commands** - Always run them immediately after each task
2. **Not reading referenced files** - These contain critical patterns you must follow
3. **Ignoring gotchas** - They're called out for a reason
4. **Deviating without justification** - Stick to the plan unless there's a good reason
5. **Not testing manually** - Automated tests don't catch everything
6. **Batch executing multiple tasks** - Do one at a time with validation

## Notes

- If you encounter errors that can't be quickly fixed, STOP and report the issue
- If the plan is unclear or incorrect, STOP and ask for clarification
- If you discover missing requirements, STOP and discuss with user
- Document all deviations and decisions in the output report
- This command should result in a working, tested, validated feature ready for commit

---

**After execution is complete and all validations pass, use `/commit` to create a structured git commit.**
