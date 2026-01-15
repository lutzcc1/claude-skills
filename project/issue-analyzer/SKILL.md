---
name: issue-analyzer
description: Analyzes issues from any tracker (Linear, GitHub, Jira), explores the codebase for context, and creates implementation plan files. Use when starting work on an issue. Takes issue URL/ID as input. Invokes implementation-plan-writer to create the plan file.
user-invocable: true
---

# Issue Analyzer

This skill helps analyze issues from any tracking system (Linear, GitHub, Jira) in the context of the Sprout Rails codebase. It gathers all necessary context (issue details, comments, codebase patterns) and then invokes the `implementation-plan-writer` skill to create a standardized plan file.

## Supported Issue Trackers

| Tracker  | URL Pattern                                        | Example                                            |
| -------- | -------------------------------------------------- | -------------------------------------------------- |
| Linear   | `https://linear.app/{org}/issue/{ID}/...`          | `https://linear.app/bonsai/issue/PROD-123/...`     |
| GitHub   | `https://github.com/{owner}/{repo}/issues/{num}`   | `https://github.com/omc/sprout/issues/456`         |
| Jira     | `https://{org}.atlassian.net/browse/{KEY}`         | `https://omc.atlassian.net/browse/PROJ-789`        |

## Instructions

Follow these steps in order:

### 1. Detect Issue Source & Get Information

First, identify the issue tracker from the user input:

#### Linear Issues

**Detection:** URL contains `linear.app` or ID matches pattern like `PROD-123`, `ENG-456`

**Fetch details:**
```
Use mcp__linear-server__get_issue with the issue ID
Use mcp__linear-server__list_comments with the issue ID
```

**Extract:**
- Issue ID, title, description
- State, priority, labels
- Assignee, creator
- Comments and discussion
- Git branch name (from issue's branch field)

#### GitHub Issues

**Detection:** URL contains `github.com` and `/issues/`

**Fetch details:**
```bash
gh issue view <number> --json title,body,state,labels,assignees,comments,url
```

**Extract:**
- Issue number, title, body
- State, labels
- Assignees
- Comments and discussion

#### Jira Issues

**Detection:** URL contains `atlassian.net/browse/` or ID matches pattern like `PROJ-123`

**Fetch details:**
```
Use WebFetch to retrieve the Jira issue page
Parse the issue title, description, status, and comments
```

**Note:** Jira access may be limited without API tokens. If web fetch fails, ask the user to provide:
- Issue title and description
- Acceptance criteria
- Any relevant comments

#### Fallback

**If unclear:** Ask the user:
- Which issue tracker is this from?
- Can you provide the issue URL or paste the issue content?

### 2. Create the Branch

After identifying the issue:

1. **Extract branch name:**
   - **Linear:** Look for the git branch name in the issue's branch field (format: `username/PROD-123-descriptive-name`)
   - **GitHub:** Create from issue number and title (format: `username/issue-123-descriptive-name`)
   - **Jira:** Create from issue key and title (format: `username/PROJ-123-descriptive-name`)

2. **Create the branch:**
```bash
git checkout main
git pull origin main
git checkout -b <branch-name>
```

3. **Verify:** Ensure the branch was created successfully. ALL subsequent work must happen in this branch, never on main.

### 3. Analyze the Issue

Present a clear summary of the issue:

1. **Issue Overview:**
   - Tracker source (Linear/GitHub/Jira)
   - ID/Number and title
   - Current state and priority (if available)
   - Link to original issue

2. **Description Analysis:**
   - Parse the issue description
   - Identify key requirements
   - Note any technical specifications
   - Highlight acceptance criteria

3. **Comments Review:**
   - Summarize important discussion points
   - Note any clarifications or decisions made
   - Identify blockers or dependencies mentioned

### 4. Codebase Context Search

**Use the Task tool with subagent_type=Explore** to search the codebase for relevant context:

1. **Identify related components:**
   - Models involved (Account, Cluster, User, etc.)
   - Services that might need changes
   - Controllers/views if UI changes needed
   - Background jobs if async work required

2. **Search for existing patterns:**
   - Look for similar features already implemented
   - Find relevant service objects to reference
   - Identify related actions/state machines if applicable
   - Check for existing tests to use as examples
   - Find similar UI/UX usage

3. **Check for integration points:**
   - External APIs that might be affected
   - Database schema changes needed
   - Feature flags to consider

### 5. Clarify Requirements

- Review the analysis and identify any ambiguities
- If business logic is unclear, list specific questions
- If technical approach needs validation, flag for discussion
- DO NOT proceed with assumptions

### 6. Create Implementation Plan File

After gathering all context (issue details, comments, codebase exploration), invoke the `implementation-plan-writer` skill to create the plan file:

```
Use the Skill tool with skill: "implementation-plan-writer"
```

The `implementation-plan-writer` skill will:
- Create an `<ISSUE-ID>-IMPLEMENTATION-PLAN.md` file in the project root
- Use the standardized plan template with all required sections
- Incorporate all the context you've gathered in previous steps

**Important:** When the skill is invoked, make sure to provide all the context gathered:
- Issue source (Linear/GitHub/Jira)
- Issue ID, title, description, and URL
- Key requirements and acceptance criteria
- Relevant comments and decisions
- Codebase patterns and components discovered during exploration
- Any clarifications or ambiguities identified

### 7. Request Approval

After the plan file is created, present it and explicitly ask:

- "Does this implementation approach make sense?"
- "Should I proceed with creating the tasks?"
- "Are there any concerns about this plan?"

## Issue Tracker-Specific Notes

### Linear
- Full MCP integration available
- Can fetch complete issue details programmatically
- Branch names often pre-configured in issue

### GitHub
- Use `gh` CLI for full access
- Comments are part of the issue view
- May need to parse markdown formatting

### Jira
- Limited access via web fetch
- May require user to paste issue content
- Consider asking for API token if frequent access needed

## Error Handling

- **Invalid issue ID/URL:** Ask user to verify the format
- **Linear MCP fails:** Suggest checking MCP server configuration
- **GitHub `gh` fails:** Suggest `gh auth login`
- **Jira access denied:** Ask user to paste issue content manually
- **Issue has no description:** Ask user if they need clarification from stakeholders
- **Codebase search yields no context:** Suggest broader search terms

## Notes

- This skill gathers context and delegates plan creation to `implementation-plan-writer`
- Be honest about complexity - if something is unclear, say so
- Challenge the approach if you see a better way
- Consider suggesting breaking large issues into smaller ones if needed
- Use WebSearch when researching API documentation or third-party integrations

Remember: Your task is to gather context and invoke `implementation-plan-writer`. Do not implement changes. ASK FOR APPROVAL before starting work on the TODO list.
