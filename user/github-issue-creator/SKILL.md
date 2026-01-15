---
name: github-issue-creator
description: Creates well-structured GitHub issues formatted for future Claude instances to work on. Use when the user asks to create a GitHub issue or wants to document a task/bug for later implementation.
---

# GitHub Issue Creator

This skill helps create comprehensive GitHub issues that provide all the context a future Claude instance needs to successfully complete the task.

## Important 🚨

DO NOT TO CREATE AN IMPLEMENTATION PLAN. Your task is to create a GH issue with all context needed so another Claude instance can work on the implementation plan.

## Instructions

When this skill is invoked:

1. **Gather Context**
   - Understand the task/bug/feature the user wants to document
   - Ask clarifying questions if the request is vague
   - Identify relevant files, functions, or areas of the codebase
   - Note any constraints, requirements, or acceptance criteria

2. **Research the Codebase** (if applicable)
   - Use `ast-grep` (preferred) or `grep` to find relevant code locations
   - Read related files to understand current implementation
   - Identify dependencies or related systems
   - Note any existing patterns or conventions to follow

3. **Research external services** (if applicable)
   - Use websearch to research documentation on any third party service involved in the feature
   - Use websearch to research best practices to interact with this third party service

4. **Structure the Issue**

   Use this template format:

   ```markdown
   ## Summary

   [1-2 sentence overview of what needs to be done]

   ## Context

   [Why this is needed, background information, business justification]

   ## Current State

   [What exists today, relevant code locations with file:line references]

   ## Desired State

   [What should exist after this issue is resolved]

   ## Acceptance Criteria

   - [ ] Specific, testable requirement 1
   - [ ] Specific, testable requirement 2
   - [ ] Tests are passing

   ## Technical Notes

   [Implementation hints, architectural decisions, potential challenges. But not an implementation guide]

   ## Related Files

   - `path/to/file.rb:123` - Description of relevance
   - `path/to/test.rb:45` - Related test file

   ## References

   [Links to docs, similar PRs, design docs, API documentation]
   ```

5. **Create the Issue**
   - Use `gh issue create --title "..." --body "$(cat <<'EOF' ... EOF)"`
   - Use a clear, actionable title (e.g., "Add user authentication", "Fix race condition in job processing")
   - Include labels if appropriate (`--label bug`, `--label enhancement`, etc.)
   - Assign to a project or milestone if relevant

6. **Confirm with User**
   - Show the created issue URL
   - Confirm the issue has all necessary context

## Best Practices

- **Be specific:** Avoid vague descriptions. Include exact file paths and line numbers.
- **Provide context:** A future Claude instance won't have the conversation history.
- **Include examples:** Show expected input/output or before/after code snippets.
- **Reference conventions:** Point to AGENTS.md, CLAUDE.md, or style guides.
- **Link related issues:** Use `#123` syntax to reference related issues or PRs.
- **Consider the future reader:** Write as if explaining to a teammate who just joined.

## Examples

### Example 1: Feature Request

```bash
gh issue create \
  --title "Add appointment confirmation via WhatsApp" \
  --label enhancement \
  --body "$(cat <<'EOF'
## Summary
Implement automatic appointment confirmations sent via WhatsApp when a customer books through Digitail.

## Context
Currently, we only send reminders for existing appointments. Customers have requested immediate confirmation when they book to ensure the appointment was registered.

## Current State
- `app/services/digitail/appointments_sync.rb:45` handles new appointment detection
- `app/clients/zendesk/notifications/client.rb:23` sends WhatsApp messages
- Confirmation messages are not currently sent for new appointments

## Desired State
When a new appointment is detected in Digitail sync, send a WhatsApp confirmation message to the customer within 5 minutes.

## Acceptance Criteria
- [ ] New appointments trigger confirmation message
- [ ] Message is sent within 5 minutes of creation
- [ ] Message includes appointment date, time, and veterinarian name
- [ ] Duplicate messages are prevented (idempotency check)
- [ ] Tests cover success and failure cases
- [ ] Follows Spanish-only messaging requirement (see AGENTS.md)

## Technical Notes
- Use existing `Zendesk::Notifications::Client#send_message` method
- Add new job `SendConfirmationJob` similar to `SendReminderJob`
- Respect notification idempotency constraints (see AGENTS.md)
- Ensure timezone handling follows project rules (AGENTS.md)

## Related Files
- `app/services/digitail/appointments_sync.rb` - Entry point for new appointments
- `app/jobs/send_reminder_job.rb` - Similar pattern to follow
- `spec/jobs/send_reminder_job_spec.rb` - Test pattern reference

## References
- Zendesk Notifications API: [link to docs]
- Project conventions: See AGENTS.md > Notification Idempotency
EOF
)"
```

### Example 2: Bug Report

```bash
gh issue create \
  --title "Fix race condition in webhook deduplication" \
  --label bug \
  --body "$(cat <<'EOF'
## Summary
Zendesk webhook retries sometimes create duplicate message records despite unique index.

## Context
Users report seeing duplicate responses to the same WhatsApp message during high traffic periods.

## Current State
- `app/models/message.rb:5` has unique index on `zendesk_id`
- `app/controllers/webhooks_controller.rb:23` uses `find_or_create_by`
- Race condition can occur between check and create

## Desired State
Webhook processing is atomic and never creates duplicates, even under high concurrency.

## Acceptance Criteria
- [ ] No duplicate messages created during concurrent webhook delivery
- [ ] Existing unique constraint is preserved
- [ ] Error handling gracefully manages constraint violations
- [ ] Tests include concurrent request simulation
- [ ] No breaking changes to webhook API

## Technical Notes
Consider using `find_or_create_by!` with rescue, or database-level upsert if supported by Rails 8.

## Related Files
- `app/controllers/webhooks_controller.rb:23` - Current implementation
- `spec/requests/webhooks_spec.rb` - Add concurrency test here

## References
- Rails upsert: https://api.rubyonrails.org/classes/ActiveRecord/Persistence/ClassMethods.html#method-i-upsert
EOF
)"
```

## Notes

- The `gh` CLI must be authenticated (`gh auth login`)
- Always use heredoc syntax for multiline bodies to preserve formatting
- Include the `--web` flag if the user wants to open the issue in browser immediately
- This GH issue will be taken by another claude instance. Ensure it has everything it may need.
