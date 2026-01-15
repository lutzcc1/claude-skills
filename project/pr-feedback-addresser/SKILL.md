---
name: pr-feedback-addresser
description: Addresses PR review comments for Sprout Rails project. Fetches PR comments via gh CLI, systematically addresses each comment, runs rubocop on changed files, and runs relevant tests to ensure nothing broke. Use this when the user asks to address PR feedback or review comments.
---

# PR Feedback Addresser

This skill helps address PR review comments systematically while following Sprout's Rails conventions and ensuring code quality.

## Instructions

Follow these steps in order:

### 1. Fetch PR Information

First, identify the current PR:

```bash
gh pr view --json number,title,url,comments,reviewThreads,files
```

If no PR is found for the current branch, ask the user for the PR number.

### 2. Parse and Organize Comments

- Extract all review comments and threads from the PR
- Group comments by file and line number
- Present comments to the user in a clear, organized format
- Ask the user if they want to address all comments or specific ones

### 3. Address Each Comment Systematically

For each comment to address, make an attack plan (use subagents to plan the code change):

1. **Read the relevant code** - Use the Read tool to view the file and surrounding context
2. **Understand the feedback** - Analyze what the reviewer is asking for
3. **Make a plan for how to address this comment** - If it's not a simple change, make a multi-steps plan to address the comment.
4. **Implement the change** following Sprout conventions
5. **Make focused edits** - Address the specific feedback without unnecessary changes

### 4. Quality Checks

After making changes:

1. **Run RuboCop on changed files:**

   ```bash
   git diff --name-only | grep '\.rb$' | xargs rubocop -a
   ```

2. **Fix any remaining RuboCop issues:**

   - Review remaining violations
   - Fix manually if autocorrect doesn't handle them
   - Run `rubocop -a` again if needed

3. **Identify and run relevant tests:**

   - For changed files in `app/`, run corresponding test files
   - For service changes: Run tests in `test/with_seeds/services/`
   - For controller changes: Run tests in `test/with_seeds/controllers/`
   - For model changes: Run tests in `test/with_seeds/models/`
   - Example: `rails test test/with_seeds/services/specific_service_test.rb`

4. **Address test failures:**
   - If tests fail, analyze the failure
   - Fix the issue following project conventions
   - Re-run tests to confirm fix
   - **Use Oaken seeds for test data** - Do not create/update records in tests unless absolutely necessary
   - Avoid using `cases/accounts/standard_lies` seed (deprecated)

### 5. Summary and Next Steps

After addressing all comments:

1. **Summarize changes made:**

   - List each comment addressed
   - Confirm all tests passing

2. **Prepare for commit:**

   - DO NOT create a commit automatically
   - Wait for user's explicit approval before committing
   - When you get commit approval, follow conventional commit format (one commit per addressed comment)

3. **Remind about workflow:**
   - User should review changes
   - Commit when ready with atomic, conventional commits
   - Push to update PR

## Example Usage

**User:** "Address the PR feedback"

**Skill actions:**

1. Fetch PR comments via `gh pr view`
2. Display organized list of comments
3. For each comment:
   - Show the feedback
   - Implement the requested change
   - Run rubocop on the file
   - Run relevant tests
4. Summarize all changes
5. Wait for user approval before committing

## Error Handling

- If `gh` CLI is not authenticated, provide instructions to run `gh auth login`
- If tests fail, fix issues before proceeding to next comment
- If RuboCop fails, address violations before continuing
- If a comment is unclear, ask the user for clarification

## Notes

- This skill is non-committal - it never creates commits without explicit approval
- Always validate changes with tests before marking a comment as addressed
- Be critical of suggested changes; challenge if they don't follow best practices
- Use WebSearch to research solutions when planning implementation
- NEVER respond to comments in GitHub, you're only allowed to read comments.
- Ignore not actionable comments ("love this", "good job", "nice touch", etc.)
