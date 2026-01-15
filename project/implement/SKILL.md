---
name: implement
description: Implements a feature from an existing implementation plan file (PROD-XXX-IMPLEMENTATION-PLAN.md). Fetches the linked Linear issue for context, asks clarifying questions, then implements ONE PHASE AT A TIME waiting for user approval between each phase.
user-invocable: true
---

# Implement

This skill executes an implementation plan that was previously created. It reads the plan file, fetches the linked Linear issue for full context, asks any necessary clarifying questions, and then **implements the feature one phase at a time, waiting for user approval after each phase before proceeding to the next**.

## CRITICAL: Extended Thinking Required

**YOU MUST USE EXTENDED THINKING (ultrathink) FOR ALL IMPLEMENTATION WORK IN THIS SKILL.**

When implementing code, think deeply about:

- Architecture decisions and trade-offs
- Edge cases and error handling
- Security implications
- Performance considerations
- Test coverage strategy

## Instructions

Follow these steps in order:

### 1. Find the Implementation Plan File

Look for an implementation plan file in the project root:

- **Pattern:** `PROD-*-IMPLEMENTATION-PLAN.md` or `<LINEAR-ISSUE-ID>-IMPLEMENTATION-PLAN.md`
- **If argument provided** (e.g., `/implement PROD-808`): Look for `PROD-808-IMPLEMENTATION-PLAN.md`
- **If no argument:** Search for any `*-IMPLEMENTATION-PLAN.md` file in the root

```
Use Glob tool with pattern: "*-IMPLEMENTATION-PLAN.md"
```

If multiple plan files exist and no argument was provided, ask the user which one to implement.

If no plan file is found:

- Inform the user that no implementation plan was found
- Suggest running `/linear-issue-analyzer` first to create one

### 2. Read and Parse the Plan

Read the implementation plan file and extract:

1. **Linear Issue ID** - from the title or filename (e.g., `PROD-808`)
2. **Overview** - understand the high-level goal
3. **Requirements** - acceptance criteria and constraints
4. **Technical Approach** - architecture decisions
5. **Implementation Steps** - phased steps to follow
6. **Testing Strategy** - what tests to write
7. **Considerations & Risks** - gotchas to watch for
8. **Questions & Decisions** - any unresolved items

### 3. Fetch Linear Issue for Context

Use the MCP Linear server to get full issue context:

```
Use mcp__linear-server__get_issue with the extracted issue ID
Use mcp__linear-server__list_comments with the issue ID
```

This provides:

- Full issue description and latest updates
- Comments with clarifications or decisions made since the plan was created
- Current status and any blockers

### 4. Verify Branch

Ensure you're on the correct feature branch:

- Check `git branch --show-current`
- The branch should match the Linear issue (e.g., `luis/prod-808-...`)
- If on `main` or wrong branch, alert the user and DO NOT proceed

### 5. Review and Ask Clarifying Questions

Before implementing, review everything and identify:

1. **Unresolved Questions** - from the plan's "Questions & Decisions" section
2. **Ambiguities** - requirements that aren't clear enough to implement
3. **Missing Information** - gaps between the plan and issue comments
4. **Architecture Decisions** - choices that need user confirmation

**Use the AskUserQuestion tool** to get clarification on anything unclear.

DO NOT proceed with implementation if there are blocking questions. It's better to ask now than to implement incorrectly.

### 6. Create Phase-Based Todo List

Use the TodoWrite tool to create a task list with **phases as the top-level items**:

- Each phase from the implementation plan becomes a todo item
- DO NOT break phases into sub-tasks yet - that happens when you start each phase
- Mark only Phase 1 as `in_progress` when starting

Example:

```
- Phase 1: Create the service class [in_progress]
- Phase 2: Add the controller action [pending]
- Phase 3: Write tests [pending]
- Phase 4: Final linting and verification [pending]
```

### 7. PHASE-BY-PHASE IMPLEMENTATION LOOP

**THIS IS THE CORE WORKFLOW. YOU MUST FOLLOW THIS EXACTLY.**

```
┌─────────────────────────────────────────────────────────┐
│  FOR EACH PHASE:                                        │
│                                                         │
│  1. Implement ONLY that phase                           │
│  2. Run relevant tests                                  │
│  3. Show diff and summary                               │
│  4. STOP and wait for user approval                     │
│  5. Only proceed to next phase after explicit approval  │
└─────────────────────────────────────────────────────────┘
```

#### For Each Phase:

**Step A: Implement the Phase**

WITH EXTENDED THINKING (ultrathink):

1. **Read relevant files first** - understand existing code before modifying
2. **Follow Sprout conventions:**
   - Service objects for business logic (inherit from `ApplicationService`)
   - Service signals: `Service::Success`, `Service::Failure`, `Service::Retry`
   - Errgonomic presence checks for validations
   - Actions for multi-step async operations
   - Oaken seeds for test data
   - Hotwire (Stimulus + Turbo) for frontend
3. **Keep changes focused** - ONLY implement what this phase specifies
4. **Write tests if this phase includes them** (see Testing section below)

**Step B: Run Tests**

```
rails test <specific_test_file>  # Run tests related to this phase
rubocop -a                       # Auto-fix safe issues
```

**Step C: Present Phase Changes**

After completing the phase:

1. **Run `git diff`** - show changes made in THIS PHASE ONLY
2. **Summary** - brief description of what was implemented
3. **Files modified** - list with brief descriptions
4. **Tests added/passing** - test status for this phase

**Step D: STOP AND WAIT**

```
╔═══════════════════════════════════════════════════════════════╗
║  🛑 PHASE [N] COMPLETE - WAITING FOR YOUR FEEDBACK            ║
║                                                               ║
║  Please review the changes above and let me know:             ║
║  • "Approved" or "LGTM" → I'll proceed to Phase [N+1]         ║
║  • Feedback/changes → I'll address them before continuing     ║
╚═══════════════════════════════════════════════════════════════╝
```

**DO NOT proceed to the next phase until the user explicitly approves.**

Acceptable approval signals:

- "approved", "lgtm", "looks good", "continue", "proceed", "next phase", "ok", "yes"

If the user provides feedback or requests changes:

1. Make the requested changes
2. Show the updated diff
3. Wait for approval again

### 8. Testing Strategy

Tests should be written as part of the relevant phase (usually a dedicated testing phase):

1. **Service tests** - in `test/with_seeds/services/`
2. **Integration tests** - if UI changes involved
3. **Use Oaken seeds** - never create records in tests
4. **Focus on business logic** - not trivial stuff

### 9. Final Phase: Lint and Verify

The last phase should always include:

```
rubocop -a  # Auto-fix safe issues
rails test  # Run full test suite
```

Fix any issues that arise before presenting for final review.

### 10. Implementation Complete

After ALL phases are approved:

1. **Final summary** - overview of everything implemented
2. **All files changed** - complete list
3. **Test coverage** - all tests added
4. **Any deviations** - if you deviated from the plan, explain why

**DO NOT COMMIT.** Ask the user if they want you to commit or if they'll do it manually.

## Error Handling

- **Plan file not found:** Suggest running `/linear-issue-analyzer` first
- **Linear issue fetch fails:** Continue with plan file content, but warn user
- **On wrong branch:** Alert user and stop - do not implement on main
- **Tests failing:** Do not mark task complete until tests pass
- **Ambiguous requirements:** Use AskUserQuestion, do not assume

## Important Reminders

- **ONE PHASE AT A TIME** - this is the most critical rule; implement phase, show diff, wait for approval
- **ALWAYS use extended thinking** - this is critical for quality implementation
- **Read before writing** - never modify code you haven't read
- **Follow the plan** - but flag if you see a better approach
- **Ask questions** - when unclear, ask; don't assume
- **Keep it simple** - avoid over-engineering
- **Update todos** - mark phase complete only after user approval
- **Run rubocop** - lint code before presenting phase for review
- **NEVER COMMIT** - the user must explicitly request a commit
- **WAIT FOR APPROVAL** - do not proceed to the next phase without explicit user approval

## After Each Phase

**STOP and present the diff.** Wait for the user to:

1. Review the phase changes
2. Approve ("lgtm", "approved", "continue") → proceed to next phase
3. Request changes → address them, show new diff, wait again

## After All Phases Complete

Once the final phase is approved:

1. Present complete summary
2. Ask user if they want you to commit or will do it manually
3. Suggest running `/create-pr` when ready
