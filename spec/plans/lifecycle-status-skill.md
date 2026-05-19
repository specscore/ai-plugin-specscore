# Plan: Lifecycle Status Skill

**Status:** Approved
**Source Feature:** lifecycle-status-skill
**Date:** 2026-05-19
**Owner:** alexander.trakhimenok@gmail.com
**Supersedes:** —

## Summary

Decomposes the `lifecycle-status-skill` Feature into three linearly ordered tasks that together ship the new `specscore:change-status` skill, its two per-kind reference files, and the catalog entry that surfaces it in `skills/README.md`. Every Feature AC is covered; nothing is deferred.

## Approach

Tasks follow the file-creation grain of the deliverable: one task per artifact group. Task 1 lands `SKILL.md` (the discovery surface — frontmatter, pre-flight, kind picker). Task 2 lands both reference files (the per-kind CLI invocation contracts). Task 3 registers the skill in the plugin catalog and confirms the deep-links-preserved invariant. Ordering is forced — Task 2's reference files are listed in Task 1's picker, so the picker rows are stale until they exist; Task 3 verifies surrounding files are untouched, so it must come after the new files are in place.

## Tasks

### Task 1: Author `skills/change-status/SKILL.md`

**Verifies:** lifecycle-status-skill#ac:skill-shipped, lifecycle-status-skill#ac:preflight-enforced

Create `skills/change-status/SKILL.md` with the required YAML frontmatter (`name: change-status`, `user-invocable: true`, and a `description` string that names every intent phrase from `REQ: description-trigger-coverage` plus both kind names). The body contains the canonical CLI pre-flight block (matching `skills/feature/SKILL.md` and `skills/idea/SKILL.md`), a one-paragraph statement of the skill's responsibility, and the kind-dispatch picker table with rows for `Feature → references/feature.md` and `Idea → references/idea.md`.

### Task 2: Author `skills/change-status/references/feature.md` and `references/idea.md`

**Verifies:** lifecycle-status-skill#ac:dispatch-picker-present, lifecycle-status-skill#ac:no-silent-fallback, lifecycle-status-skill#ac:success-line-relayed

Create the two per-kind reference files. Each documents the exact `specscore <kind> change-status` invocation, the legal `--to` values and legal-transition matrix, the inherited exit-code table from the shared CLI contract, the no-silent-fallback rule (CLI exit codes are surfaced verbatim — never paraphrased, never substituted by a direct file edit), and the success-output relay rule (the single-line `<id>: <from> → <to>` stdout is emitted byte-for-byte). Each MAY cite the existing `skills/<kind>/references/change-status.md` but MUST stand alone enough that an agent never needs to backtrack.

### Task 3: Register skill in `skills/README.md` and verify deep-link survival

**Verifies:** lifecycle-status-skill#ac:deep-links-survive

Add a row for the new `change-status/` skill to `skills/README.md` — in a new "Cross-kind action skills" category (or equivalent placement that does not mis-classify a verb-level skill as a CLI-wrapper or infrastructure skill). Then verify, by diff against `origin/main`, that `skills/feature/SKILL.md`, `skills/idea/SKILL.md`, `skills/feature/references/change-status.md`, and `skills/idea/references/change-status.md` are byte-identical to their pre-Feature versions — explicitly satisfying both REQs that `AC: deep-links-survive` verifies. No edits to those four files are permitted under this task.

## Outstanding Questions

None at this time.

---
*This document follows the https://specscore.md/plan-specification*
