---
format: https://specscore.md/idea-specification
status: Specified
---

# Idea: Lifecycle Status Skill

**Status:** Specified
**Date:** 2026-05-19
**Owner:** alexander.trakhimenok@gmail.com
**Promotes To:** lifecycle-status-skill
**Supersedes:** —
**Related Ideas:** —

## Problem Statement

How might we make `specscore feature change-status` and `specscore idea change-status` the obvious path for an AI agent that needs to transition a SpecScore artifact's lifecycle status, instead of letting agents fall back to direct file edits plus `specscore spec lint --fix`?

## Context

Captured 2026-05-19 in sibling repo `synchestra-io/specstudio-skills` during a `specstudio:implement` dogfood session; moved here because the change lives in the `ai-plugin-specscore` plugin.

In that session, the agent missed `specscore feature change-status` and defaulted to a direct file edit followed by `specscore spec lint --fix` to flip a Feature's `**Status:**`. Both CLI verbs exist (`specscore feature change-status`, `specscore idea change-status`) and produce the correct result — including legal-transition validation and audit-event emission — but the lifecycle-transition capability is currently buried inside the seven-verb `specscore:feature` skill and the two-verb `specscore:idea` skill. Neither skill's `description` frontmatter advertises "use me to change status," unlike the focused `/sidekick`, `/ideate`, `/specify`, `/plan` patterns elsewhere in the ecosystem.

Related artifacts:
- [`skills/feature/SKILL.md`](../../skills/feature/SKILL.md) and [`skills/feature/references/change-status.md`](../../skills/feature/references/change-status.md)
- [`skills/idea/SKILL.md`](../../skills/idea/SKILL.md) and [`skills/idea/references/change-status.md`](../../skills/idea/references/change-status.md)

## Recommended Direction

Add a single dedicated skill — `specscore:change-status` — that wraps both `specscore feature change-status` and `specscore idea change-status` behind one entry point. The skill's `description` frontmatter explicitly advertises that whenever an AI agent needs to change the lifecycle status of any SpecScore artifact (Feature or Idea today; more kinds later), this skill is the canonical route.

The skill dispatches by kind: it asks the agent which artifact kind is being transitioned, validates the target status is a legal transition for that kind, runs the matching CLI verb, then reports the resulting `<from> → <to>` diff. The existing `specscore:feature` and `specscore:idea` skills keep their `change-status` reference docs as deep-links from the verb table — navigation-flow users (who started by listing features) still discover the verb in context.

A single unified skill is preferred over two split skills (`specscore:feature-status` + `specscore:idea-status`) because the action is the same shape regardless of kind ("transition an artifact's `**Status:**`"), and one entry point makes the skill description searchable for any kind without duplicating the trigger surface.

## Alternatives Considered

- **Two split skills (`specscore:feature-status` and `specscore:idea-status`).** Pros: each skill is laser-focused on one verb, mirroring the existing `skills/feature` / `skills/idea` split. Cons: doubles the skill count, fragments the trigger language, and forces the agent to disambiguate kind before picking a skill — duplicating work the unified skill can do internally. The split also scales poorly when more kinds (Task, Plan) eventually grow `change-status` verbs.
- **Leave `change-status` as a sub-reference inside the existing `specscore:feature` / `specscore:idea` skills (status quo).** Pros: zero new skills, zero duplication. Cons: this is exactly the discoverability failure the dogfood surfaced — the `description` strings on those skills are dominated by navigation verbs (list/tree/info/refs/deps/new), so agents searching for "change status" or "approve" or "deprecate" don't reliably retrieve them.
- **Inline status-changing prose into every methodology skill that needs it (`/specify`, `/plan`, `/implement`).** Pros: callers don't need to know about a separate skill. Cons: duplicates the validation/dispatch logic across every consumer, and the dogfood showed that even hand-rolled prose doesn't reliably trigger the right CLI verb.

## MVP Scope

One new skill at `skills/change-status/SKILL.md` whose `description` frontmatter triggers on phrases like "change status", "transition status", "approve idea", "approve feature", "archive idea", "mark stable", "deprecate feature" — for either Feature or Idea artifacts. It dispatches to a matching reference (`references/feature.md` or `references/idea.md`) which are thin pointers to the corresponding `change-status.md` files already shipping in `skills/feature` and `skills/idea`. No new CLI capability; no changes to the CLI contract. The MVP is shipped when an unprompted agent, given an `implement` task that lands a Feature, picks this skill instead of editing the file directly.

## Not Doing (and Why)

- Add a `specscore:lifecycle` umbrella skill that also handles list / info / refs verbs — out of scope; this skill is transitions only. List / info / refs already have a home.
- Build separate `feature-status` and `idea-status` skills — rejected above for discoverability and duplication reasons; one unified skill wins.
- Cover Task or Plan kind transitions — those CLI verbs do not exist yet on the published `specscore` 0.1.x line. The skill begins with Feature + Idea and extends when new kinds ship a `change-status` verb.
- Remove the existing `change-status` references from `skills/feature` and `skills/idea` — they stay as deep links so navigation-flow users still discover the verb in context.
- Re-implement validation in the skill — legal-transition checks and `spec lint --fix` follow-up are owned by the CLI; the skill is a thin wrapper that surfaces CLI exit codes verbatim.

## Key Assumptions to Validate

| Tier | Assumption | How to validate |
|------|------------|-----------------|
| Must-be-true | A skill with a transition-focused `description` will be retrieved by Claude Code's skill-matching layer when an agent reasons "I need to approve this Feature" or "I need to archive this Idea", in preference to the broader `specscore:feature` / `specscore:idea` skills. | Run the same `implement` dogfood that produced the original miss, with the new skill installed, and observe whether the agent picks `specscore:change-status` unprompted. Repeat across 3+ unrelated transition prompts. |
| Must-be-true | The CLI's legal-transition matrix is stable enough to encode in skill prose without immediate drift. | Diff `specscore feature change-status --help` and `specscore idea change-status --help` against the matrix the new skill cites, on every CLI bump; gate releases on parity. |
| Should-be-true | A single unified skill outperforms two split skills on retrieval, even though the description must mention both "Feature" and "Idea". | A/B the unified description against a hypothetical split via the existing skill-creator eval harness on a corpus of transition-intent prompts. |
| Should-be-true | Keeping the deep-link references in `skills/feature` and `skills/idea` does not cause double-retrieval problems (both the dedicated skill and the parent skill scoring high). | Observe retrieval rankings on representative prompts; if the parent skills out-rank the dedicated one, soften their `description` strings to de-emphasize change-status. |
| Might-be-true | The same skill will scale cleanly to Task and Plan kinds when those CLI verbs ship, by adding another reference file under `references/` and one line to the dispatch table. | Defer until those kinds exist; revisit the skill shape at that point. |

## SpecScore Integration

- **New Features this would create:** [`lifecycle-status-skill`](../features/lifecycle-status-skill/README.md) — the dedicated skill artifact, its trigger description, dispatch behavior, CLI pre-flight, and exit-code translation.
- **Existing Features affected:** none in this repo yet (the existing `specscore:feature` and `specscore:idea` skills are unaffected; their `change-status` references stay).
- **Dependencies:** the `specscore` CLI's `feature change-status` and `idea change-status` verbs (already shipped in 0.1.x).

## Open Questions

- Should the dedicated skill's slug be `change-status` or `lifecycle-status` in the plugin's `skills/` namespace? `change-status` matches the CLI verb name and the existing reference filename; `lifecycle-status` more directly mirrors the conceptual surface ("lifecycle"). Decision deferred to the Feature spec.

---
*This document follows the https://specscore.md/idea-specification*
