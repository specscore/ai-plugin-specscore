# SpecScore AI Plugin

AI plugin for [SpecScore](https://specscore.md) — skills that teach AI agents how to use the `specscore` CLI for spec navigation, linting, and lifecycle operations.

This repository contains the plugin source. It is installed on top of the [`specscore` CLI](https://github.com/specscore/specscore-cli); the CLI is a prerequisite.

## Contents

| Directory | Description |
|---|---|
| [`skills/`](skills/README.md) | Agent skills — one per SpecScore CLI resource group, progressively loaded per-verb |
| [`.claude-plugin/`](.claude-plugin/plugin.json) | Claude Code plugin manifest |
| [`.codex-plugin/`](.codex-plugin/plugin.json) | Codex plugin manifest |
| [`gemini-extension.json`](gemini-extension.json) | Gemini CLI extension manifest |
| [`.github/plugin.json`](.github/plugin.json) | GitHub Copilot CLI / VS Code agent plugin manifest |

## Install

The same `skills/<name>/SKILL.md` payload is shared across all agents — only the manifest each agent reads differs.

### Claude Code

Via the [SpecScore AI marketplace](https://github.com/specscore/ai-marketplace):

```
/plugin marketplace add specscore/ai-marketplace
/plugin install specscore@specscore
```

Requires Claude Code v2.1.110 or later if installed transitively as a dependency of another plugin.

### Codex

```
codex plugin marketplace add specscore/ai-marketplace
```

Then enable the `specscore` plugin from Codex's plugin directory. (Codex has no `codex plugin add` subcommand; installation happens through the plugin directory after the marketplace is added.) Codex reads [`.codex-plugin/plugin.json`](.codex-plugin/plugin.json).

### Gemini CLI

```
gemini extensions install https://github.com/specscore/ai-plugin-specscore
```

Gemini reads [`gemini-extension.json`](gemini-extension.json) and auto-discovers the bundled `skills/` tree.

### GitHub Copilot CLI

```
copilot plugin install specscore/ai-plugin-specscore
```

Copilot reads [`.github/plugin.json`](.github/plugin.json).

### Cursor

Cursor has no remote-install command — it loads skills from `.cursor/skills/` (project) or `~/.cursor/skills/` (global). Add these skills by copying them in:

```
git clone https://github.com/specscore/ai-plugin-specscore
mkdir -p .cursor/skills && cp -R ai-plugin-specscore/skills/* .cursor/skills/
```

## First use

The `specscore` CLI must be on your `PATH` before any wrapper skill can run. Two options:

- Invoke `specscore:install` inside your AI agent — the [install skill](skills/install/SKILL.md) shows the official installer options.
- Or run manually in your terminal: `curl -fsSL https://specscore.md/get-cli | sh`.

Verify with `specscore --version`.

## Relationship to the CLI

The plugin wraps the `specscore` CLI — it does not replace it. Skills encode *when* to call a command, *which* flags to pass, and *how* to interpret exit codes. The CLI contract (commands, flags, exit codes) is defined in [`specscore/specscore-cli`](https://github.com/specscore/specscore-cli).

A change in the CLI surface typically produces a matching skill update in this repository; the two evolve together but release independently.

## Relationship to other plugins

`specscore` is a base-layer **CLI wrapper** plugin. Methodology plugins such as [`specstudio`](https://github.com/specscore/specstudio-skills) depend on it to compose multi-step workflows. See [ADR-0004](https://github.com/synchestra-io/synchestra/blob/main/spec/decisions/0004-layered-plugin-architecture.md) for the full layering rationale.

Sister plugin: [`synchestra-cli`](https://github.com/synchestra-io/ai-plugin-synchestra) — wraps the `synchestra` CLI using the same structure.

## Releases

Releases are tagged as `specscore--v<version>` on this repository to support Claude Code's dependency resolution. See [ADR-0004](https://github.com/synchestra-io/synchestra/blob/main/spec/decisions/0004-layered-plugin-architecture.md#follow-ups) for the convention.

## License

MIT — see [LICENSE](LICENSE).

## Open Questions
None at this time.
