# Implementation Plan: CLI Install Skill

**Source idea:** [`docs/ideas/cli-install-skill.md`](../docs/ideas/cli-install-skill.md)

## Overview

Ship `/specscore:install` — a single skill that wraps the existing `curl -fsSL https://specscore.md/get-cli | sh` installer — and document a pre-flight convention that every future CLI-wrapper skill in this plugin will follow. The plan is markdown-only: three files touched, no runtime code, no binary distribution, no hooks. Verification is end-to-end manual install from a clean machine.

## Architecture Decisions

These decisions resolve the "Open Questions" section of the one-pager before implementation begins. They are made *here* (at plan time) rather than per-task so implementation never stalls on a style call.

1. **Install skill lives at `skills/install/SKILL.md`.** Directory name `install` — not `specscore-install` — to avoid the double-prefix problem called out in ADR-0003. Rendered invocation: `/specscore:install`.

2. **`user-invocable: true`.** Install is a human entry point on first use, exactly the case ADR-0005 calls out for `true`. A human who just ran `/plugin install` should be able to type `/specscore:install` and see it in the slash menu without needing Claude to auto-invoke it.

3. **Install is "infrastructure," not a CLI-wrapper.** It does not wrap a `specscore install` CLI command (no such command exists). This makes it the first *infrastructure* skill in the plugin, distinct from the planned resource-verb skills (`feature/`, `task/`, `spec/`, etc.). `skills/README.md` will grow a short section distinguishing the two categories.

4. **Pre-flight pattern: inline snippet in `skills/README.md`, not a standalone file.** There are zero consumer skills today (scaffold only). Creating `skills/install/references/preflight.md` now would be a file with no readers. Instead, `skills/README.md` documents the exact copy-paste snippet future skill authors use. When the first real wrapper skill (`feature/`) lands and we confirm the snippet is right, extracting to a shared file becomes a five-minute change. Per CLAUDE.md §2 (Simplicity First): no abstractions for zero-use code.

5. **No `--force` / re-install flag for v1.** The one-pager's Q5 confirmed the installer is idempotent (always-latest, no breaking changes). Re-running is safe. Add a flag only if evidence contradicts this.

6. **The install skill does not re-implement platform detection.** It shells out to `curl | sh` and lets the upstream `get-cli` script handle uname/arch. This is the core reason the plugin stays thin.

## Dependency Graph

```
Task 1: skills/install/SKILL.md
   │
   ├── references to → /specscore:install (self)
   │
   └── prerequisite for →
         │
         Task 2: skills/README.md
            │      (adds install to catalogue + documents pre-flight convention
            │       + adds "infrastructure vs wrapper" distinction)
            │
            └── prerequisite for →
                  │
                  Task 3: README.md
                         ("First use" paragraph linking to the install skill)
```

Strictly sequential. Each task is single-file, S-sized. No parallelism opportunity; not worth the coordination overhead for three markdown files.

## Task List

### Phase 1: Ship the install skill

#### Task 1: Write `skills/install/SKILL.md`

**Description:** Create the install skill. Frontmatter declares `name`, `description`, `user-invocable: true`. Body is a short explainer + the single Bash command `curl -fsSL https://specscore.md/get-cli | sh`. Include exit-code interpretation, a "when to use" trigger section, and a note on idempotency (re-running is safe).

**Acceptance criteria:**
- [ ] File exists at `skills/install/SKILL.md` with valid frontmatter (`name`, `description`, `user-invocable: true`).
- [ ] Frontmatter `description` names the one user-facing outcome ("install the `specscore` CLI") in a single sentence that Claude's auto-invoke can match on.
- [ ] Body contains exactly one bash command block (`curl -fsSL https://specscore.md/get-cli | sh`) — the canonical command, no variants.
- [ ] "When to use" section lists the trigger conditions (CLI missing, user asked to install, another skill reports `command not found: specscore`).
- [ ] Exit-code interpretation section maps `0` → success, non-zero → show stderr and stop.
- [ ] Notes that re-running is safe (idempotent).
- [ ] No `references/` subdirectory (single action, no verbs).

**Verification:**
- [ ] Visual review: frontmatter parses as YAML (no tab/indent errors).
- [ ] Visual review: description is ≤ 1 sentence, names the outcome, would realistically match a user asking "install specscore."
- [ ] Manual end-to-end (deferred to checkpoint): on a machine without `specscore`, installing this plugin locally and invoking `/specscore:install` results in `specscore` on `$PATH`.

**Dependencies:** None.

**Files likely touched:**
- `skills/install/SKILL.md` (new)

**Estimated scope:** S (1 file).

---

### Phase 2: Document the conventions

#### Task 2: Update `skills/README.md` — catalogue + pre-flight + categories

**Description:** Three additive edits in one file: (a) add `install` to the planned skill catalogue table, marked as **Infrastructure**; (b) introduce a short "Skill categories" subsection distinguishing *Infrastructure* skills (plugin plumbing) from *CLI-wrapper* skills (resource+verb); (c) add a "Pre-flight pattern" subsection containing the canonical copy-paste snippet that future CLI-wrapper skills include at the top of their `SKILL.md` to check for `specscore` and emit the two-path error on miss.

**Acceptance criteria:**
- [ ] Catalogue table in `skills/README.md` includes an `install` row with category = Infrastructure and a link to `install/SKILL.md`.
- [ ] New "Skill categories" subsection explains the Infrastructure vs CLI-wrapper distinction in ≤ 5 sentences.
- [ ] New "Pre-flight pattern" subsection contains the canonical pre-flight snippet as a fenced code block, the exact error message text, and a one-sentence instruction: *future wrapper skills include this snippet at the top of their `SKILL.md`*.
- [ ] The two-path error wording mentions both options (invoke `/specscore:install` **and** the manual `curl | sh` fallback).
- [ ] The existing "Status: Scaffold only" note is updated to reflect that install is the first shipped skill.

**Verification:**
- [ ] Visual review: table renders correctly, no broken relative links.
- [ ] Visual review: the pre-flight snippet is syntactically valid bash and actually runs (`command -v specscore >/dev/null 2>&1 || { ...; exit 1; }`) — test by pasting into a shell.
- [ ] Re-read as a prospective skill author: is the pre-flight snippet copy-pasteable without modification? If it needs edits per skill, the snippet is wrong.

**Dependencies:** Task 1 (the pre-flight snippet references `/specscore:install`, which must exist as a path before we document it).

**Files likely touched:**
- `skills/README.md` (edit)

**Estimated scope:** S (1 file).

---

### Phase 3: Surface it to humans

#### Task 3: Update root `README.md` — "First use" paragraph

**Description:** Add a short section after the existing "Install" block that walks a first-time user through the single additional step required after `/plugin install`: invoke `/specscore:install` (or run the manual curl fallback). Keep the existing "CLI is a prerequisite" sentence — the new section clarifies *how* to satisfy that prerequisite without leaving Claude Code.

**Acceptance criteria:**
- [ ] New "First use" section appears after the existing "Install" section, before "Relationship to the CLI."
- [ ] Section names both paths (slash invocation + manual curl).
- [ ] Section is ≤ 6 lines of prose. If longer, it's overbuilt.
- [ ] No change to existing sections beyond minimal linking if needed.

**Verification:**
- [ ] Visual review: reads cleanly as a continuation of the existing README tone (which is terse and directive).
- [ ] Re-read as a first-time user who has never seen the repo: after finishing the Install block, is the "what next?" question answered within 30 seconds of scanning?

**Dependencies:** Task 1 (the skill must exist for the README to link to it).

**Files likely touched:**
- `README.md` (edit)

**Estimated scope:** S (1 file).

---

## Checkpoints

### Checkpoint A: After Task 1

- [ ] `skills/install/SKILL.md` exists and parses.
- [ ] Local plugin install path chosen: either `/plugin install` from a file-system path, or `/plugin marketplace add` from a local/test marketplace. Document which in the commit message.
- [ ] On a machine without `specscore` (confirm with `which specscore` = empty), install this plugin locally, invoke `/specscore:install`, approve the Bash permission prompt, and confirm `which specscore` now resolves.
- [ ] Invoke `specscore --version` and confirm it returns a semver. This is the true end-to-end success signal.

**If Checkpoint A fails:** Stop. The install skill is the load-bearing piece; Tasks 2–3 are documentation around it. Do not proceed.

### Checkpoint B: After Tasks 2–3 (complete)

- [ ] `skills/README.md` and `README.md` render cleanly on GitHub (push to a branch and eyeball).
- [ ] All relative links resolve (`./install/SKILL.md`, plugin manifest, ADRs).
- [ ] A fresh reader can go from "I just ran `/plugin install specscore@sneat-co`" to "I have a working `specscore` CLI" using only the repo's README and one slash invocation.
- [ ] Human review + approval before commit/PR.

---

## Risks and Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| `specscore.md/get-cli` URL moves / 404s | High — breaks the skill end-to-end | Keep the URL as the single source of truth. If it changes, the skill is a one-line edit; no logic duplication. |
| `curl \| sh` triggers org-level security policy | Medium — some users will refuse to approve | The manual-curl fallback in the error message *is* the escape hatch; they can paste into their own terminal. Document in skill body. |
| Windows users (no bash/curl natively) | Low–Medium — Q2 narrowed audience to humans on laptops, but that includes Windows | Call out in the skill body: Windows users should use WSL or follow the upstream `install.md` page. Do not block the PR on this. Track as follow-up. |
| Claude Code plugin reload UX varies between versions | Low | Pin the minimum Claude Code version in `plugin.json` if testing reveals a floor; the README already notes v2.1.110 for transitive installs. |
| `get-cli` script contract drifts (e.g., changes env-var behavior, requires sudo) | Medium | The skill trusts upstream entirely; any drift shows up in the next manual end-to-end run. Coupling is deliberate — it is cheaper than re-implementing detection. |
| The pre-flight snippet in `skills/README.md` decays before the first consumer skill lands | Low | Snippet is ≤ 10 lines and shell-obvious; decay risk low. When `feature/` lands it will force the first real test. |

## Open Questions (deferred to implementation)

These are small enough to resolve while writing, not by planning:

- Exact `description:` frontmatter string (needs to be short and auto-invoke-friendly — iterate when writing).
- Exact error-message wording in the pre-flight snippet (two-path phrasing — settle after seeing it render).
- Whether to surface the install skill in the root `README.md` catalogue or only under `skills/README.md` (leaning: mention in root, detail in `skills/`).

## Parallelization

None. Three sequential markdown tasks. Coordination overhead exceeds benefit.
