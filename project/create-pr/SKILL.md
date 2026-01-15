---
name: create-pr
description: Creates a well-formatted pull request for Sprout Rails project. Runs linting, reviews changes, and generates PR with proper title and description following project conventions. Use when the user asks to create a PR or open a pull request.
user-invocable: true
---

# PR Creator

This skill helps create well-formatted pull requests following Sprout's conventions, ensuring code quality checks pass before creating the PR.

## Instructions

Follow these steps in order:

### 1. Check Current State

First, verify the current branch and git status:

```bash
git status
git log main..HEAD --oneline
git diff main...HEAD --stat
```

- Confirm you're not on the main branch (never create PRs from main)
- Review the commits that will be included in the PR
- Understand the scope of changes

### 2. Extract Linear Issue Context

Extract the Linear issue ID from the branch name and fetch issue details:

#### Branch Name Pattern

Branch names typically follow the pattern: `username/PROD-123-description` or `PROD-123-description`

Extract the issue ID using regex pattern: `PROD-\d+`

#### Fetch Issue Details

If a Linear issue ID is found in the branch name:

```
Use mcp__linear-server__get_issue with the issue ID (e.g., "PROD-123")
```

Also fetch any comments for additional context:

```
Use mcp__linear-server__list_comments with the issue ID
```

#### Information to Extract

From the Linear issue, gather:

- **Title**: Use this as the base for the PR title
- **Description**: Understand the full scope and requirements
- **Acceptance Criteria**: Ensure the PR addresses all criteria
- **Labels**: Note any relevant labels (bug, feature, etc.)
- **Comments**: Check for clarifications, decisions, or additional requirements

#### No Linear Issue Found

If no Linear issue ID is found in the branch name:

- Ask the user if there's an associated Linear issue
- If no issue exists, proceed without Linear context but note this in the process

### 3. Run Linting

Before creating the PR, ensure code quality:

```bash
rubocop
```

If there are any violations:

1. **Attempt auto-fix:**

   ```bash
   rubocop -a
   ```

2. **If issues remain, fix manually or ask user:**

   - Review the violations
   - If simple, fix them following RuboCop rules
   - If complex or unclear, ask the user how they want to proceed

3. **Commit linting fixes (if any) with user approval:**
   - Wait for user's explicit approval before committing
   - Use message: `style: fix rubocop violations`

### 4. Analyze Changes

Review all changes that will be included in the PR:

```bash
git diff main...HEAD
```

Understand:

- What feature/fix/refactor is being implemented
- Which files are affected
- The overall purpose and impact of the changes

**IMPORTANT:** Look at ALL commits in the branch, not just the latest one, to understand the full scope of the PR.

### 5. Capture Screenshots for UI Changes

If the changes include UI modifications (views, stylesheets, JavaScript, Stimulus controllers), capture screenshots using Chrome DevTools MCP:

#### Identifying UI Changes

Look for changes in:

- `.html.erb` files (views)
- `.css` or `.scss` files (stylesheets)
- `.js` files (JavaScript)
- Stimulus controllers
- Any frontend-related files

#### Using Chrome DevTools MCP Tools

**Available tools:**

- `mcp__chrome-devtools__list_pages` - List all open browser tabs
- `mcp__chrome-devtools__select_page` - Select a page to work with
- `mcp__chrome-devtools__take_screenshot` - Capture visual screenshots
- `mcp__chrome-devtools__take_snapshot` - Capture text-based accessibility tree (smaller, faster)
- `mcp__chrome-devtools__navigate_page` - Navigate to a URL if page isn't open
- `mcp__chrome-devtools__resize_page` - Set viewport size for consistent screenshots

#### Screenshot Workflow

1. **Ask user** if they have the relevant pages open in their browser

   - If not, ask for URLs or offer to navigate using `mcp__chrome-devtools__navigate_page`

2. **List available pages:**

   ```
   Use mcp__chrome-devtools__list_pages to see all open tabs
   Returns: Array of pages with index, URL, and title
   ```

3. **Select the target page:**

   ```
   Use mcp__chrome-devtools__select_page with pageIdx parameter
   Example: pageIdx: 0 (for the first page in the list)
   ```

4. **Optionally resize viewport for consistency:**

   ```
   Use mcp__chrome-devtools__resize_page
   Example: width: 1280, height: 720
   ```

5. **Take screenshot:**

   **Full page screenshot (recommended for major changes):**

   ```
   Use mcp__chrome-devtools__take_screenshot
   Parameters:
   - fullPage: true
   - format: "png"
   - filePath: "/path/to/screenshots/feature-name.png"
   ```

   **Viewport screenshot only:**

   ```
   Use mcp__chrome-devtools__take_screenshot
   Parameters:
   - format: "png"
   - filePath: "/path/to/screenshots/feature-name-viewport.png"
   ```

   **Specific element screenshot:**

   ```
   First, take a snapshot to get element UIDs:
   Use mcp__chrome-devtools__take_snapshot

   Then use the uid from snapshot:
   Use mcp__chrome-devtools__take_screenshot
   Parameters:
   - uid: "element-uid-from-snapshot"
   - filePath: "/path/to/screenshots/component.png"
   ```

6. **Repeat for before/after if applicable:**
   - Capture screenshots of both old behavior (from main branch) and new behavior
   - Use consistent naming: `feature-before.png` and `feature-after.png`

#### Screenshot Guidelines

- **Focus on major changes only** - Don't capture minor tweaks (font size, margins, hover effects)
- **Use descriptive filenames** - `pods-index-filters.png` not `screenshot1.png`
- **Consider responsive views** - Capture mobile/tablet if layout changes affect them
- **Save to temporary location** - Screenshots don't need to be committed to repo
- **Include in PR description** using markdown:

  ```markdown
  ## UI

  ### Pods Index Filters

  ![Pods index with new filters](path/to/screenshot.png)

  ### Pod Form Layout

  ![Improved pod form layout](path/to/pod-form.png)
  ```

#### Notes

- Screenshots are **optional** - only include for significant UI changes
- If pages aren't open, you can navigate to them using `mcp__chrome-devtools__navigate_page`
- Text snapshots (`take_snapshot`) are smaller and faster but don't show visual styling
- Full page screenshots capture the entire page, including content below the fold

### 6. Generate PR Title and Description

#### Title Format

The title of the PR must match the title of the Linear issue, prepended by the Linear issue ID.
Example: `[PROD-746] Create script to deprovision cluster`

**Do not** escape backticks when writing titles or descriptions (This is a `method_name` example).

If no Linear issue is associated, use a descriptive title following conventional commit style:

- `feat: Add new feature name`
- `fix: Resolve issue description`
- `refactor: Improve code structure`

#### Description Format

Create a comprehensive PR description using context from the Linear issue and git changes:

**Using Linear Issue Context:**

- Use the issue description to inform the Summary section
- Reference acceptance criteria when listing changes
- Include any decisions or clarifications from issue comments
- Link to the Linear issue in the Related section

```markdown
## Summary

[1-3 sentence overview of what this PR does and why. Use context from the Linear issue description to explain the business value and purpose.]

## Changes

- [Bulleted list of changes made]
- [Keep it high level, mention only the core changes. Avoid mentioning helpers or minor fixes]
- [Group related changes together]
- [Ensure changes address the acceptance criteria from the Linear issue]

## UI (Optional)

- [Optional: Include browser screenshots if the feature includes UI changes]
- [Optional: Take enough screenshots to include all **major** UI/UX changes. Avoid including minor changes (font resize, hovering effect, margin changes, etc.)]

## Notes (Optional)

[Optional: Include any important implementation details, design decisions, or trade-offs. Reference any technical discussions from Linear comments if relevant.]

## Related

- Linear: [PROD-XXX](https://linear.app/bonsai/issue/PROD-XXX)
- [Optional: Link to related PRs or documentation]
```

### 7. Check Branch Status

Before creating the PR, check if the branch needs to be pushed:

```bash
git status
```

If the branch is not tracking a remote or is behind:

```bash
git push -u origin HEAD
```

### 8. Create the PR

Use `gh pr create` with the `--draft` flag to create the PR in draft state:

```bash
gh pr create --draft --title "feat: Add feature name" --body "$(cat <<'EOF'
## Summary

[Your summary here]

## Changes

- [Change 1]
- [Change 2]

EOF
)"
```

**Important notes:**

- Always use `--draft` flag to create PRs in draft state (user will manually set to ready when they decide)
- Use heredoc with `<<'EOF'` to preserve formatting
- Do not escape backticks in code references
- Ensure the title matches the Linear issue title
- Make the summary clear enough for reviewers to understand the PR at a glance
- Do not mention "comprehensive tests" in the "Changes" section. It is implicit that tests were added/updated.

### 9. Return PR URL

After creating the PR, provide the URL to the user:

```
PR created: https://github.com/onemorecloud/sprout/pull/123
```

## Example Usage

**User:** "Create a PR"

**Skill actions:**

1. Check git status and branch
2. Extract Linear issue ID from branch name (e.g., `luis/PROD-123-feature` → `PROD-123`)
3. Fetch Linear issue details and comments using MCP tools
4. Run `rubocop` and fix any violations
5. Review commits: `git log main..HEAD`
6. Analyze changes: `git diff main...HEAD`
7. Capture screenshots if UI changes detected (using Chrome DevTools MCP)
8. Generate PR title from Linear issue title (e.g., `[PROD-123] Feature name`)
9. Generate PR description using Linear context and git changes
10. Push branch if needed: `git push -u origin HEAD`
11. Create PR with `gh pr create`
12. Return PR URL

## Error Handling

- **On main branch:** Warn user and ask them to checkout a feature branch first
- **Uncommitted changes:** Ask user if they want to commit changes first
- **RuboCop failures:** Attempt to auto-fix, then ask user to review remaining issues
- **No commits ahead of main:** Explain there are no changes to create a PR for
- **gh CLI not authenticated:** Provide instructions to run `gh auth login`
- **Push failures:** Check for branch protection or authentication issues
- **No browser pages open:** Ask user to open relevant pages or provide URLs to navigate to
- **Screenshot failures:** If MCP tools fail, continue without screenshots (they're optional)
- **No Linear issue ID in branch:** Ask user if there's an associated Linear issue, proceed without if none
- **Linear MCP connection fails:** Warn user and proceed without Linear context, use git changes only
- **Linear issue not found:** The issue ID may be invalid; ask user to verify or proceed without

## Quality Checklist

Before creating the PR, ensure:

- [ ] Not on main branch
- [ ] All commits follow project conventions
- [ ] RuboCop passes with no violations
- [ ] Linear issue fetched (if ID found in branch name)
- [ ] PR title matches Linear issue title (format: `[PROD-XXX] Issue title`)
- [ ] PR description incorporates Linear issue context (description, acceptance criteria)
- [ ] PR description is comprehensive and clear
- [ ] All changes are properly summarized
- [ ] Linear issue linked in Related section
- [ ] Screenshots included if UI changes are significant
- [ ] Branch is pushed to remote

## Notes

- This skill is non-committal for code changes - it only creates the PR
- If linting requires fixes, always get user approval before committing
- Be critical: If the changes don't seem ready for review, ask the user first
- Look at ALL commits in the branch, not just the latest one
- PR descriptions should help reviewers understand the context and changes quickly
- Remember to not escape backticks in titles or descriptions
- Linear context is preferred but optional - if no issue ID is found or MCP fails, proceed with git-only context
- The Linear issue provides valuable context: use the description to explain "why", acceptance criteria to verify completeness
