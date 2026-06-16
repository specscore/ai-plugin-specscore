---
captured_by: user
status: queued
---
# Update the idea skill to transition artifact status via the change-status CLI instead of manual file edits

Surfaced while running the ideate/specify flow on an external project. The idea skill in THIS repo (skills/idea/SKILL.md) instructs the agent to change an Idea's Status by hand-editing the body **Status:** line (e.g. 'Update Status: Draft -> Approved in the body metadata'). That conflicts with the 'prefer stable CLI contracts over ad-hoc file writes' tenet and causes frontmatter/body/index drift -> lint violations (status-mirror, idea-index-row-sync) needing migrate/--fix to recover.

Fix: replace manual-status-edit guidance with the dedicated verb 'specscore idea change-status <slug> --to=<status>' (and the analogous feature verb). Companion seed in specstudio-skills covers the ideate/specify skills there.
