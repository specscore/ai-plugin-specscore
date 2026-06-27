---
format: https://specscore.md/plan-specification
status: Approved
---
# Plan: Lifecycle Status Skill Task Kind

**Status:** Implemented
**Source Feature:** lifecycle-status-skill
**Date:** 2026-06-26
**Owner:** alex
**Supersedes:** —
**Parent:** specscore:implementation-commit-provenance

## Summary

Adds **Task-kind dispatch** to the `specscore:change-status` skill — the `ai-plugin-specscore` slice of the cross-repo `implementation-commit-provenance` effort, and a sub-plan of the master `specscore:implementation-commit-provenance`. Scope is the increment only: the Task picker row, the description/frontmatter update, and the new `references/task.md`. The skill's original ACs (preflight, no-fallback, success-relay, deep-links) are unchanged by this increment and were implemented by the original `lifecycle-status-skill` plan; they are deferred here.

## Approach

Two small, linear tasks gated on the upstream `specscore task change-status` verb existing (sub-plan `cli-task-change-status`, which this plan's master sequences first). Task 1 edits `SKILL.md` (picker row + description); Task 2 authors the standalone `references/task.md`. Both feed `AC: dispatch-picker-present`; Task 1 also satisfies the description half of `AC: skill-shipped`. Note: `references/feature.md` and `references/idea.md` (the other two files `AC: dispatch-picker-present` asserts exist) pre-date this increment and are not touched here — only the new `Task` row and `references/task.md` are added.

## Tasks

### Task 1: Add the Task picker row and update SKILL.md frontmatter

**Verifies:** lifecycle-status-skill#ac:skill-shipped, lifecycle-status-skill#ac:dispatch-picker-present
**Depends-On:** —
**Status:** done

Add the `Task → references/task.md` row to the kind-dispatch picker in `skills/change-status/SKILL.md`, and extend the `description` frontmatter to name the `Task` kind and its action vocabulary ("complete a task"), so the skill matcher ranks for Task-transition intents.

### Task 2: Author `references/task.md`

**Verifies:** lifecycle-status-skill#ac:dispatch-picker-present
**Depends-On:** 1
**Status:** done

Create the standalone `skills/change-status/references/task.md`: the exact `specscore task change-status` invocation, the legal `--to` values, the optional `--repo`/`--commit`/`--branch` provenance flags (valid only with `--to=complete`), the `--plan` flag and dual board/plan-inline target resolution, and the inherited exit-code table — citing the upstream `cli/task/change-status` contract.

## Deferred AC Coverage

- lifecycle-status-skill#ac:preflight-enforced — unchanged by the Task-kind increment; implemented and verified by the original lifecycle-status-skill plan.
- lifecycle-status-skill#ac:no-silent-fallback — unchanged by this increment (the REQ prose was generalized to all kinds, but the behavior is kind-agnostic); covered by the original lifecycle-status-skill plan.
- lifecycle-status-skill#ac:success-line-relayed — unchanged by this increment; covered by the original lifecycle-status-skill plan.
- lifecycle-status-skill#ac:deep-links-survive — unchanged by this increment; covered by the original lifecycle-status-skill plan.

## Open Questions

None at this time.

---
*This document follows the https://specscore.md/plan-specification*
