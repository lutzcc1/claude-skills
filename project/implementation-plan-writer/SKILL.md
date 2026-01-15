---
name: implementation-plan-writer
description: Creates a high-level implementation plan file for issues from any tracker (Linear, GitHub, Jira) that another Claude Code instance can use as a guide. Use when the user asks to create an implementation plan file or document a feature's implementation approach.
---

# Implementation Plan Writer

This skill creates a standalone markdown file (`<ISSUE-ID>-IMPLEMENTATION-PLAN.md`) containing a high-level implementation plan for an issue from any tracker (Linear, GitHub, Jira). The plan is designed to guide another Claude Code instance through implementing the feature.

## Instructions

Follow these steps in order:

### 1. Get Issue Information

First, extract the issue identifier from user input:

**Linear:**
- **If given a URL** (e.g., `https://linear.app/bonsai/issue/PROD-123/...`): Extract the issue ID (e.g., `PROD-123`)
- **If given just an ID** (e.g., `PROD-123`): Use it directly

**GitHub:**
- **If given a URL** (e.g., `https://github.com/omc/sprout/issues/456`): Extract the issue number and repo
- **If given just a number** (e.g., `#456`): Use it with repository context

**Jira:**
- **If given a URL** (e.g., `https://omc.atlassian.net/browse/PROJ-789`): Extract the issue key
- **If given just a key** (e.g., `PROJ-789`): Use it directly

**If unclear:** Ask the user for the issue ID/URL and which tracker it's from

### 2. Create Implementation Plan File

Create a file named `<ISSUE-ID>-IMPLEMENTATION-PLAN.md` in the project root with the following structure:

```markdown
# [ISSUE-ID]: [Issue Title]

## Overview

[2-3 sentence summary of what needs to be implemented and why]

### Issue Tracker

- **Source:** [Linear | GitHub | Jira]
- **Link:** [URL to the original issue]
- **Status:** [Current status/state]

## Requirements

- Bullet point list of key requirements from the issue
- Include acceptance criteria
- Note any constraints or dependencies

## Technical Approach

[High-level description of how to implement this, written in plain English]

### Architecture Decision

[Explain the overall approach - e.g., "Use a service object to orchestrate the process", "Add a state machine action for multi-step async flow", etc.]

### Components Affected

- **Models:** [List models that need changes]
- **Services:** [New or modified service objects]
- **Controllers:** [If UI changes needed]
- **Views:** [If UI changes needed]
- **Jobs (application services called async):** [If background processing needed]
- **Database:** [If migrations needed]

## Implementation Steps

[High-level steps in plain English or pseudocode - NOT detailed Ruby code]

### Phase 1: [Name]

1. Step description
2. Another step
   - Sub-point if needed

### Phase 2: [Name]

1. Step description
2. Another step

[Add more phases if needed]

## Pseudocode

[OPTIONAL: basic pseudocode to illustrate the flow - NOT Ruby syntax]

// Example - keep it simple and language-agnostic
service CreateClusterCredential
  validate cluster exists
  validate user has permission

  if external_api_needed
    call external API
    handle response
  end

  create credential record
  notify user
  return success
```

## Testing Strategy

- **Service tests:** [Which services need tests and what to test]
- **Integration tests:** [User flows to test end-to-end]
- **Edge cases:** [Specific scenarios to handle]
- **Manual testing:** [Steps to verify in UI/console]

## Considerations & Risks

- **Performance:** [Any N+1 query concerns, need for caching, etc.]
- **Security:** [Authorization checks, data validation, etc.]
- **Error handling:** [What can go wrong and how to handle it]
- **External dependencies:** [API rate limits, third-party service reliability, etc.]

## References

- [Link to similar feature implementation in codebase]
- [Link to relevant documentation]
- [Links to related issues in the same tracker]

## Questions & Decisions

[Document any ambiguities or decisions that need stakeholder input]

- Question 1?
- Question 2?
```

### 3. Writing Guidelines

When creating the implementation plan:

**DO:**
- Write in plain English, not code
- Keep it high-level and strategic, but with enough detail for the coding agent to be able to work on it.
- Focus on WHAT needs to be done and WHY
- Reference existing patterns in the codebase
- Include very basic pseudocode if it helps clarify flow
- Mention Sprout-specific conventions (Service objects, Actions, Errgonomic, etc.)
- Think about edge cases and error scenarios
- Consider performance and security implications
- Include the issue tracker source and link in the plan

**DON'T:**
- Write actual Ruby/Javascript code
- Make it too prescriptive (allow flexibility in implementation)
- Skip the "why" behind decisions
- Forget to mention testing approach
- Assume context - explain enough for someone new to understand

### 4. Present the Plan

After creating the file:

1. Show the user the file path
2. Give a brief summary of the plan
3. Ask if they want any adjustments

## Sprout-Specific Guidelines

When writing plans, reference these Sprout conventions:

### Service Objects
- Business logic lives in service objects inheriting from `ApplicationService`
- Services return `Service::Success`, `Service::Failure`, or `Service::Retry`
- Use Errgonomic presence checks for validations
- Initialize instance variables in `call` method

### Actions/State Machines
- Use Actions for multi-step async operations needing audit trail
- Inherit from `Action` model with states: new, running, success, error
- Each step recorded in `steps` table

### Testing
- Use Oaken seeds from `test/seeds/` directory
- Test in `test/with_seeds/` directory (not `test/with_fixtures/`)
- Focus on business logic, not trivial stuff
- Avoid creating records in tests - use seeds instead

### Code Style
- Keep controllers thin - delegate to services
- Use Hotwire (Stimulus + Turbo) over custom JavaScript
- Follow RuboCop conventions
- RESTful controller patterns

## Important Reminders

- **DO NOT write any actual implementation code** - this is just a plan
- **DO NOT start implementation** - this skill only creates the plan file
- **DO create todos** - this is helpful to outline the plan
- **DO be critical** - challenge unclear requirements or suggest better approaches
- **DO use WebSearch** - look up best practices, API docs, or patterns when needed
- **DO keep it concise** - this is a guide, not a detailed specification
- **DO ask clarifying questions** - if something is ambiguous, flag it in the plan

## Error Handling

- If issue ID is invalid: Ask user to verify
- If issue tracker connection fails: Suggest checking credentials/MCP configuration
- If issue has insufficient detail: Note this in the "Questions & Decisions" section
- If codebase search yields no context: Use broader search terms or note the need for new patterns

Remember: This skill creates a strategic implementation guide. Be clear and focused on helping another Claude instance understand the approach.
