# CLI Install Skill

## Problem Statement

**How might we ensure that after `/plugin install specscore@sneat-co`, the very next `/specscore:*` invocation succeeds — without making the plugin itself own multi-platform binary distribution?**

Today there is an implicit gap: the plugin README declares the `specscore` CLI a prerequisite, but nothing in the plugin surface helps the user cross that gap. A human runs `/plugin install`, invokes a skill, and gets `command not found: specscore` with no in-context path forward.

## Recommended Direction

Ship a single skill — `/specscore:install` — that wraps the existing `curl -fsSL https://specscore.md/get-cli | sh` installer. Every other skill in the plugin performs a pre-flight `command -v specscore` check; on miss, it emits a single error that references both paths (invoke the install skill, or run the curl command manually). Reject bundling binaries, reject a SessionStart hook, reject a long-lived "doctor" skill.

This direction leans on three assets the project already has: (1) the `get-cli` script at `specscore.md/get-cli` already handles platform detection (darwin/linux/windows × amd64/arm64) by pulling the right GoReleaser archive from GitHub releases — the plugin does not re-solve multi-platform; (2) Claude Code's existing Bash permission prompt *is* the consent UI for agent-executed installers — we do not invent a new one; (3) the CLI guarantees no breaking changes and "always latest" is acceptable (Q5), which collapses the version-enforcement job to nothing and eliminates the need for a doctor skill.

The pattern ports cleanly to sibling plugins (`synchestra-cli` has the same shape): the install skill is three files and is essentially a template with the CLI name swapped.

## Key Assumptions to Validate

- [ ] **Claude Code's Bash permission UX is acceptable consent for `curl | sh`.** Test: write the install skill, invoke it fresh, confirm the permission prompt surfaces the exact command and that users find the flow unsurprising.
- [ ] **Error-with-hint friction is tolerable for the once-per-machine case.** Test: run a skill without the CLI installed, read the error as a new user would, confirm the next step is obvious in under 5 seconds.
- [ ] **`get-cli` is a stable enough external dependency to lean on.** Test: review its recent change history; confirm the script is versioned or stable, and that we are comfortable coupling the plugin's install flow to its URL remaining live.

## MVP Scope

**In:**
- `skills/install/SKILL.md` — single skill, invokes `curl -fsSL https://specscore.md/get-cli | sh` via Bash tool.
- Shared pre-flight text used across all skills in the plugin: `command -v specscore` check, and on miss, a two-option error (invoke `/specscore:install` OR run the curl manually). Location TBD (see Open Questions).
- README update: add a short "First use" paragraph pointing at the install skill, complementing the existing "CLI is a prerequisite" line.

**Out:**
- Any per-skill embedding of install logic beyond the shared pre-flight block.
- Any attempt to re-implement platform detection inside the plugin.
- Any attempt to pin, check, or compare CLI versions.

## Not Doing (and Why)

- **Bundling CLI binaries in the plugin** — ~10–20MB × 5 platforms = 50–100MB plugin weight, and it fights ADR-0004 / README §Releases, which explicitly decouple plugin and CLI release cycles.
- **SessionStart / PreToolUse hook** — runs on every Claude Code session for anyone who has the plugin, including transitive installs via methodology plugins (e.g. `spec-studio`). Executing `curl | sh` checks on sessions where the user never intends to use specscore is surprising behavior for a CLI wrapper plugin.
- **A separate "doctor" skill** — Q5 confirmed "always latest, no breaking changes." There is no version-enforcement job, so `doctor` collapses into `install`. Don't build the thing you don't need.
- **Lazy-download shim (`bin/specscore` in plugin that fetches on first run)** — duplicates what `get-cli` already does; adds a second install path to maintain alongside the canonical one.
- **Version pinning of the CLI from the plugin** — explicitly out of scope per Q5.
- **Interactive / CI-aware variants of the install flow** — Q2 confirmed the audience is humans on laptops; CI is not a first-class concern for v1.

## Open Questions

- **Shape of the shared pre-flight.** Inline in each `SKILL.md`, a common `references/preflight.md` that every skill links, or a plugin-level convention documented once in `skills/README.md`? This is style, not architecture — decide at implementation time.
- **Wording of the error message.** Needs to make both paths (skill invocation vs manual curl) legible without being verbose. One iteration after seeing it render.
- **Does `/specscore:install` need a `--force` / re-install mode?** For v1, no — "always latest, no breaking changes" means re-running the script is safe and idempotent enough. Revisit if that assumption breaks.
- **Should the same pattern be documented once for re-use by `synchestra-cli` and future sibling wrapper plugins?** Likely yes, but belongs in a Synchestra-level ADR, not this plugin's docs.
