---
type: sidekick-seed
slug: idea-and-feature-skill-docs-should-show-status-query-recipes
captured_at: 2026-06-04T00:00:00Z
captured_by: user
captured_during: null
trigger: explicit
status: queued
synchestra_task: null
---

# idea and feature skill docs should show status query recipes so agents don't grep

The `specscore:idea` and `specscore:feature` skills describe `list` but an agent can miss that status is available via flags, and fall back to grepping `**Status:**` out of files by hand (observed in practice).

The flags already exist:
- `specscore idea list --format json|yaml` — includes status per idea.
- `specscore idea list --status <X>` — filter by lifecycle status.
- `specscore feature list --fields status` — yaml with status per feature.

Fix: add an explicit "list with status" recipe to the `idea` and `feature` skill reference docs (skills/idea/references/list.md, skills/feature/references/list.md) so agents reach for the flag instead of grepping. Low-cost docs change; no CLI work.

Captured from a session where an agent grepped statuses manually despite the flags being one argument away.
